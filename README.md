# ShopSphere

재고 동시성과 결제 정합성을 고려하여 설계한 이커머스 백엔드 시스템

---

## 1. Project Overview

ShopSphere는 소규모 브랜드를 위한 이커머스 플랫폼의 백엔드 시스템이다.  
단순 CRUD 구현이 아니라, 주문 처리 과정에서 발생할 수 있는 동시성 문제와 결제 정합성을 중심으로 설계하였다.

### 주요 설계 목표

- 주문 데이터 정합성 보장
- 재고 동시성 제어
- RESTful API 설계
- 확장 가능한 아키텍처 구조

---

## 2. Tech Stack

### Backend
- Spring Boot
- JPA (Hibernate)
- MySQL

### Cache
- Redis

### Security
- JWT 기반 인증

### DevOps
- Docker
- GitHub Actions
- AWS 배포 구조 설계

---

## 3. Architecture Summary

- 도메인 중심 패키지 구조 채택
- Redis 기반 재고 분산락 적용
- 주문 상태 전이 (PENDING → PAID / CANCELLED) 설계
- Load Balancer 기반 수평 확장 고려
- 공통 응답 및 에러 코드 체계 정의

---

## 4. Core Features

- 회원 가입 / 로그인 (JWT)
- 상품 조회 (페이징 및 정렬)
- 주문 생성 및 결제 연동
- 재고 동시성 제어
- 단위 및 통합 테스트 구성

---

## 5. Design Documents

- [Requirements](./docs/01_requirements.md)
- [System Architecture](./docs/02_architecture.md)
- [ERD](./docs/03_erd.md)
- [API Specification](./docs/04_api.md)
- [Sequence Diagram](./docs/05_sequence.md)
- [Package Structure](./docs/06_package.md)
- [Test Strategy](./docs/07_test.md)
- [Performance Considerations](./docs/08_performance.md)
- [Deployment Architecture](./docs/09_deployment.md)

---

## 6. Testing

- JUnit 기반 단위 테스트
- SpringBootTest 기반 통합 테스트
- 동시성 테스트 (ExecutorService 활용)
- 예외 케이스 검증

---

## 7. Run Locally

```bash
./gradlew build
docker-compose up
```

---

## 8. Design Focus

이 프로젝트는 다음 질문에 답할 수 있도록 설계되었다.

- 왜 Redis를 사용했는가?
- 왜 주문 상태를 PENDING으로 두는가?
- 왜 Order와 OrderItem을 분리했는가?
- 왜 도메인 중심 패키지 구조를 선택했는가?

---

## 9. Future Improvements

- Read Replica 도입
- 비동기 주문 처리 구조 도입
- MSA 전환 고려
- Kubernetes 기반 확장


---

## 10. Team

| Name | Role | Responsibility |
|------|------|----------------|
| Kim JM | Backend Lead | 전체 아키텍처 설계, 주문/결제 도메인 구현 |
| Lee JS | Backend Developer | 상품/회원 도메인 구현, API 설계 |
| Park HY | DevOps | Docker 구성, CI/CD 설정, 배포 구조 설계 |
| Choi MK | QA & Testing | 테스트 전략 수립, 통합 테스트 작성 |

---

## 11. Project Timeline

### Week 1 – Requirement & Design
- 요구사항 정의
- 유저 스토리 정리
- ERD 설계
- API 명세 작성
- 아키텍처 다이어그램 작성

### Week 2 – Core Implementation
- 회원 / 상품 기능 구현
- 주문 도메인 구현
- 재고 동시성 제어 적용
- JWT 인증 적용

### Week 3 – Advanced Features & Testing
- 결제 연동
- 예외 처리 체계 정리
- 단위 테스트 / 통합 테스트 작성
- 동시성 테스트 검증

### Week 4 – Performance & Deployment
- N+1 문제 해결
- Redis 캐싱 적용
- Docker 기반 배포 구성
- CI/CD 설정
- 문서 정리 및 리팩토링