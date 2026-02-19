# 06. Package Structure

본 프로젝트는 도메인 중심(Domain-driven) 패키지 구조를 채택하였다.
기능 단위로 응집도를 높이고, 도메인별 책임을 명확히 분리하는 것을 목표로 한다.

---

## 1. 전체 패키지 구조

```
com.shopsphere
 ├── global
 │    ├── config
 │    ├── exception
 │    ├── response
 │    └── security
 │
 ├── domain
 │    ├── member
 │    │     ├── controller
 │    │     ├── service
 │    │     ├── repository
 │    │     ├── entity
 │    │     └── dto
 │    │
 │    ├── product
 │    │     ├── controller
 │    │     ├── service
 │    │     ├── repository
 │    │     ├── entity
 │    │     └── dto
 │    │
 │    ├── order
 │    │     ├── controller
 │    │     ├── service
 │    │     ├── repository
 │    │     ├── entity
 │    │     └── dto
 │    │
 │    └── payment
 │          ├── service
 │          ├── entity
 │          └── dto
 │
 └── external
      └── paymentgateway
```

---

## 2. 패키지 설계 의도

### 1) global 패키지

공통 기능을 모아둔 영역이다.

- config: Redis, JPA, Security 설정
- exception: 전역 예외 처리
- response: 공통 응답 포맷 정의
- security: JWT 필터, 인증 처리

도메인과 무관한 기술적 공통 요소를 분리하여
의존성을 낮추고 유지보수성을 높였다.

---

### 2) domain 패키지

비즈니스 핵심 영역이다.

도메인 단위로 패키지를 구성하여
각 기능이 독립적으로 응집되도록 설계하였다.

예: order 도메인 안에는

- OrderController
- OrderService
- OrderRepository
- Order 엔티티
- Order DTO

가 모두 포함된다.

이는 기능 중심으로 코드를 추적하기 쉽게 만든다.

---

### 3) external 패키지

외부 시스템과의 연동을 담당한다.

예:
- 결제 시스템 API 연동
- 외부 배송 API 연동

비즈니스 로직과 외부 의존성을 분리하기 위해 별도 패키지로 구성하였다.

---

## 3. Layered 구조를 선택하지 않은 이유

전통적인 Layered 구조:

```
controller
service
repository
entity
```

이 방식은 기능이 많아질수록
관련 파일이 여러 패키지에 흩어져 추적이 어렵다.

반면 도메인 중심 구조는

- 기능 단위로 코드가 모여있어 가독성이 높고
- 팀 단위 협업 시 충돌을 줄일 수 있다.

---

## 4. 의존성 방향

의존성은 다음 방향으로만 흐르도록 설계하였다.

Controller → Service → Repository → Entity

도메인 간 직접 참조는 최소화하며,
필요 시 인터페이스를 활용하여 결합도를 낮춘다.

---

## 5. 향후 확장 고려

- Admin 도메인 분리 가능
- CQRS 패턴 도입 시 query 패키지 분리 가능
- MSA 전환 시 도메인 단위로 서비스 분리 가능

현재 구조는 이러한 확장을 고려하여 설계되었다.
