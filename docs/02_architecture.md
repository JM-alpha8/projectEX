# 02. System Architecture

## 1. 시스템 아키텍처 다이어그램

![System Architecture](./images/architecture.png)

---

## 2. 아키텍처 구성 설명

### 1) Client Layer
- 사용자는 Web Browser 또는 Mobile App을 통해 접속한다.
- 모든 요청은 HTTP/HTTPS 기반으로 전달된다.

### 2) Reverse Proxy
- Nginx를 사용하여 요청을 Spring Boot 서버로 전달한다.
- SSL 종료 처리 및 로드 밸런싱 역할을 수행한다.

### 3) Application Layer
- Spring Boot 기반 REST API 서버
- 비즈니스 로직 처리
- 트랜잭션 관리
- 인증 및 인가 처리 (JWT)

### 4) Database Layer
- MySQL을 사용하여 회원, 주문, 상품 데이터를 저장한다.
- ACID 기반 트랜잭션을 보장한다.

### 5) Cache Layer
- Redis를 사용하여
  - 재고 동시성 제어 (분산 락)
  - 상품 조회 캐싱

---

## 3. 아키텍처 설계 의도

- 애플리케이션 서버와 데이터 계층을 분리하여 확장성을 확보한다.
- Redis를 통해 동시성 문제를 해결한다.
- Stateless 인증(JWT)을 통해 수평 확장에 유리한 구조를 설계한다.
