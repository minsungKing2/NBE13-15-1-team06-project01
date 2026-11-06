# NBE13-15-1-team06

> 1차 팀 프로젝트 6팀 · 카페 원두 메뉴/주문 관리 서비스

## 목차
1. [팀원별 담당 작업](#1-팀원별-담당-작업)
2. [기능 구현 현황](#2-기능-구현-현황)
3. [개발 환경](#3-개발-환경)
4. [DB 설정](#4-db-설정)
5. [DB 준비 및 초기 데이터](#5-db-준비-및-초기-데이터)
6. [실행 방법](#6-실행-방법)
7. [테스트 및 점검](#7-테스트-및-점검)
8. [평가 대상 및 선택 심화](#8-평가-대상-및-선택-심화)

---

## 1. 팀원별 담당 작업

| 팀원 | 담당 도메인 | 구현 API |
| --- | --- | --- |
| 정민성 | category | 카테고리 목록(A01), 상세(A02), 등록(A03), 수정(A04), 삭제(A05), 카테고리별 메뉴(A15) |
| 배재영 | menu | 전체 메뉴(A06), id 조회(A07), 생성(A08), 수정(A09), 삭제(A10), 판매 중 메뉴(A19), 메뉴명 검색(A20), 판매 여부 변경(A21) |
| 맹주석 | order | 주문 접수(A11), 이메일별 목록(A12), 상세(A13), 취소(A16), 메모 수정(A22), 재주문(A23) |
| 박성훈 | statistics | 배송 묶음(A14), 접수일별 주문 통계(A17), 메뉴 통계(A18), 고객 통계(A24) |

### 공동 구현 내역

| 항목 | 내용 | 담당 |
| --- | --- | --- |
| 예외 처리 | 공통 Exception 10개를 나누어 구현 | ⚠️ TODO: 팀원별로 맡은 Exception 이름 기재 |
| 초기 데이터 | `InitDataRunner` | ⚠️ TODO: 담당자 기재 |

---

## 2. 기능 구현 현황

### ✅ 완료 기능 (24개)

| 도메인 | 기능 |
| --- | --- |
| 카테고리 | 목록 조회, 상세 조회, 등록, 수정, 삭제, 카테고리별 메뉴 |
| 메뉴 | 전체 목록 조회, 상세 조회, 등록, 일반 수정, 삭제, 판매 중 메뉴, 메뉴명 검색, 판매 여부 변경 |
| 주문 | 접수, 이메일별 주문, 상세, 취소, 메모 수정, 재주문 |
| 배송·통계 | 배송 묶음, 접수일별 주문 통계, 접수일별 메뉴 통계, 접수일별 고객 통계 |

업무 규칙 자동 테스트
### ❌ 미완료 기능
- 실제 DB 롤백 저장 테스트
- Redis 메뉴 조회 캐시
- 별도 메뉴 페이징 API
- RestClient 외부 조회 및 실패 처리

### 🐞 미해결 오류
- 없음 

---

## 3. 개발 환경

| 항목 | 버전 |
| --- | --- |
| Java | 21 |
| Spring Boot | 4.1.2 |
| MySQL | 8.4.11 |

---

## 4. DB 설정

- 설정 파일: `src/main/resources/application.yaml`
- 접속 정보는 환경변수로 주입하며, **실제 비밀번호는 저장소에 올리지 않습니다.**

| 환경변수 | 예시 값 |
| --- | --- |
| `DB_URL` | `jdbc:mysql://127.0.0.1:3306/kdt_project1?allowPublicKeyRetrieval=true&sslMode=DISABLED` |
| `DB_USERNAME` | `root` |
| `DB_PASSWORD` | `kdtpass` |



---

## 5. DB 준비 및 초기 데이터

### 5-1. 데이터베이스 생성

```sql
CREATE DATABASE kdt_project1 DEFAULT CHARACTER SET utf8mb4;
USE kdt_project1;
```

### 5-2. 테이블 생성

<details>
<summary>테이블 4개 DDL 펼치기</summary>

```sql
CREATE TABLE categories (
    id   BIGINT       NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) COLLATE utf8mb4_bin NOT NULL,
    CONSTRAINT pk_categories PRIMARY KEY (id),
    CONSTRAINT uk_categories_name UNIQUE (name)
);

CREATE TABLE menus (
    id          BIGINT       NOT NULL AUTO_INCREMENT,
    name        VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
    price       INT          NOT NULL,
    description VARCHAR(300) NOT NULL,
    available   BOOLEAN      NOT NULL,
    category_id BIGINT       NOT NULL,
    CONSTRAINT pk_menus PRIMARY KEY (id),
    CONSTRAINT fk_category_id FOREIGN KEY (category_id) REFERENCES categories (id)
);

CREATE TABLE orders (
    id            BIGINT       NOT NULL AUTO_INCREMENT,
    email         VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
    address       VARCHAR(200) NOT NULL,
    postcode      VARCHAR(5)   NOT NULL,
    created_at    DATETIME     NOT NULL,
    updated_at    DATETIME,
    total_amount  BIGINT       NOT NULL,
    status        ENUM('PLACED', 'CANCELLED') NOT NULL,
    canceled_at   DATETIME,
    shipping_date DATE         NOT NULL,
    memo          VARCHAR(200) NOT NULL DEFAULT '',
    CONSTRAINT pk_orders PRIMARY KEY (id)
);

CREATE TABLE order_items (
    id          BIGINT       NOT NULL AUTO_INCREMENT,
    order_id    BIGINT       NOT NULL,
    menu_id     BIGINT       NOT NULL,
    menu_name   VARCHAR(100) NOT NULL,
    unit_price  BIGINT       NOT NULL,
    quantity    BIGINT       NOT NULL,
    line_amount BIGINT       NOT NULL,
    CONSTRAINT pk_order_items PRIMARY KEY (id),
    CONSTRAINT fk_order_items_order FOREIGN KEY (order_id) REFERENCES orders (id),
    CONSTRAINT fk_order_items_menu  FOREIGN KEY (menu_id)  REFERENCES menus (id)
);
```

</details>

### 5-3. 초기 데이터

스프링 애플리케이션 실행 시 `InitDataRunner`가 아래 데이터를 **자동으로 저장**합니다. 별도 INSERT는 필요 없습니다.

**categories**: 싱글오리진, 블렌더

**menus**

| 메뉴명 | 가격(원) | 설명 |
| --- | ---: | --- |
| Columbia Nariñó | 5,000 | 콜롬비아 나리뇨 원두를 담은 커피빈 패키지입니다. |
| Brazil Serra Do Caparaó | 7,000 | 브라질 세하 두 카파라오 원두를 담은 커피빈 패키지입니다. |
| Columbia Quindío (White Wine Extended Fermentation) | 9,000 | 콜롬비아 킨디오 원두를 담은 커피빈 패키지입니다. |
| Ethiopia Sidamo | 11,000 | 에티오피아 시다모 원두를 담은 커피빈 패키지입니다. |

---

## 6. 실행 방법

### 6-1. 백엔드 실행
1. MySQL을 실행하고 [5-1](#5-1-데이터베이스-생성), [5-2](#5-2-테이블-생성)를 완료합니다.
2. 프로젝트 루트에서 환경변수를 지정하고 실행합니다.

```bash
export DB_URL="jdbc:mysql://127.0.0.1:3306/kdt_project1?allowPublicKeyRetrieval=true&sslMode=DISABLED"
export DB_USERNAME=root
export DB_PASSWORD=your-password
./gradlew bootRun
```

백엔드 주소: `http://127.0.0.1:8080`

### 6-2. web 실행
1. 다른 터미널에서 제공된 `web` 폴더로 이동합니다.
2. 아래 명령을 실행합니다.

```bash
cd web
java -m jdk.httpserver -b 127.0.0.1 -p 8000
```

3. 브라우저에서 `http://127.0.0.1:8000/` 에 접속합니다.
4. **연결 설정 → 실제 API 연결**을 선택하고 API 서버 주소에 `http://127.0.0.1:8080` 을 입력합니다.
   > 주소 끝에 `/api/v1` 을 붙이지 않습니다.

---

## 7. 테스트 및 점검

### 7-1. 테스트 실행

```bat
gradlew.bat test --tests "com.kdt.nbe13151team06.categories.CategoriesApiResponseControllerTest"
gradlew.bat test --tests "com.kdt.nbe13151team06.categories.CategoriesApiControllerTest"
gradlew.bat test --tests "com.kdt.nbe13151team06.menus.MenuApiControllerTest"
gradlew.bat test --tests "com.kdt.nbe13151team06.orders.OrderApiControllerTest"
```

**테스트 케이스 (총 7개)**

| API | 검증 내용 | 기대 응답 | 테스트 클래스 |
| --- | --- | --- | --- |
| A01 | 카테고리 목록 조회, ID 오름차순 | 200 · Category[] |  CategoriesApiponseControllerTest|
| A02 | 카테고리 상세 조회 | 200 · Category |  CategoriesApiponseControllerTest|
| A05 | 메뉴가 없는 카테고리 삭제 | 204 · 빈 본문 | CategoriesApiponseControllerTest |
| A07 | 없는 메뉴 조회 | 404 · `MENU_NOT_FOUND` | MenuApiContollerTest|
| A08 | 가격 상한 초과 메뉴 등록 | 400 · `VALIDATION_ERROR` (`fieldErrors.price`) | MenuApiControllerTest|
| A11 | 이메일 형식이 틀린 주문 접수 | 400 · `VALIDATION_ERROR` (`fieldErrors.email`) | OrderApiControllerTest|
| A11 | 같은 이메일·배송일에 다른 배송지로 접수 | 409 · `ADDRESS_MISMATCH` | OrderApiControllerTest|



### 7-2. HTTP 요청 점검

| 항목 | 내용 |
| --- | --- |
| 파일 경로 | `requests/api-check.http` |
| 변수 | `@baseUrl = http://127.0.0.1:8080`  |
| 실행 순서 | 파일 위에서부터 차례로 실행 (정상 24개 → 오류 6개) |

**점검 대상 API (정상 요청 24개)**

| ID | 기능 | Method | 경로 | 응답 |
| --- | --- | --- | --- | --- |
| A01 | 카테고리 목록 | GET | `/api/v1/categories` | 200 · Category[] |
| A02 | 카테고리 상세 | GET | `/api/v1/categories/{categoryId}` | 200 · Category |
| A03 | 카테고리 등록 | POST | `/api/v1/categories` | 201 · Category |
| A04 | 카테고리 수정 | PUT | `/api/v1/categories/{categoryId}` | 200 · Category |
| A05 | 카테고리 삭제 | DELETE | `/api/v1/categories/{categoryId}` | 204 · 빈 본문 |
| A06 | 전체 메뉴 | GET | `/api/v1/menus` | 200 · Menu[] |
| A07 | 메뉴 상세 | GET | `/api/v1/menus/{menuId}` | 200 · Menu |
| A08 | 메뉴 등록 | POST | `/api/v1/menus` | 201 · Menu |
| A09 | 메뉴 일반 수정 | PUT | `/api/v1/menus/{menuId}` | 200 · Menu |
| A10 | 메뉴 삭제 | DELETE | `/api/v1/menus/{menuId}` | 204 · 빈 본문 |
| A11 | 주문 접수 | POST | `/api/v1/orders` | 201 · Order |
| A12 | 이메일별 주문 | GET | `/api/v1/orders?email={email}` | 200 · OrderSummary[] |
| A13 | 주문 상세 | GET | `/api/v1/orders/{orderId}` | 200 · Order |
| A14 | 배송 묶음 | GET | `/api/v1/delivery-batches?shippingDate={shippingDate}` | 200 · DeliveryBatch[] |
| A15 | 카테고리별 메뉴 | GET | `/api/v1/categories/{categoryId}/menus` | 200 · Menu[] |
| A16 | 주문 취소 | POST | `/api/v1/orders/{orderId}/cancel` | 200 · 전체 Order |
| A17 | 접수일별 주문 통계 | GET | `/api/v1/statistics/orders?date={date}` | 200 · OrderStatistics |
| A18 | 접수일별 메뉴 통계 | GET | `/api/v1/statistics/menus?date={date}` | 200 · MenuStatistics[] |
| A19 | 판매 중 메뉴 | GET | `/api/v1/menus/available` | 200 · Menu[] |
| A20 | 메뉴명 검색 | GET | `/api/v1/menus/search?keyword={keyword}` | 200 · Menu[] |
| A21 | 판매 여부 변경 | PUT | `/api/v1/menus/{menuId}/availability` | 200 · 전체 Menu |
| A22 | 주문 메모 수정 | PUT | `/api/v1/orders/{orderId}/memo` | 200 · 전체 Order |
| A23 | 재주문 | POST | `/api/v1/orders/{orderId}/reorder` | 201 · 새 Order |
| A24 | 접수일별 고객 통계 | GET | `/api/v1/statistics/customers?date={date}` | 200 · CustomerStatistics[] |

**오류 요청 (6개)**

| ID  | 기능                                 | Method | 경로                    | 응답                    |
| --- | ------------------------------------ | ------ | ----------------------- | ----------------------- |
| A08 | 메뉴 등록 (가격 상한 초과)           | POST   | `/api/v1/menus`         | 400  VALIDATION_ERROR  |
| A11 | 주문 접수 (이메일 형식 오류)         | POST   | `/api/v1/orders`        | 400  VALIDATION_ERROR  |
| A07 | 메뉴 상세 (없는 번호)                | GET    | `/api/v1/menus/999999`  | 404  MENU_NOT_FOUND    |
| A10 | 메뉴 삭제 (주문에 사용된 메뉴)       | DELETE | `/api/v1/menus/1`       | 409  MENU_IN_USE                  |
| A11 | 주문 접수 (배송지 불일치)            | POST   | `/api/v1/orders`        | 409  ADDRESS_MISMATCH  |
| A05 | 카테고리 삭제 (메뉴가 등록된 카테고리) | DELETE | `/api/v1/categories/1`  | 409  CATEGORY_IN_USE   |

메뉴 삭제(409)와 배송지 불일치(409)는 앞의 정상 요청에서 buyer@example.com으로 1번 메뉴를 주문한 상태여야 재현됩니다. 정상 요청을 먼저 실행한 뒤 오류 요청을 실행합니다.

### 7-3. 시각 경계값 확인

모든 시각 기준은 **서울 시각(Asia/Seoul) 14:00:00**입니다. 경계 시각은 `OrderRules`에 시각 값을 직접 넣어 검증하므로, 실제 현재 시각과 관계없이 항상 같은 결과가 나옵니다.

**규칙**

| 규칙 | 기준 |
| --- | --- |
| 배송 대상일 | 14:00:00 **전** 접수 → 당일 배송, 14:00:00 **부터** 접수 → 다음 날 배송 |
| 주문 취소 마감 | 배송 대상일 14:00:00 **전**까지만 취소 가능 (전날에는 언제든 가능) |

**경계값 테스트**

| 대상 | 입력 시각 | 기대 결과 | 테스트 |
| --- | --- | --- | --- |
| 배송 대상일 | 11/3 `13:59:59` 접수 | 11/3 (당일) | `OrderRulesTest` |
| 배송 대상일 | 11/3 `14:00:00` 접수 | 11/4 (다음 날) | `OrderRulesTest` |
| 배송 대상일 | 11/3 `14:00:01` 접수 | 11/4 (다음 날) | `OrderRulesTest` |
| 배송 대상일 (연도 변경) | 12/31 `14:00:00` 접수 | 다음 해 1/1 | `OrderRulesTest` |
| 취소 마감 | 배송일 `13:59:59` | 취소 가능 | `OrderRulesTest`, `OrderServiceTest` |
| 취소 마감 | 배송일 `14:00:00` | 취소 불가 | `OrderRulesTest`, `OrderServiceTest` |
| 취소 마감 | 배송일 전날 `15:00:00` | 취소 가능 | `OrderRulesTest` |
| 마감 후 취소 요청 | 배송일이 지난 주문 | `ORDER_NOT_CANCELLABLE`, 주문은 `PLACED` 유지 | `OrderServiceTest` |
| 이미 취소된 주문 재취소 | 첫 취소 이후 다시 취소 | 첫 취소 시각(`canceledAt`) 유지 | `OrderServiceTest` |

**실행 명령**

```bat
gradlew.bat test --tests "com.kdt.nbe13151team06.orders.OrderRulesTest"
gradlew.bat test --tests "com.kdt.nbe13151team06.orders.OrderServiceTest"
```



## 8. 평가 대상 및 선택 심화

| 항목 | 내용 |
| --- | --- |
| 평가 브랜치 | `main` |
| 태그 / 커밋 | |
| 선택 심화 | 업무 규칙 자동 테스트 구현 |