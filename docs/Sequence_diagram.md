# 시퀀스 다이어그램

---

## 잔액 충전 / 조회 API
- 결제에 사용될 금액을 충전하는 API를 작성합니다.
- 사용자 식별자와 충전할 금액을 받아 잔액을 충전합니다.
- 사용자 식별자를 통해 해당 사용자의 잔액을 조회합니다.

### 잔액 충전
```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant Auth as Auth 인증
    participant PointService as 잔액 서비스
    participant HistoryService as 잔액 이력 서비스

%% 잔액 충전
    User ->>+ API: POST /members/{memberId}/points/charge
    API ->>+ Auth: JWT 토큰 검증
    alt 인증 성공
        Auth -->>- API: 인증 성공
        API ->>+ PointService: 포인트 조회 (userId)
        PointService -->>- API: 현재 포인트
        API ->>+ PointService: 포인트 충전 (userId, amount)
        PointService -->>- API: 충전 성공 응답
        API ->>+ HistoryService: 충전 내역 기록 (userId, amount)
        HistoryService -->>- API: 내역 기록 성공
        API -->> User: 충전 성공 응답
    else 인증 실패
        Auth -->> API: 인증 실패 응답
        API -->>- User: 인증 실패 응답 (401 Unauthorized)
    end

```
### 잔액 조회
```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant Auth as Auth 인증
    participant PointService as 잔액 서비스

    %% 잔액 조회
    User ->>+ API: GET /members/{memberId}/points
    API ->>+ Auth: JWT 토큰 검증
    alt 인증 성공
        Auth -->>- API: 인증 성공
        API ->>+ PointService: 포인트 조회 (userId)
        PointService -->>- API: 현재 포인트
        API -->> User: 잔액 조회 응답 (points)
    else 인증 실패
        Auth -->> API: 인증 실패 응답
        API -->>- User: 인증 실패 응답 (401 Unauthorized)
    end

```

---

## 기존 상품 조회 API
- 상품 정보(ID, 이름, 가격, 잔여수량)를 조회하는 API를 작성합니다.
- 조회 시점의 상품별 잔여 수량이 정확해야 합니다.<br/>
<br/>

```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant Auth as JWT 인증
    participant ProductService as 상품 서비스

%% 기존 상품 조회
    User ->>+ API: POST /products/{productId}
    API ->>+ Auth: JWT 토큰 검증
    alt 인증 성공
        Auth -->>- API: 인증 성공
        API ->>+ ProductService: 상품 정보 조회 (productId)
        ProductService -->>- API: 상품 정보 응답
        API -->> User: 상품 정보 응답 (product details)
    else 인증 실패
        Auth -->> API: 인증 실패 응답
        API -->>- User: 인증 실패 응답 (401 Unauthorized)
    end

```

---

## 주요 선착순 쿠폰 발급 / 조회 API
- 선착순 쿠폰 발급 API 및 보유 쿠폰 목록 조회 API를 작성합니다.
- 사용자는 선착순으로 할인 쿠폰을 발급받을 수 있습니다.
- 주문 시 유효한 할인 쿠폰을 제출하면 전체 주문 금액에 대해 할인 혜택을 받을 수 있습니다.

### 선착순 쿠폰 발급
```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant Auth as JWT 인증
    participant CouponService as 쿠폰 서비스

    %% 쿠폰 발급
    User ->>+ API: POST /coupons/{couponId}
    API ->>+ Auth: JWT 토큰 검증
    alt 인증 성공
        Auth -->>- API: 인증 성공
        API ->>+ CouponService: 쿠폰 조회 및 유효성 검사 (couponId)
        alt 쿠폰 유효
            API ->>+ CouponService: 쿠폰 발급 요청
            CouponService ->> CouponService: 쿠폰 재고 차감
            CouponService ->> CouponService: 사용자 쿠폰 등록
            CouponService -->>- API: 쿠폰 발급 응답
            API -->> User: 쿠폰 발급 성공 응답
        else 쿠폰 유효하지 않음
            CouponService -->>- API: 유효하지 않은 쿠폰
            API -->> User: 유효하지 않은 쿠폰 응답
        end
    else 인증 실패
        Auth -->> API: 인증 실패 응답
        API -->>- User: 인증 실패 응답 (401 Unauthorized)
    end
```

### 선착순 쿠폰 조회
```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant Auth as Jwt 인증
    participant CouponService as 쿠폰 서비스

    %% 선착순 쿠폰 조회
    User ->>+ API: GET /coupons/{couponId}
    API ->> Auth: JWT 토큰 검증
    alt 인증 성공
        Auth -->> API: 인증 성공
        API ->>+ CouponService: 쿠폰 조회 (couponId)
        CouponService ->> CouponService: 재고 수량 확인
        opt 쿠폰 재고가 없을 경우
            CouponService ->> API: 쿠폰 발급 실패 응답
            API ->> User: 쿠폰 발급 실패 응답
        end
        CouponService -->>- API: 쿠폰 조회 응답 (재고 수량 포함)
        API -->> User: 쿠폰 조회 응답 (재고 수량 포함)
    else 인증 실패
        Auth -->> API: 인증 실패 응답
        API -->>- User: 인증 실패 응답 (401 Unauthorized)
    end

```

---

## 주요 주문 / 결제 API
- 사용자 식별자와 (상품 ID, 수량) 목록을 입력받아 주문하고 결제를 수행하는 API를 작성합니다.
- 결제는 충전된 잔액을 기반으로 수행하며, 성공 시 잔액을 차감합니다.
- 데이터 분석을 위해 결제 성공 시 실시간으로 주문 정보를 데이터 플랫폼에 전송해야 합니다.
    - 데이터 플랫폼은 어플리케이션 외부에 존재하며, Mock API나 Fake Module 등으로 구현 가능합니다.

### 주문 API
```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant Auth as Jwt 인증
    participant OrderService as 주문 서비스
    participant ProductService as 상품 서비스
    participant CartService as 장바구니 서비스
    participant PaymentService as 결제 서비스

    %% 상품 주문
    User ->>+ API: POST /order/{userId} (상품 ID, 수량)
    API ->> Auth: JWT 토큰 검증
    alt 인증 성공
        Auth -->> API: 인증 성공
        API ->>+ OrderService: 주문 요청
        OrderService ->>+ ProductService: 재고 조회 요청
        ProductService ->> ProductService: 재고 수량 확인
        ProductService -->>- OrderService: 재고 조회 응답
        opt 재고가 없을 경우
            OrderService -->> API: 주문 실패 응답
            API -->> User: 주문 실패 응답
        end
        OrderService -->> API: 상품 조회 응답 (재고 수량 포함)
        OrderService ->>+ CartService: 장바구니 제거 요청
        CartService -->>- OrderService: 장바구니 제거 응답
        OrderService ->>+ PaymentService: 결제 생성 요청 (결제 준비)
        PaymentService -->>- OrderService: 결제 생성 응답
        OrderService -->> OrderService: 주문 생성
        OrderService -->>- API: 주문 생성 응답
        API -->> User: 주문 생성 응답
    else 인증 실패
        Auth -->> API: 인증 실패 응답
        API -->>- User: 인증 실패 응답 (401 Unauthorized)
    end
```

### 결제 API
```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant Auth as Jwt 인증
    participant PaymentService as 결제 서비스
    participant OrderService as 주문 서비스
    participant BalanceService as 잔액 서비스
    participant DataPlatform as 데이터 플랫폼

%% 결제 처리
    User ->>+ API: POST /payment/{paymentId}
    API ->>+ Auth: JWT 토큰 검증
    alt 인증 성공
        Auth -->>- API: 인증 성공
        API ->>+ PaymentService: 결제 요청
        PaymentService ->>+ OrderService: 주문 조회
        OrderService -->>- PaymentService: 주문 정보 응답
        PaymentService ->>+ BalanceService: 잔액 확인 및 차감
        activate BalanceService
        BalanceService ->> BalanceService: 잔액 확인
        alt 잔액 부족
            BalanceService -->>- PaymentService: 잔액 부족 응답
            PaymentService -->> API: 결제 실패 응답 (잔액 부족)
            API -->> User: 결제 실패 (잔액 부족)
        else 잔액 충분
            BalanceService -->> BalanceService: 잔액 차감
            BalanceService -->> PaymentService: 잔액 차감 성공
            PaymentService ->> PaymentService: 결제 상태 업데이트 (결제 완료)
            PaymentService ->> OrderService: 주문 상태 변경 요청 (주문 완료)
            OrderService ->> PaymentService: 주문 상태 변경 완료 응답
            PaymentService -->>- API: 결제 성공 응답
            API -->> User: 결제 성공 응답
        end
        deactivate BalanceService
    else 인증 실패
        Auth -->> API: 인증 실패 응답
        API -->>- User: 인증 실패 응답 (401 Unauthorized)
    end

```

---

## 상위 상품 조회 API

```mermaid
sequenceDiagram
    actor User as 유저
    participant API as API
    participant ProductService as 상품 서비스

    User->>+API: GET /products/top
    API->>+ProductService: 상위 상품 조회 요청
    ProductService-->>-API: 상위 상품 조회 응답
    API-->>-User: 상위 상품 조회 응답
```

