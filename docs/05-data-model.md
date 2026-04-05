# 05. 데이터 모델 (초안)

## 설계 원칙

- **멀티테넌트**: 모든 테이블에 `tenant_id` 컬럼 포함(공유 DB 전략 기준)
- **한 테넌트 = 여러 스토어(매장)** 허용 구조
- 소프트 삭제(`deleted_at`) 적용 원칙
- 감사 로그를 위해 `created_at`, `updated_at`, `created_by` 포함

## 핵심 엔티티 목록

### 1. 테넌시 / 사용자
- **Tenant** — 가입 사업자 단위
- **Store** — 실제 매장(한 테넌트에 1~N)
- **User** — 로그인 계정(사장/매니저/직원/고객)
- **Role** — 역할 정의
- **Permission** — 세부 권한
- **StoreMember** — 스토어별 직원 연결 + 역할

### 2. 메뉴
- **Category** — 메뉴 카테고리(다단계)
- **Menu** — 메뉴 항목
- **MenuImage** — 메뉴 사진
- **OptionGroup** — 옵션 그룹(예: 맵기)
- **Option** — 개별 옵션 값
- **MenuOptionGroup** — 메뉴-옵션그룹 매핑
- **Recipe** — 메뉴-식자재 매핑(소요량)

### 3. 테이블 / 주문
- **Table** — 테이블 정보(위치, 인원, QR 토큰)
- **TableLayout** — 배치도(층/구역)
- **Order** — 주문 헤더
- **OrderItem** — 주문 상세(메뉴 + 수량)
- **OrderItemOption** — 주문별 옵션 선택
- **OrderEvent** — 주문 상태 변경 이력

### 4. 결제
- **Payment** — 결제 기록
- **PaymentMethod** — 결제 수단
- **Refund** — 환불 기록
- **Discount** — 할인 적용 내역

### 5. 재고
- **Ingredient** — 식자재
- **IngredientCategory** — 식자재 카테고리
- **Supplier** — 거래처
- **StockMovement** — 입출고 이력
- **StockLot** — 로트(유통기한 단위)
- **PurchaseOrder** — 발주서
- **PurchaseOrderItem** — 발주 품목
- **StockCheck** — 재고 실사 기록

### 6. 인사
- **Employee** — 직원 인사정보(User와 1:1)
- **Shift** — 스케줄(근무 배정)
- **Attendance** — 출퇴근 기록
- **PayrollSummary** — 근무시간 집계(월별)

### 7. 예약
- **Reservation** — 예약
- **ReservationChannel** — 예약 경로(수동/온라인/네이버/카카오)
- **Waitlist** — 대기 리스트 (TBD)

### 8. 고객 / 마케팅
- **Customer** — 고객 프로필
- **CustomerVisit** — 방문 이력(Order와 연결)
- **Point** — 포인트 잔액
- **PointTransaction** — 포인트 이력
- **Coupon** — 쿠폰 마스터
- **CouponIssue** — 개별 발행 쿠폰
- **CouponRedemption** — 사용 기록
- **Notification** — 알림 발송 이력
- **NotificationConsent** — 마케팅 수신동의

### 9. 시스템 공통
- **AuditLog** — 감사 로그
- **FileAsset** — 업로드 파일
- **Setting** — 가게별 설정

## 주요 관계 (텍스트 ERD)

```
Tenant 1──N Store
Store  1──N StoreMember N──1 User (Role)
Store  1──N Category 1──N Menu
Menu   N──M OptionGroup (via MenuOptionGroup)
OptionGroup 1──N Option
Menu   1──N Recipe N──1 Ingredient
Store  1──N Table
Store  1──N Order 1──N OrderItem N──1 Menu
OrderItem 1──N OrderItemOption N──1 Option
Order  1──N Payment
Order  N──1 Customer (optional)
Customer 1──N PointTransaction
Customer 1──N CouponIssue N──1 Coupon
Store  1──N Reservation N──1 Customer (optional)
Store  1──N Employee 1──N Shift
Employee 1──N Attendance
Ingredient 1──N StockMovement
Ingredient 1──N StockLot
Store  1──N PurchaseOrder 1──N PurchaseOrderItem N──1 Ingredient
```

## 인덱스 / 성능 고려

- 모든 조회 쿼리는 `(tenant_id, store_id, ...)` 복합 인덱스 사용
- `Order`: `(store_id, created_at)`, `(store_id, table_id, status)`
- `Menu`: `(store_id, category_id, display_order)`
- `Attendance`: `(employee_id, work_date)`
- `Reservation`: `(store_id, reservation_at)`

## 미정 사항 (TBD)

- ID 체계: UUID vs 숫자 PK
- 시간대 처리(TIMESTAMP WITH TIME ZONE vs LOCAL)
- 이력/버전 관리 방식(메뉴 가격 변경 이력 등)
- 파일 스토리지 경로 규칙
- 다국어 확장 시 컬럼 구조(현재는 한국어 단일이므로 보류)
