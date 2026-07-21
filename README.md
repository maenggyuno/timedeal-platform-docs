# timedeal-platform-docs
# 🛒 동네콕 (DongneKok) - 하이퍼로컬 타임딜 커머스 플랫폼
O2O 커머스 도메인의 실제 비즈니스 딜레마를 시스템적으로 해결하는 데 집중한 프로젝트입니다. PM의 시선으로 비즈니스를 분석하고, CTO의 기준으로 아키텍처를 최적화하며, 엔지니어의 코드로 리얼 월드의 문제를 해결했습니다.

> **💡 안내 사항 (Notice)**
> 핵심 비즈니스 로직이 담긴 **소스 코드는 비공개(Private)** 처리하였습니다. 대신 본 문서를 통해 프로젝트의 아키텍처 설계, 보안/인프라 구축 과정, 트러블슈팅 경험 및 협업 시스템(이슈 템플릿 등)을 상세히 공개합니다. 상세한 기술 블로그 기록도 함께 참고 부탁드립니다.

<br>

## 🎯 1. Project Overview
오프라인 베이커리의 재고 관리 비효율을 해결하기 위해 기획된 지역 기반 타임딜 플랫폼입니다. 
초기 팀 프로젝트로 시작하여 백엔드를 전담하였으며, 현재는 **1인 리팩토링**을 통해 기존 레거시 인프라를 전면 폐기하고 **Cloudflare Zero Trust와 AWS Serverless 기반의 클라우드 네이티브 아키텍처**로 고도화하였습니다.

* **개발 기간:** 2025.03 - 2025.11 (초기 모델) / 2026.01 - 현재 (인프라 및 아키텍처 고도화)
* **담당 역할:** Backend & Cloud Infrastructure

<br>

## 🏗️ 2. System Architecture
<img width="925" height="507" alt="동네콕 아키텍처 drawio" src="https://github.com/user-attachments/assets/199231ee-5f43-42c6-a7eb-fa94f126818e" />




### 🌐 Domain & Routing Strategy
* **Frontend (`dongnekok.shop`):** AWS S3와 CloudFront를 연동하여 정적 자산의 글로벌 캐싱 성능을 확보하고 비용을 최적화했습니다. (Cloudflare DNS Only 적용)
* **Backend (`api.dongnekok.shop`):** Cloudflare Tunnel을 활용해 백엔드 서버의 공인 IP를 은닉하고, 단일 보안 터널을 통해서만 API 통신이 가능하도록 방어 표면을 구축했습니다. (Cloudflare Proxy 적용)

### 🗄️ Database Architecture (Logical ERD)
<img width="2084" height="3035" alt="erd" src="https://github.com/user-attachments/assets/84b83dd3-cd57-41ff-ae22-87e648ab886e" />



* **데이터 모델링 포인트:** 타임딜 커머스의 특성상 특정 시간대에 빈번하게 발생하는 '재고 조회 및 주문' 트랜잭션을 안정적으로 처리하기 위해 정규화를 진행하고, 로컬 상점과 사용자 간의 데이터 정합성을 보장하도록 관계형 데이터베이스 구조를 설계했습니다.

### [[🔗 벨로그 MerMaid 시리즈 ERD 설계 트러블 슈팅 기록](https://velog.io/@mgo0415/series/Project-ERD-%EB%8B%A4%EC%9D%B4%EC%96%B4%EA%B7%B8%EB%9E%A8)]

### 1️⃣ UX 편의성과 AI 데이터 무결성 충돌 해결: '판매권(Sale Offer)' 추상화 레이어 도입 [[🔗 묶음 상품 아키텍처 설계 기록](https://velog.io/@mgo0415/DB-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%82%AC%EC%9E%A5%EB%8B%98%EC%9D%98-%EA%B7%80%EC%B0%AE%EC%9D%8C-vs-AI-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%AC%B4%EA%B2%B0%EC%84%B1-%EB%AC%B6%EC%9D%8C-%EC%83%81%ED%92%88-%EC%84%A4%EA%B3%84%EC%9D%98-%EB%94%9C%EB%A0%88%EB%A7%88%EC%99%80-%ED%95%B4%EA%B2%B0%EC%B1%85)]

문제: 사장님 편의를 위한 묶음 상품(랜덤박스) 판매 요구사항과, AI 생산량 예측을 위한 개별 상품 단위의 데이터 수집(원자성)이 정면 충돌. 기존 상품 테이블에 묶음 데이터를 혼합할 경우 마스터 데이터 오염 및 인덱스 용량 비대화, 검색 성능 저하(Garbage Data 98% 차지) 문제 식별.

해결: 이커머스 및 O2O 플랫폼 아키텍처를 분석하여 상품(Master)과 이벤트성 판매 단위(Offer)를 분리하는 sale_offer 및 sale_offer_item 다대다(N:M) 매핑 테이블 구조 도입.

성과: Product 테이블의 오염도를 0%로 방어하고 검색 성능을 광속으로 유지함과 동시에, 사장님에게는 '원터치 묶음 등록(UX)'을 제공하고 백엔드에서는 이를 개별 상품으로 역산하여 AI 학습용 순도 100%의 데이터를 확보.

### 2️⃣ 관심사 분리(SoC)를 통한 레거시 DB 리팩토링: 매직 넘버 제거 및 도메인 상태(Status) 3분할 [[🔗 Status 리팩토링 기록](https://velog.io/@mgo0415/DB%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81-%EC%9D%B4%EA%B2%8C-%EC%99%9C-%EB%8F%8C%EC%95%84%EA%B0%80%EC%A7%80-%EB%A7%A4%EC%A7%81-%EB%84%98%EB%B2%84-%EC%A7%80%EC%98%A5%EC%97%90%EC%84%9C-%ED%83%88%EC%B6%9C%ED%95%98%EC%97%AC-%EC%83%81%ED%83%9CStatus-%EB%B6%84%EB%A6%AC%ED%95%98%EA%B8%B0)]

문제: 과거 레거시 설계에서 단일 status 숫자 컬럼(Magic Number) 하나로 배송 상태와 결제 수단을 혼재하여 관리. 이로 인해 데이터 확장성이 없고(OCP 위배), 휴먼 에러 시 데이터 전체가 오염될 수 있는 치명적 안티 패턴 발견.

해결: 상태 값의 책임을 상품(Product), 주문(Order), 결제(Payment) 3개의 독립적인 도메인으로 전면 분리. JPA @Enumerated(EnumType.STRING)을 적용하고 DB 타입을 VARCHAR로 변경하여 매직 넘버 의존성 완벽 제거.

성과: '결제 완료되었으나 픽업 취소됨'과 같은 복합적인 예외 상황에서 발생할 수 있는 데이터 모순(Contradiction)을 원천 차단하고, 새로운 결제 수단이 추가되어도 기존 주문 로직을 수정할 필요가 없는 안전하고 유연한 아키텍처 구축.

### 📊 도메인 상태 관리 설계 (State Management)

비즈니스 로직의 복잡도를 제어하기 위해 주문(Order)과 결제(Payment)의 상태를 분리하여 설계했습니다.
"과거에는 이 두 가지 흐름을 하나의 숫자형(TINYINT) 컬럼으로 관리하여 매직 넘버 지옥과 예외 처리 불가 문제를 겪었습니다. 이를 Order와 Payment 두 개의 도메인으로 분리하고 상태를 재정의하여, '노쇼 시 환불 불가' 같은 복잡한 비즈니스 정책을 안전하게 수용할 수 있는 아키텍처를 완성했습니다."

#### 1. 주문 상태 (Order Status)
```mermaid
stateDiagram-v2
    %% 시작점
    [*] --> PENDING_PAYMENT : 주문 생성 (장바구니 결제 요청)

    %% 결제 대기 단계
    
    PENDING_PAYMENT --> CANCELED : 유저 주문 직접 취소
    PENDING_PAYMENT --> CANCELED : 유저 결제창 이탈 / 승인 실패
    PENDING_PAYMENT --> READY_FOR_PICKUP : 토스페이먼츠 승인 완료
    note right of READY_FOR_PICKUP
      [QR 발급]은 상태 변화가 아님. 
      단순히 현재 상태와 만료시간을 JWT로 암호화하여 화면에 보여줄 뿐!
      (DB 접근 X)
    end note

    %% 픽업 대기 단계 (핵심 비즈니스 로직 분기)
    READY_FOR_PICKUP --> COMPLETED : 손님 방문 (사장님이 '픽업 완료' 처리)
    READY_FOR_PICKUP --> CANCELED : 사장님 직권 취소 (재고 파손 등, 100% 환불)
    READY_FOR_PICKUP --> NO_SHOW : 미방문 (자정 경과 배치 작업, 환불 불가)

    %% 종료점
    COMPLETED --> [*]
    CANCELED --> [*]
    NO_SHOW --> [*]
```

#### 2. 결제 상태 (Payment Status)
```mermaid
stateDiagram-v2
    %% 시작점
    [*] --> READY : 결제창 호출 및 주문 번호 채번

    %% 승인 단계
    READY --> PAID : PG사 결제 승인 성공 (입금 완료)
    READY --> FAILED : PG사 결제 승인 실패 (한도 초과, 잔액 부족 등)

    %% 환불 단계
    PAID --> CANCELED : 주문 취소 발생 (사장님 직권 취소로 인한 PG사 환불 API 호출 성공)

    %% 종료점
    PAID --> [*]
    FAILED --> [*]
    CANCELED --> [*]
```

## 📖 API Documentation & Specification
프론트엔드 협업 효율성 및 문서 신뢰성을 위해 **OpenAPI 3.0(Swagger/Redoc)** 기반의 자동화된 명세 시스템을 구축했습니다.

*   **API 명세서 (Redoc):** [https://maenggyuno.github.io/dongnekok-docs/](https://maenggyuno.github.io/timedeal-platform-docs/)
*   **설계 포인트:** 
    *   **전역 에러 체계 문서화:** `GlobalExceptionHandler`와 연동된 공통 에러 코드(ErrorCode)를 명시하여 프론트엔드 예외 처리 가이드 제공
    *   **협업 최적화:** 모든 API 응답에 대한 명확한 스키마(Schema)와 예시 데이터(Example)를 정의하여 커뮤니케이션 비용 최소화
    *   <img width="582" height="618" alt="image" src="https://github.com/user-attachments/assets/f2c78659-d926-4060-a533-f163e02be1cc" />


---

### [시퀀스 다이어그램]

### 💳 주문/결제 및 오프라인 QR 픽업 통합 흐름
```mermaid
---
title: 주문/결제 및 오프라인 QR 픽업 통합 흐름
---

sequenceDiagram
autonumber
actor User as 구매자(프론트엔드)
participant BE as 백엔드(Spring)
participant DB as Database/Redis
participant PG as 결제사(Toss)
actor Seller as 사장님(앱)

rect rgb(240, 248, 255)
    note right of User: 1. 주문 생성 및 결제 준비 (tossOrderId 분리)
    User->>BE: [POST] /api/v2/orders <br> (장바구니/바로구매 상품 정보)
    BE->>DB: 빵 재고 조회 및 차감 (동시성 제어)
    BE->>DB: 주문(Order) 데이터 생성 (PENDING)
    BE-->>User: 201 Created (내부 orderId: 105 반환)
    note right of User: 1. 결제 재료 준비 (우리 백엔드 API)
    User->>BE: [GET] /api/v2/orders/105/payment <br> (결제 위젯용 데이터 요청)
    BE->>BE: tossOrderId (유추 불가능한 난수) 생성
    BE-->>User: 200 OK <br> (tossOrderId, amount 반환)
end

rect rgb(255, 235, 238)
    note right of User: 2. 토스 결제창 호출 (프론트 SDK) 및 검증
    note over User, PG: 💻 프론트엔드 JS: tossPayments.requestPayment(...) 실행
    User->>PG: 토스 결제창 UI 호출 (tossOrderId, amount 전달)
    PG-->>User: 고객 인증 완료 (paymentKey 반환)

    note over User, BE: 🛡️ 여기서부터 최종 승인 단계
    User->>BE: [POST] /api/v2/payments/confirm <br> (우리 서버로 paymentKey, amount 전달)
    BE->>DB: DB의 total_price와 프론트가 보낸 amount 대조 (위변조 방어)

    alt 금액 불일치
        BE-->>User: 400 Bad Request (결제 차단)
    else 금액 일치 (검증 통과)
        note over BE, PG: 🔒 Spring Boot ➔ Toss Server (Secret Key 사용)
        BE->>PG: [POST] <https://api.tosspayments.com/v1/payments/confirm> <br> (paymentKey, orderId, amount 전송)
        PG-->>BE: 200 OK (최종 결제 승인 및 영수증 반환)
        BE->>DB: 결제 데이터 생성 및 주문 상태 변경 (READY_FOR_PICKUP)
        BE-->>User: 200 OK (우리 서비스 결제 성공 응답)
    end
end

rect rgb(255, 250, 205)
    note right of User: 3. 장바구니 뒷정리
    opt 장바구니 구매인 경우
        BE->>DB: 결제 완료된 상품 장바구니에서 삭제 (Soft Delete)
    end
end

rect rgb(240, 255, 240)
    note right of User: 4. 오프라인 매장 방문 및 QR 발급
    User->>BE: [POST] /api/v2/orders/105/qr <br> (일회용 토큰 요청)
    BE->>DB: 주문 조회 및 상태 검증

    alt 정상 상태
        BE->>BE: 일회용 QR 토큰 생성 (CPU 작업)
        BE->>DB: 생성된 토큰 Redis에 저장 (TTL: 3분, I/O 작업)
        BE-->>User: 200 OK { qrToken, expiresAt }
        User->>User: QR 코드 화면에 렌더링
    end
end

rect rgb(245, 245, 245)
    note right of User: 5. 픽업 스캔 완료
    User->>Seller: 사장님께 QR 코드 제시
    Seller->>BE: [POST] /api/v2/orders/pickup <br> (qrToken 전송)
    BE->>DB: Redis 토큰 검증 및 DB 상태 변경 (PICKED_UP)
    BE-->>Seller: 200 OK (픽업 성공)
end
```

<br>
💳 주문 취소 및 토스페이먼츠 환불 흐름

```mermaid 
---
title: 주문 취소 및 토스페이먼츠 환불 흐름
---

sequenceDiagram
autonumber
actor User as 구매자(프론트)
participant BE as 백엔드(Spring)
participant DB as Database
participant PG as 결제사(Toss)

note right of User: 프론트엔드는 취소할 상품 목록(cancelItems)을 담아 단일 API 호출
User->>BE: [POST] /api/v2/orders/{orderId}/cancel <br> { cancelItems, cancelReason }

note over BE, DB: 🛡️ 동시성 방어: 배타적 락(Pessimistic Lock) 적용
BE->>DB: 주문(Order) 및 상세 내역(OrderItem) 조회
BE->>BE: 취소 요청된 상품들의 총합 금액(cancelAmount) 계산

alt 상태가 PENDING (결제 승인 전)
    rect rgb(240, 248, 255)
        BE->>DB: 주문 및 해당 상품 상태 CANCELED 변경
        BE->>DB: 취소된 빵 재고 롤백 (+N)
        BE-->>User: 200 OK (결제 전 단순 취소 완료)
    end

else 상태가 READY_FOR_PICKUP (결제 완료됨)

    alt 🅰️ 전액 취소 (cancelAmount == totalAmount)
        rect rgb(255, 235, 238)
            note over BE, PG: URL은 동일하지만 Body에 cancelAmount를 빼고 보냄
            BE->>PG: [POST] <https://api.tosspayments.com/v1/payments/{paymentKey}/cancel> <br> Body: { "cancelReason": "..." }
            PG-->>BE: 200 OK (전액 환불 성공)
            BE->>DB: 💡 [전액 취소] 전체 주문(Order) 상태 ➔ CANCELED
            BE->>DB: 💡 전체 상세 상품(OrderItem) 상태 ➔ CANCELED
            BE->>DB: 모든 빵 재고 롤백 (+N)
        end

    else 🅱️ 부분 취소 (cancelAmount < totalAmount)
        rect rgb(255, 250, 205)
            note over BE, PG: Body에 취소할 금액(cancelAmount)을 명시해서 보냄
            BE->>PG: [POST] <https://api.tosspayments.com/v1/payments/{paymentKey}/cancel> <br> Body: { "cancelReason": "...", "cancelAmount": 2000 }
            PG-->>BE: 200 OK (부분 환불 성공)
            BE->>DB: 💡 [부분 취소] 전체 주문(Order) 상태 ➔ 유지 (READY_FOR_PICKUP)
            BE->>DB: 💡 취소 요청된 상세 상품(OrderItem)만 상태 ➔ CANCELED
            BE->>DB: 부분 취소된 빵만 재고 롤백 (+N)
        end
    end

    BE-->>User: 200 OK (환불 완료 및 남은 픽업 안내)

else 이미 픽업이 완료되었거나 이미 취소된 상태
    rect rgb(245, 245, 245)
        BE-->>User: 409 Conflict 또는 400 Bad Request (취소 불가 상태 예외)
    end
end
```
<br>
🌐 OAuth 2.0 소셜 로그인 및 토큰 생명주기

```mermaid
---
title: OAuth 2.0 소셜 로그인 및 토큰 생명주기 전체 흐름
---
sequenceDiagram
    autonumber
    actor User as 사용자
    participant FE as 프론트엔드(브라우저)
    participant BE as 백엔드(Spring)
    participant DB as Database/Redis
    participant OAuth as 소셜 서버(네이버/구글)

    %% ==========================================
    %% [대분류 1] 소셜 로그인 및 자동 회원가입
    %% ==========================================
    rect rgb(240, 248, 255)
        note right of User: 🟢 [대분류 1] 소셜 로그인 및 자동 회원가입
        note right of User: 1-1. 소셜 인증 요청 (프론트 ➔ 네이버 다이렉트 호출)
        User->>FE: '네이버로 로그인' 버튼 클릭
        note over FE, OAuth: GET nid.naver.com/oauth2.0/authorize?client_id=...&response_type=code
        FE->>OAuth: 네이버 로그인 페이지로 브라우저 이동
        OAuth-->>User: 초록색 네이버 로그인 팝업창 표시 (최초 1회만 동의 창 포함)
        User->>OAuth: ID/PW 입력 및 정보 제공 동의
        OAuth-->>FE: 🔑 인증 코드(Code)를 쿼리 파라미터로 달아 프론트로 리다이렉트
    end

    rect rgb(255, 240, 245)
        note right of User: 1-2. 토큰 교환 및 소셜 프로필 조회 (백엔드의 뒷단 통신)
        note over FE, BE: 프론트는 네이버에서 받아온 Code를 드디어 우리 백엔드로 전달!
        FE->>BE: [POST] /api/v2/auth/tokens <br> { "provider": "NAVER", "code": "abc1234..." }

        note over BE, OAuth: 🔒 백엔드 ➔ 소셜 서버 직접 통신 (Secret Key 사용)
        BE->>OAuth: [POST] Code를 주면서 네이버 Access Token 요청
        OAuth-->>BE: 네이버 Access Token 발급

        BE->>OAuth: [GET] 네이버 Access Token으로 유저 프로필(이메일, 이름 등) 요청
        OAuth-->>BE: 유저 프로필 정보 (이메일, 고유 식별자 등) 반환
    end

    rect rgb(240, 255, 240)
        note right of User: 1-3. 동네콕 자동 회원가입 및 로그인 처리 (하나의 API 안에서 발생)
        BE->>DB: 백엔드 내부 로직: 소셜 식별자로 DB 조회

        alt DB에 없는 신규 유저인 경우 (조용히 자동 가입)
            BE->>DB: 네이버에서 받은 이름/이메일로 User 새로 저장 (Role: USER)
        end

        note over BE, DB: 기존/신규 유저 공통 로직 (자체 토큰 발급)
        BE->>BE: 🔑 동네콕 전용 Access Token & Refresh Token 생성
        BE->>DB: Refresh Token을 Redis에 저장 (RTR 보안 용도)

        note over FE, BE: 프론트는 이게 가입인지 로그인인지 알 필요 없음!
        BE-->>FE: 200 OK <br> (Body: AccessToken / Cookie: RefreshToken)
        FE->>User: 메인 화면으로 렌더링 (로그인 성공)
    end

    %% ==========================================
    %% [대분류 2] 토큰 재발급
    %% ==========================================
    rect rgb(255, 250, 205)
         note right of User: 🟡 [대분류 2] 토큰 자동 재발급 (Access Token 수명 만료 시)
         note right of User: 2-1. API 실패 감지 및 몰래 재발급 처리
         User->>BE: [GET] /api/v2/orders (Header: 만료된 AccessToken)
         BE-->>User: 401 Unauthorized (토큰 만료 에러코드 반환)

         note over User, BE: 💡 프론트엔드 로직: 401 에러를 낚아채서 재발급 API 호출
         User->>BE: [POST] /api/v2/auth/reissue <br> (Cookie: RefreshToken 자동 전송)
         BE->>DB: Redis에서 RefreshToken 일치 여부 확인
         BE->>BE: 새로운 Access Token 및 Refresh Token 생성
         BE->>DB: 기존 RefreshToken 덮어쓰기 (RTR 기법)
         BE-->>User: 200 OK <br> (Body: 새 AccessToken / Cookie: 새 RefreshToken)

         User->>BE: 발급받은 새 토큰으로 아까 실패했던 API 다시 요청
         BE-->>User: 200 OK
    end

    %% ==========================================
    %% [대분류 3] 로그아웃
    %% ==========================================
    rect rgb(245, 245, 245)
         note right of User: ⚪ [대분류 3] 로그아웃
         note right of User: 3-1. 로그아웃 요청 및 토큰 블랙리스트 처리
         User->>User: '로그아웃' 버튼 클릭
         
         note over User, BE: 브라우저가 Refresh Token 쿠키를 알아서 같이 보냄!
         User->>BE: [DELETE] /api/v2/auth/tokens <br> (Header: AccessToken) <br> 🍪 (Cookie: RefreshToken)

         BE->>DB: Redis에서 해당 유저의 Refresh Token 영구 삭제
         BE->>BE: 현재 Access Token의 남은 만료 시간 계산
         BE->>DB: 남은 시간만큼 해당 Access Token을 Redis '블랙리스트'에 등록

         BE-->>User: 200 OK <br> (Cookie: Refresh Token 삭제 처리)
         User->>User: JS 메모리에 들고 있던 Access Token 폐기 후 메인 이동
    end
  
    %% ==========================================
    %% [대분류 4] 회원 탈퇴
    %% ==========================================
    rect rgb(255, 235, 238)
        note right of User: 🔴 [대분류 4] 회원 탈퇴
        note right of User: 4-1. 회원 탈퇴 요청 및 소셜 연동 해제
        User->>FE: '회원 탈퇴' 버튼 클릭 (경고 팝업 확인)
        FE->>BE: [DELETE] /api/v2/users/me <br> (Header: AccessToken) <br> 🍪 (Cookie: RefreshToken)
        BE->>BE: Access Token 검증 및 유저 식별
        
        BE->>DB: 해당 유저 데이터 삭제 (또는 Soft Delete 처리)
        
        opt 소셜 연동 해제 (Unlink)
            note over BE, OAuth: 현업 Best Practice: 네이버 측에도 "이 유저랑 앱 연동 끊어줘"라고 통보
            BE->>OAuth: [POST] 네이버/구글 연동 해제 API 호출
            OAuth-->>BE: 연동 해제 완료 응답
        end
        
        note right of User: 4-2. 토큰 파기 및 프론트엔드 후처리 (로그아웃과 동일)
        BE->>DB: Redis에서 해당 유저의 Refresh Token 영구 삭제
        BE->>BE: 현재 Access Token의 남은 만료 시간 계산
        BE->>DB: Access Token을 Redis '블랙리스트'에 등록 (탈취 방어)
        BE-->>FE: 200 OK <br> (Cookie: RefreshToken 삭제 처리)
        
        FE->>FE: JS 메모리에 들고 있던 Access Token 폐기
        FE->>User: 메인 화면으로 리다이렉트 (탈퇴 완료)
    end
```
<br>
🌐 로컬 인증(이메일/비밀번호) 전체 생명주기 및 토큰 관리 흐름

```mermaid
---
title: 로컬 인증(이메일/비밀번호) 전체 생명주기 및 토큰 관리 흐름
---
sequenceDiagram
    autonumber
    actor User as 사용자
    participant FE as 프론트엔드(브라우저)
    participant BE as 백엔드(Spring Security)
    participant DB as Database(MySQL)
    participant Redis as Redis(In-Memory)

    %% ==========================================
    %% [대분류 1] 로컬 회원가입 (BCrypt 암호화 및 자동 로그인)
    %% ==========================================
    rect rgb(240, 248, 255)
        note right of User: 🔵 [대분류 1] 로컬 회원가입 및 자동 로그인
        User->>FE: 가입 정보(이메일, 비밀번호, 이름 등) 입력 후 가입 클릭
        FE->>BE: [POST] /api/v2/users <br> { "email": "...", "password": "...", ... }
        
        BE->>DB: 이메일 중복 가입 여부 확인
        alt 이미 존재하는 이메일
            BE-->>FE: 409 Conflict (중복된 이메일)
            FE->>User: "이미 가입된 이메일입니다." 표시
        else 사용 가능한 이메일
            note over BE, DB: 🔒 보안 핵심: 평문 비밀번호를 단방향 해시로 암호화
            BE->>BE: BCryptPasswordEncoder.encode(password)
            BE->>DB: 유저 정보 및 해시된 비밀번호 INSERT
            
            note over BE, Redis: 자동 로그인 (소셜과 동일한 토큰 발급 로직)
            BE->>BE: 🔑 동네콕 전용 Access Token & Refresh Token 생성
            BE->>Redis: Refresh Token 저장 (TTL 설정)
            BE-->>FE: 201 Created <br> (Body: AccessToken / Cookie: RefreshToken)
            FE->>User: 메인 화면으로 리다이렉트 (가입 및 로그인 완료)
        end
    end

    %% ==========================================
    %% [대분류 2] 로컬 로그인
    %% ==========================================
    rect rgb(255, 240, 245)
        note right of User: 🔴 [대분류 2] 로컬 로그인 (자격 증명 검증)
        User->>FE: 이메일, 비밀번호 입력 후 '로그인' 버튼 클릭
        FE->>BE: [POST] /api/v2/auth/tokens <br> { "provider": "LOCAL", "email": "...", "password": "..." }
        
        BE->>DB: 전달받은 이메일로 유저 정보 및 해시 비밀번호 조회
        
        alt 유저가 없거나 비밀번호 불일치 (BCrypt.matches 검증)
            BE-->>FE: 401 Unauthorized (또는 404)
            FE->>User: "계정 정보가 일치하지 않습니다." 표시
        else 비밀번호 일치 (검증 통과)
            BE->>BE: 🔑 새로운 Access Token & Refresh Token 생성
            BE->>Redis: Refresh Token 저장 (기존 토큰 있으면 덮어쓰기)
            BE-->>FE: 200 OK <br> (Body: AccessToken / Cookie: RefreshToken)
            FE->>User: 메인 화면으로 리다이렉트 (로그인 성공)
        end
    end

    %% ==========================================
    %% [대분류 3] 토큰 재발급 (소셜/로컬 100% 공통 로직)
    %% ==========================================
    rect rgb(255, 250, 205)
         note right of User: 🟡 [대분류 3] 토큰 자동 재발급 (Access Token 수명 만료 시)
         User->>BE: [GET] /api/v2/orders (Header: 만료된 AccessToken)
         BE-->>User: 401 Unauthorized (토큰 만료 에러코드 반환)

         note over User, BE: 💡 프론트엔드 로직: 401 에러 감지 후 몰래 재발급 API 호출
         User->>BE: [POST] /api/v2/auth/reissue <br> 🍪 (Cookie: RefreshToken 자동 전송)
         BE->>Redis: Redis에서 RefreshToken 존재 및 일치 여부 확인
         BE->>BE: 새로운 Access Token 및 Refresh Token 생성
         BE->>Redis: 기존 RefreshToken 교체 (RTR 기법 적용)
         BE-->>User: 200 OK <br> (Body: 새 AccessToken / Cookie: 새 RefreshToken)

         User->>BE: 발급받은 새 토큰으로 실패했던 API 재요청
         BE-->>User: 200 OK (정상 데이터 반환)
    end

    %% ==========================================
    %% [대분류 4] 로그아웃 (소셜/로컬 100% 공통 로직)
    %% ==========================================
    rect rgb(245, 245, 245)
         note right of User: ⚪ [대분류 4] 로그아웃 (토큰 블랙리스트 처리)
         User->>User: '로그아웃' 버튼 클릭
         
         User->>BE: [DELETE] /api/v2/auth/tokens <br> (Header: AccessToken) <br> 🍪 (Cookie: RefreshToken)

         BE->>Redis: 1. 해당 유저의 Refresh Token 영구 삭제
         BE->>BE: 2. 현재 Access Token의 남은 만료 시간 계산
         BE->>Redis: 3. 남은 시간만큼 Access Token을 '블랙리스트'에 등록

         BE-->>User: 200 OK <br> (Cookie: Refresh Token 만료 처리)
         User->>User: JS 메모리에 들고 있던 Access Token 폐기 후 홈 이동
    end
  
    %% ==========================================
    %% [대분류 5] 회원 탈퇴 (소셜 연동 해제 과정 없음)
    %% ==========================================
    rect rgb(240, 255, 240)
        note right of User: 🟢 [대분류 5] 로컬 회원 탈퇴
        User->>FE: '회원 탈퇴' 버튼 클릭 (경고 팝업 확인)
        FE->>BE: [DELETE] /api/v2/users/me <br> (Header: AccessToken) <br> 🍪 (Cookie: RefreshToken)
        
        note over BE, DB: 💡 소셜 회원은 여기서 OAuth Unlink 과정이 추가되지만, 로컬은 생략됨
        BE->>DB: 해당 유저 데이터 삭제 (또는 Soft Delete `deleted_at` 업데이트)
        
        note right of User: 탈퇴 시 로그아웃과 동일한 토큰 파기 처리 진행
        BE->>Redis: Refresh Token 영구 삭제
        BE->>Redis: Access Token을 '블랙리스트'에 등록
        BE-->>FE: 200 OK <br> (Cookie: RefreshToken 삭제 처리)
        
        FE->>FE: JS 메모리에 들고 있던 Access Token 폐기
        FE->>User: 메인 화면으로 리다이렉트 (탈퇴 완료)
    end
```


<br>

## 🛠️ 3. Tech Stack
| 분류 | 기술 스택 |
| --- | --- |
| **Backend** | Java 21, Spring Boot, Spring Security, Spring Data JPA |
| **Database** | MySQL 8.0, Redis |
| **Infra & DevOps:** | AWS (EC2, S3, CloudFront, EventBridge, Lambda), Cloudflare (DNS, Tunnel), Docker, GitHub Actions |
| **Frontend** | React, Axios |

<br>

## ✨ 4. Key Features
* **타임딜 핵심 로직:** 유통기한 임박 상품의 실시간 재고 관리 및 타임 세일 자동화
* **위치 기반 서비스:** 사용자 위치 기반의 하이퍼로컬 상점 큐레이션 및 지도 연동
* **보안 특화:** Cloudflare Zero Trust 기반의 인프라 보호 및 서버리스 무결성 검증 파이프라인
* **관리자 백오피스:** 상점주를 위한 직관적인 대시보드 및 주문 처리 시스템

## 🔥 5. Key Experience & Troubleshooting

### 1️⃣ 린 스타트업 기반 MVP 검증 및 비즈니스 문제 해결 [[🔗 MVP 검증 기록](https://velog.io/@mgo0415/%EB%8F%99%EB%84%A4%EC%BD%95-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%BD%94%EB%94%A9%EB%B3%B4%EB%8B%A4-%EB%A8%BC%EC%A0%80-%EB%8F%99%EB%84%A4-%EB%B9%B5%EC%A7%91-%EC%82%AC%EC%9E%A5%EB%8B%98%EB%93%A4%EA%B3%BC-MOU%EB%A5%BC-%EB%A7%BA%EB%8B%A4-MVP-%EA%B3%A0%EA%B0%9D-%EA%B2%80%EC%A6%9D)]
* **문제:** 타임딜 커머스 특성상 특정 시간대 트래픽 집중이 예상되나, 실제 수요 검증 없이 오버엔지니어링될 위험 인지
* **해결:** 로컬 베이커리의 '당일 폐기 원가 손실'을 타겟으로 한 상생 MOU 의향서를 직접 작성해 현장 영업 및 파트너사 선제 확보
* **성과:** 실제 사용자의 Pain Point를 반영한 직관적 UI 설계 및 마감 시간 트래픽 방어를 위한 백엔드 아키텍처의 비즈니스적 당위성 확립

### 2️⃣ S3 투트랙(Two-Track) 아키텍처 및 Event-Driven 무결성 검증 [[🔗 아키텍처 설계 문서](https://velog.io/@mgo0415/Refactoring-%EC%A1%B8%EC%97%85%EC%9E%91%ED%92%88%EC%9D%98-%ED%95%9C%EA%B3%84%EB%A5%BC-%EB%84%98%EC%96%B4-S3-%EB%B9%84%EC%9A%A9-%ED%9A%A8%EC%9C%A8%ED%99%94%EC%99%80-%EB%B3%B4%EC%95%88-%EB%91%90-%EB%A7%88%EB%A6%AC-%ED%86%A0%EB%81%BC-%EC%9E%A1%EA%B8%B0-feat.-Tunnel-CDN)]
* **문제:** 단일 S3 Presigned URL 사용 시 썸네일 캐싱 불가로 인한 비용 낭비와, 클라이언트 직업로드 시 보안 위협 식별
* **해결:** 공개 이미지(CDN)와 민감 문서(Presigned URL)의 접근 방식을 분리하고, AWS Lambda와 EventBridge를 연동한 무결성(매직 넘버, 크기) 검사 로직 구현
* **성과:** 백엔드 I/O 부하 0% 유지 및 불필요한 스토리지 비용/런타임 보안 위협 방어

### 3️⃣ Zero Trust 인프라 및 백오피스 논리적 망 분리 [[🔗 인프라 구축기](https://velog.io/@mgo0415/Refactoring-%EC%8B%A0%EC%9E%85%EC%9D%98-%EC%8B%9C%EC%84%A0-%EB%B0%B1%EC%98%A4%ED%94%BC%EC%8A%A4Admin-%EA%B5%AC%EC%B6%95%EA%B3%BC-Zero-Trust-%EB%B3%B4%EC%95%88-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%84%A4%EA%B3%84%EC%9D%98-%EC%97%AC%EC%A0%95)]
* **문제:** Admin 페이지 구축 시, 서버 IP 노출로 인한 DDoS 및 포트 스캐닝 위협 차단 필요
* **해결:** AWS EC2 인바운드 포트를 전면 차단하고 Cloudflare Tunnel을 도입해 공격 표면(Attack Surface) 원천 제거, OTP 인증 정책 적용
* **성과:** 사내 VPN 구축 비용 없이 논리적 망 분리 효과 달성 및 고정 IP 유지 비용 절감

### 4️⃣ 우아한 기능 저하(Graceful Degradation)를 통한 무중단 UX 제공 [[🔗 트러블슈팅 기록](https://velog.io/@mgo0415/Troubleshooting-AWS-%EB%B9%84%EC%9A%A9-0%EC%9B%90-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%9C%EB%B2%84%EA%B0%80-%EA%BA%BC%EC%A0%B8%EB%8F%84-%EA%B3%A0%EC%9E%A5-%EB%82%9C-%EC%82%AC%EC%9D%B4%ED%8A%B8%EC%B2%98%EB%9F%BC-%EB%B3%B4%EC%9D%B4%EC%A7%80-%EC%95%8A%EA%B2%8C-%ED%95%98%EA%B8%B0)]
* **문제:** 클라우드 비용 절감을 위한 서버 중단 시, 백엔드 다운으로 인해 프론트엔드 UI가 깨지고 502/CORS 에러 노출
* **해결:** 앱 초기화 시 경량 `HEAD` 요청으로 서버 헬스체크를 수행하는 HOC(MaintenanceGuard) 패턴 구현 및 Spring Security 403/CORS 에러 오버라이딩 해결
* **성과:** 추가 인프라 비용 없이 서버 OFF 상태에서도 안정적인 '점검 중' 화면 제공

### 5️⃣ 리버스 프록시(Nginx) 라우팅 충돌 해결 및 OAuth2 커스터마이징 [[🔗 트러블슈팅 기록](https://velog.io/@mgo0415/%ED%8A%B8%EB%9F%AC%EB%B8%94%EC%8A%88%ED%8C%85-Cloudflare-Tunnel-Docker-Nginx-Spring-Boot-OAuth2-404-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0%EA%B8%B0)]
* **문제:** Nginx 리버스 프록시 환경에서 소셜 로그인 리다이렉트 시 404 에러 연쇄 발생
* **해결:** Spring Security의 기본 엔드포인트를 Nginx Prefix(`/api`) 라우팅 규칙에 맞게 동적으로 오버라이딩 처리
* **성과:** 프론트엔드-프록시-백엔드 통신 흐름을 단일화하여 안정적인 API 네트워크 통신망 구축

<br>

## 🤝 6. Collaboration & Team Workspace

단순한 코드 공유를 넘어, 실제 비즈니스 환경과 동일한 협업 파이프라인을 구축하여 프로젝트를 운영했습니다.

* **Issue Template & PR Convention:** 기능 추가, 버그 수정 등 목적에 맞는 템플릿을 고도화하여 리뷰어의 맥락 파악 시간을 단축했습니다.
* ### 📌 체계적인 이슈 추적 및 문서화 (Issue Tracker & Convention)

<p align="center">
  <img src="https://github.com/user-attachments/assets/0d613762-cbc6-408d-b723-97eda6512a1b" width="48%" alt="이슈 목록">
  <img src="https://github.com/user-attachments/assets/fbb4eb3f-8c76-41c2-bc63-9cc843b39984" width="48%" alt="상세 이슈 템플릿">
</p>

* **커밋/이슈 컨벤션:** `[FEAT]`, `[Refactor]`, `[Infra]` 등 직관적인 말머리와 라벨링을 통해 작업의 성격을 명확히 분리하고 추적성을 높였습니다.
* **상세 템플릿:** 단순한 버그 수정이나 기능 추가를 넘어, '배경/목적 - 기술 스펙 - 체크리스트'로 이어지는 상세 템플릿을 사용하여 작업 전 설계의 타당성을 검증했습니다.


### 🤝 7. Collaboration & Workspace (문서화 및 에셋 관리)
단순한 코드 공유를 넘어, 실제 스타트업 환경과 동일한 수준의 체계적인 문서화 및 자산 관리 파이프라인을 구축했습니다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/6c9a7b35-441c-4f54-a01b-08ccdf688eb7" width="48%" alt="노션 워크스페이스">
  <img src="https://github.com/user-attachments/assets/07c8a1ba-6646-4b87-acd0-ccaa2b70d1c3" width="48%" alt="구글 드라이브 에셋 관리">
</p>

* **Notion 기반 사내 위키:** 기술(Architecture, ERD, API) 문서뿐만 아니라 기획, 영업(상생 협력 의향서) 등 목적별로 문서를 카테고리화하여 중앙 집중식으로 관리합니다.
* **Google Drive 자산 동기화:** 아키텍처 원본 파일(.drawio), 데이터베이스 산출물 등을 기술 개발 전용 폴더 체계(01_Architecture ~ 04_Infra)로 세분화하여 안전하게 보관 및 링크 연동하고 있습니다.
* **추후 고도화 계획 (To-be):** Jira 티켓 기반의 스프린트 관리 및 Slack 연동을 통한 CI/CD 배포 자동화 알림 파이프라인 구축 (진행 예정)

<img width="1545" height="813" alt="image" src="https://github.com/user-attachments/assets/2cf38854-d019-4594-a629-688cd2766fad" />
<img width="524" height="750" alt="image" src="https://github.com/user-attachments/assets/1115f5d2-a2c9-42fd-a799-16681433601a" />

