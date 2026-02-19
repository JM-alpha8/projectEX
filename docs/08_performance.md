# 08. Performance Considerations

본 프로젝트는 이커머스 특성상 조회 트래픽이 많고,
주문 시 동시성이 집중되는 구조를 고려하여 성능 최적화를 설계하였다.

---

## 1. 상품 조회 성능 최적화

### 문제

- 상품 목록 조회는 가장 많이 호출되는 API
- N+1 문제 발생 가능
- 페이지네이션 없이 전체 조회 시 성능 저하

---

### 해결 전략

1. 페이징 처리 적용 (Pageable)
2. Fetch Join 또는 DTO Projection 사용
3. 인덱스 설정

예시:

- product.name 인덱스
- category_id 인덱스

---

## 2. N+1 문제 해결 전략

### 문제

주문 조회 시:

- Order 조회
- OrderItem 조회
- Product 조회

Lazy Loading으로 인해 쿼리가 반복 실행될 수 있음

---

### 해결

- Fetch Join 사용
- @EntityGraph 활용
- DTO Projection 활용

예시:

```java
@Query("select o from Order o join fetch o.orderItems oi join fetch oi.product where o.id = :id")
Optional<Order> findOrderWithItems(Long id);
```

---

## 3. 캐싱 전략

### 적용 대상

- 상품 상세 조회
- 인기 상품 목록

---

### Redis 활용

- 상품 상세는 Cache-Aside 패턴 적용
- TTL 설정 (예: 5분)

동작 흐름:

1. Redis 조회
2. 없으면 DB 조회
3. Redis 저장
4. 응답 반환

---

## 4. 재고 동시성 제어

### 문제

동시에 여러 주문이 들어올 경우:

- 재고 음수 발생 가능
- 중복 주문 발생 가능

---

### 해결 전략

1. Redis 분산 락
2. DB 레벨 Lock (FOR UPDATE)
3. 낙관적 락 (Version Column)

상황에 따라 전략 선택 가능하도록 설계하였다.

---

## 5. 트랜잭션 범위 최소화

- 외부 결제 API 호출은 트랜잭션 범위 밖에서 처리 고려
- DB 락 유지 시간을 최소화
- 긴 트랜잭션 방지

---

## 6. 인덱스 전략

다음 컬럼에 인덱스 적용:

- member.email (로그인 빈도 높음)
- product.name
- order.created_at
- order.member_id

목적:

- 로그인 성능 향상
- 주문 내역 조회 속도 개선

---

## 7. 확장 고려

### 읽기 트래픽 증가 시

- Read Replica 도입
- 캐시 레이어 확장

### 주문 트래픽 증가 시

- 주문 처리 비동기화 (Kafka 등)
- Redis Cluster 구성

---

## 8. 성능 설계 의도

- 조회와 쓰기 트래픽 특성을 구분하여 최적화
- 동시성 문제를 사전에 고려
- 확장 가능한 구조 설계

이커머스는 단순 CRUD 시스템이 아니라
"동시성 + 정합성 + 조회 최적화"가 핵심이라는 점을 반영하였다.
