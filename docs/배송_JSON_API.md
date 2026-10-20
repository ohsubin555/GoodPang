# 배송 JSON API (로그인 불필요)

`/admin` 밖에 있어서 `AdminAuthFilter`를 타지 않고, 관리자 로그인 없이 누구나 호출할 수 있는 JSON 버전 API.

| URL | Method | Servlet | 설명 |
|---|---|---|---|
| `/deliveries/json` | GET | DeliveryJsonServlet | 배송중인 상품 목록을 JSON 배열로 조회 |
| `/delivery-complete/json` | POST | DeliveryCompleteJsonServlet | 배송완료 처리 (JSON 응답) |

## GET /deliveries/json

- **동작**: `AdminDeliveryDAO.findShipping()` 결과를 그대로 JSON 배열로 반환 (`/admin/deliveries` 화면과 같은 데이터).
- **응답 예시**:
  ```json
  [
    {
      "deliveryNo": 12,
      "orderNo": 52,
      "deliveryServiceCode": "CJ",
      "invoiceNo": "123456789",
      "deliveryStatus": "배송중",
      "deliveryStartDate": "...",
      "buyerName": "홍*동",
      "buyerPhone": "010-0000-****",
      "zipcode": "12345",
      "address": "서울특별시 강남구 테헤란로 1",
      "detailAddress": "101동 101호",
      "productName": "반팔 티셔츠",
      "productImageUrl": "./pds/short_1.jpg",
      "storeName": "베이직루트",
      "itemCount": 2
    }
  ]
  ```
- **주의**: `buyerName`(가운데 글자 마스킹, 예: `홍*동`)과 `buyerPhone`(뒷 4자리 마스킹, 예: `010-0000-****`)은 마스킹해서 반환하지만, 그 외 항목은 그대로 노출되므로 로컬/테스트 용도로만 사용할 것.

## POST /delivery-complete/json

- **파라미터**: `deliveryNo` (int, `application/x-www-form-urlencoded`)
- **동작**: 내부적으로 `AdminDeliveryDAO.completeDelivery(deliveryNo)`를 그대로 재사용 — `/admin/delivery-complete`와 동일하게 배송 상태 + 주문 상태를 한 트랜잭션으로 갱신.
- **응답 예시**:
  - 성공: `{"success": true}`
  - 실패(이미 처리됐거나 존재하지 않는 배송번호): `{"success": false, "message": "이미 처리되었거나 존재하지 않는 배송번호입니다."}`
  - 실패(`deliveryNo` 누락/숫자 아님): `{"success": false, "message": "deliveryNo가 올바르지 않습니다."}`
