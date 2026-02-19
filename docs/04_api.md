# 04. API Specification

이 문서는 ShopSphere 이커머스 시스템의 API를 요약 표로 정리한다.  
상세 Request/Response 예시는 핵심 API(주문 생성, 로그인)에만 포함한다.

---

## 1. 공통 규칙

- Base URL: `/api/v1`
- 인증 방식: JWT (Authorization: Bearer {token})
- 응답은 JSON 형식
- 시간은 ISO-8601 문자열 사용

---

## 2. Authentication API

| API | Method | URL | Auth | Request 요약 | Response 요약 | Status |
|---|---|---|---|---|---|---|
| 회원가입 | POST | `/auth/signup` | X | email, password, name | id, email | 201 |
| 로그인 | POST | `/auth/login` | X | email, password | accessToken, refreshToken | 200 |
| 토큰 재발급(선택) | POST | `/auth/refresh` | X | refreshToken | accessToken | 200 |
| 로그아웃(선택) | POST | `/auth/logout` | O | - | - | 204 |

### 로그인 Request/Response 예시

**POST** `/api/v1/auth/login`

Request
```json
{
  "email": "test@test.com",
  "password": "1234"
}
```

Response (200)
```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token"
}
```

---

## 3. Product API

| API | Method | URL | Auth | Request 요약 | Response 요약 | Status |
|---|---|---|---|---|---|---|
| 상품 목록 조회 | GET | `/products?page=&size=&sort=` | X | query: page,size,sort | 상품 리스트 + 페이징 | 200 |
| 상품 상세 조회 | GET | `/products/{productId}` | X | path: productId | 상품 상세 | 200 |
| 상품 등록(판매자/관리자) | POST | `/products` | O | name, price, stockQty, categoryId | productId | 201 |
| 상품 수정(판매자/관리자) | PUT | `/products/{productId}` | O | name/price/stockQty 등 | productId | 200 |
| 상품 삭제(판매자/관리자) | DELETE | `/products/{productId}` | O | - | - | 204 |

---

## 4. Cart API (선택)

| API | Method | URL | Auth | Request 요약 | Response 요약 | Status |
|---|---|---|---|---|---|---|
| 장바구니 담기 | POST | `/cart/items` | O | productId, quantity | cartItemId | 201 |
| 장바구니 조회 | GET | `/cart` | O | - | 장바구니 목록 | 200 |
| 장바구니 수량 변경 | PATCH | `/cart/items/{cartItemId}` | O | quantity | cartItemId | 200 |
| 장바구니 삭제 | DELETE | `/cart/items/{cartItemId}` | O | - | - | 204 |

---

## 5. Order API

| API | Method | URL | Auth | Request 요약 | Response 요약 | Status |
|---|---|---|---|---|---|---|
| 주문 생성 | POST | `/orders` | O | items[{productId,quantity}] | orderId, status, totalPrice | 201 |
| 내 주문 목록 조회 | GET | `/orders?page=&size=` | O | query | 주문 목록 + 페이징 | 200 |
| 주문 상세 조회 | GET | `/orders/{orderId}` | O | path: orderId | 주문 상세(아이템 포함) | 200 |
| 주문 취소 | POST | `/orders/{orderId}/cancel` | O | - | status=CANCELLED | 200 |

### 주문 생성 Request/Response 예시

**POST** `/api/v1/orders`

Request
```json
{
  "items": [
    { "productId": 1, "quantity": 2 },
    { "productId": 3, "quantity": 1 }
  ]
}
```

Response (201)
```json
{
  "orderId": 10,
  "status": "PENDING",
  "totalPrice": 5400000
}
```

---

## 6. Payment API (선택)

| API | Method | URL | Auth | Request 요약 | Response 요약 | Status |
|---|---|---|---|---|---|---|
| 결제 승인 요청 | POST | `/payments/confirm` | O | orderId, paymentKey | paymentId, status=PAID | 200 |
| 결제 조회 | GET | `/payments/{paymentId}` | O | path: paymentId | 결제 상세 | 200 |

---

## 7. 공통 에러 응답 규격

| 상황 | Status | errorCode | message 예시 |
|---|---:|---|---|
| Validation 실패 | 400 | INVALID_REQUEST | "필수 값이 누락되었습니다." |
| 인증 실패 | 401 | UNAUTHORIZED | "로그인이 필요합니다." |
| 권한 없음 | 403 | FORBIDDEN | "접근 권한이 없습니다." |
| 리소스 없음 | 404 | NOT_FOUND | "상품을 찾을 수 없습니다." |
| 재고 부족 | 409 | OUT_OF_STOCK | "재고가 부족합니다." |
| 서버 오류 | 500 | INTERNAL_ERROR | "서버 오류가 발생했습니다." |

에러 Response 예시
```json
{
  "timestamp": "2026-02-19T12:00:00",
  "status": 409,
  "errorCode": "OUT_OF_STOCK",
  "message": "재고가 부족합니다."
}
```
