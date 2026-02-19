# 05. Sequence Diagram

이 문서는 ShopSphere 이커머스 시스템의 핵심 기능인 
"주문 생성" 프로세스의 정상 흐름과 예외 흐름을 설명한다.

---

# 1. 주문 생성 - 결제 성공 시퀀스

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant FE as Client (Web/Mobile)
    participant API as Spring Boot API
    participant AUTH as JWT Filter
    participant OS as OrderService
    participant PS as ProductService
    participant RS as Redis (Distributed Lock)
    participant DB as MySQL
    participant PG as Payment Gateway (External)

    Customer->>FE: 주문 요청 (상품, 수량)
    FE->>API: POST /api/v1/orders (JWT 포함)

    API->>AUTH: JWT 검증
    AUTH-->>API: 인증 성공

    API->>OS: createOrder(request)

    note over OS,RS: 재고 동시성 제어 시작

    OS->>RS: LOCK product:{id} (SETNX + TTL)
    RS-->>OS: lock acquired

    OS->>PS: validateStock(productId, quantity)
    PS->>DB: SELECT product WHERE id=? FOR UPDATE
    DB-->>PS: product row 반환

    PS->>DB: UPDATE stock_qty = stock_qty - quantity
    DB-->>PS: 재고 감소 완료

    OS->>DB: INSERT INTO orders (status=PENDING)
    DB-->>OS: order_id 생성

    OS->>DB: INSERT INTO order_items
    DB-->>OS: 저장 완료

    OS->>PG: 결제 요청 (order_id, total_price)
    PG-->>OS: 결제 성공 응답

    OS->>DB: UPDATE orders SET status=PAID
    DB-->>OS: 상태 변경 완료

    OS->>RS: UNLOCK product:{id}
    RS-->>OS: lock released

    OS-->>API: 주문 성공 응답
    API-->>FE: 201 Created (orderId, status=PAID)
    FE-->>Customer: 주문 완료 화면 표시
```

---

# 2. 주문 생성 - 결제 실패 시퀀스

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant FE as Client
    participant API as Spring Boot API
    participant AUTH as JWT Filter
    participant OS as OrderService
    participant PS as ProductService
    participant RS as Redis
    participant DB as MySQL
    participant PG as Payment Gateway

    Customer->>FE: 주문 요청
    FE->>API: POST /api/v1/orders

    API->>AUTH: JWT 검증
    AUTH-->>API: 인증 성공

    API->>OS: createOrder(request)

    OS->>RS: LOCK product:{id}
    RS-->>OS: lock acquired

    OS->>PS: validateStock()
    PS->>DB: SELECT product FOR UPDATE
    DB-->>PS: product row

    PS->>DB: UPDATE stock_qty = stock_qty - quantity
    DB-->>PS: 재고 감소 완료

    OS->>DB: INSERT INTO orders (status=PENDING)
    DB-->>OS: order_id 생성

    OS->>PG: 결제 요청
    PG-->>OS: 결제 실패 (timeout / declined)

    note over OS,DB: 보상 트랜잭션 시작

    OS->>DB: UPDATE orders SET status=CANCELLED
    OS->>DB: UPDATE products SET stock_qty = stock_qty + quantity

    OS->>RS: UNLOCK product:{id}
    RS-->>OS: lock released

    OS-->>API: 409 Conflict (결제 실패)
    API-->>FE: 결제 실패 안내
    FE-->>Customer: 결제 실패 메시지 표시
```

---

# 3. 설계 의도 설명

## 1) 상태 전이 기반 주문 관리

주문은 다음과 같은 상태 흐름을 가진다:

- PENDING → PAID (정상 결제)
- PENDING → CANCELLED (결제 실패)

결제 성공 이전에는 반드시 PENDING 상태로 유지하여,
결제 실패 시 보상 트랜잭션이 가능하도록 설계하였다.

---

## 2) 재고 동시성 제어 전략

이커머스 시스템에서는 동일 상품에 대한 동시 주문이 발생할 수 있다.

이를 해결하기 위해:

- Redis 기반 분산 락 사용
- TTL을 설정하여 데드락 방지
- DB 레벨에서도 FOR UPDATE 또는 낙관적 락 적용 가능

이중 보호 전략을 고려하였다.

---

## 3) Order와 OrderItem 분리 이유

- 하나의 주문에 여러 상품이 포함될 수 있음
- 다대다 관계를 해소하기 위해 중간 테이블(OrderItem) 설계
- OrderItem에 가격을 저장하여 주문 시점 가격 보존

---

## 4) 결제 실패 시 보상 트랜잭션

외부 결제 시스템은 실패 가능성이 존재한다.

따라서:
- 주문 상태를 CANCELLED로 변경
- 차감했던 재고를 복구
- 락을 해제

이를 통해 데이터 정합성을 유지한다.

---

## 5) 확장 고려 사항

향후 트래픽 증가 시:

- Redis 클러스터 구성
- 읽기/쓰기 DB 분리 (Read Replica)
- 주문 처리 비동기화 (Kafka 등 도입 가능)

현재 구조는 이러한 확장을 고려하여 설계되었다.
