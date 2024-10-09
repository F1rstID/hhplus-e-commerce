# 항해 + 이커머스

> ### Description
- `e-커머스 상품 주문 서비스`를 구현해 봅니다.
- 상품 주문에 필요한 메뉴 정보들을 구성하고 조회가 가능해야 합니다.
- 사용자는 상품을 여러개 선택해 주문할 수 있고, 미리 충전한 잔액을 이용합니다.
- 상품 주문 내역을 통해 판매량이 가장 높은 상품을 추천합니다.

> ### Requirements
- 아래 4가지 API 를 구현합니다.
   - 잔액 충전 / 조회 API
   - 상품 조회 API
   - 주문 / 결제 API
   - 인기 판매 상품 조회 API
- 각 기능 및 제약사항에 대해 단위 테스트를 반드시 하나 이상 작성하도록 합니다.
- 다수의 인스턴스로 어플리케이션이 동작하더라도 기능에 문제가 없도록 작성하도록 합니다.
- `동시성 이슈`를 고려하여 구현합니다.
- 재고 관리에 문제 없도록 구현합니다.

> ### 잔액 충전 (POST - /balance/charge)
- 사용자 식별자 및 충전할 금액을 받아 잔액을 충전합니다.
```mermaid
sequenceDiagram
    actor User
    participant Controller
    participant Usecase
    participant Service
    participant Repository
    participant DB

    User->>Controller: 충전 요청 (금액)
    Controller->>Usecase: 충전 요청 전달
    Usecase->>Service: 충전 로직 수행
    Service->>Repository: 사용자 조회 (UserId)
    Repository->>DB: 사용자 조회 (UserId)
    DB-->>Repository: 사용자 데이터
    Repository-->>Service: 사용자 존재 여부
    alt 사용자 존재
        Service->>Service: 충전 금액 유효성 검증
        Service->>Repository: 잔액 업데이트 (UserID, 금액)
        Repository->>DB: 잔액 업데이트 (UserID, 금액)
        DB-->>Repository: 업데이트 완료
        Repository-->>Service: 업데이트 결과
        Service-->>Usecase: 충전 성공 (새 잔액)
    else 사용자 미존재
        Service-->>Usecase: 에러 반환 (사용자 없음)
    end
    Usecase-->>Controller: 충전 결과 반환
    Controller-->>User: 충전 결과 응답
```


> ### 잔액 조회 (GET - /balance)
- 사용자 식별자를 통해 해당 사용자의 잔액을 조회합니다.
```mermaid
sequenceDiagram
    actor User
    participant Controller
    participant Usecase
    participant Service
    participant Repository
    participant DB

    User->>Controller: 잔액 조회 요청
    Controller->>Usecase: 잔액 조회 요청 전달
    Usecase->>Service: 잔액 조회 로직 수행
    Service->>Repository: 사용자 조회 (UserId)
    Repository->>DB: 사용자 조회 (UserId)
    DB-->>Repository: 사용자 데이터
    Repository-->>Service: 사용자 존재 여부
    alt 사용자 존재
        Service->>Repository: 잔액 조회 (UserId)
        Repository->>DB: 잔액 조회 (UserId)
        DB-->>Repository: 잔액 데이터
        Repository-->>Service: 잔액 정보
        Service-->>Usecase: 잔액 정보 반환
    else 사용자 미존재
        Service-->>User: 에러 반환 (USER_NOT_FOUND)
    end
    Usecase-->>Controller: 잔액 정보 반환
    Controller-->>User: 잔액 조회 결과 응답
```

> ### 상품 조회 (GET - /product)
- 상품 정보 ( ID, 이름, 가격, 잔여수량 ) 을 조회하는 API 를 작성합니다.
- 조회시점의 상품별 잔여수량이 정확하면 좋습니다.
```mermaid
sequenceDiagram
    actor User
    participant Controller
    participant Usecase
    participant Service
    participant Repository
    participant DB

    User->>Controller: 상품 목록 조회 요청 (page)
    Controller->>Usecase: 상품 목록 조회 요청 전달
    Usecase->>Service: 상품 목록 조회 로직 수행
    Service->>Repository: 상품 목록 조회 (page, offset)
    Repository->>DB: 상품 목록 조회 (page, offset)
    DB-->>Repository: 상품 목록
    Repository-->>Service: 상품 데이터
    Service-->>Usecase: 상품 목록 반환
    Usecase-->>Controller: 상품 목록 반환
    Controller-->>User: 상품 목록 응답
```

> ### 주문/결제 (POST - /product/order/:orderId)
- 사용자 식별자와 (상품 ID, 수량) 목록을 입력받아 주문하고 결제를 수행하는 API 를 작성합니다.
- 결제는 기 충전된 잔액을 기반으로 수행하며 성공할 시 잔액을 차감해야 합니다.
- 데이터 분석을 위해 결제 성공 시에 실시간으로 주문 정보를 데이터 플랫폼에 전송해야 합니다. ( 데이터 플랫폼이 어플리케이션 `외부` 라는 가정만 지켜 작업해 주시면 됩니다 )

> 데이터 플랫폼으로의 전송 기능은 Mock API, Fake Module 등 다양한 방법으로 접근해 봅니다.
```mermaid
sequenceDiagram
    actor User
    participant Controller
    participant Usecase
    participant Service
    participant Repository
    participant External Service
    participant DB

    User->>Controller: 주문 요청 ([상품Id, 수량])
    Controller->>Usecase: 주문 요청 전달
    Usecase->>Service: 주문 및 결제 로직 수행
    Service->>Repository: 사용자 잔액 확인 (UserId)
    Repository->>DB: 사용자 잔액 확인 (UserId)
    DB-->>Repository: 잔액 데이터
    Repository-->>Service: 잔액 정보
    Service->>Repository: 상품 재고 확인 ([상품Id, 수량])
    Repository->>DB: 상품 재고 확인 ([상풍Ids])
    DB-->>Repository: 재고 데이터
    Repository-->>Service: 재고 정보
    alt 잔액 충분하고 재고 충분함
        Service->>Repository: 잔액 차감 (UserID, 총금액)
        Repository->>DB: 잔액 차감 (UserID, 총금액)
        DB-->>Repository: 잔액 차감 완료
        Repository-->>Service: 잔액 차감 결과
        Service->>Repository: 재고 차감 (상품ID, 수량)
        Repository->>DB: 재고 차감 (상품ID, 수량)
        DB-->>Repository: 재고 차감 완료
        Repository-->>Service: 재고 차감 결과
        Service->>Repository: 주문 정보 저장 (UserID, [상품ID, 수량])
        Repository->>DB: 주문 정보 저장 (UserID, [상품ID, 수량])
        DB-->>Repository: 주문 저장 완료
        Repository-->>Service: 주문 저장 결과
        Service->>External Service: 주문 정보 전송 (외부 서비스)
        External Service-->>Service: 전송 완료
        Service-->>Usecase: 주문 성공 응답
    else 잔액 부족 또는 재고 부족
        alt 잔액 부족
            Service-->>Usecase: 에러 반환 (잔액 부족)
        else 재고 부족
            Service-->>Usecase: 에러 반환 (재고 부족)
        end
    end
    Usecase-->>Controller: 주문 결과 반환
    Controller-->>User: 주문 결과 응답
```


> ### 상위 상품 조회 (GET - /product/top)
- 최근 3일간 가장 많이 팔린 상위 5개 상품 정보를 제공하는 API 를 작성합니다.
- 통계 정보를 다루기 위한 기술적 고민을 충분히 해보도록 합니다.
```mermaid
sequenceDiagram
  actor User
  participant Controller
  participant Usecase
  participant Service
  participant Repository
  participant DB

  User->>Controller: 상위 상품 조회 요청
  Controller->>Usecase: 상위 상품 조회 요청 전달
  Usecase->>Service: 상위 상품 조회 로직 수행
  Service->>Repository: 최근 3일간 판매 데이터 조회
  Repository->>DB: 최근 3일간 판매 데이터 조회
  DB-->>Repository: 판매 데이터
  Repository-->>Service: 상위 5개 상품 통계
  Service->>Repository: 상위 상품 상세 정보 조회 (상위 5개 productId)
  Repository->>DB: 상위 상품 상세 정보 조회 (상위 5개 productId)
  DB-->>Repository: 상품 상세 정보
  Repository-->>Service: 인기 상품 정보
  Service-->>Usecase: 인기 상품 목록 반환
  Usecase-->>Controller: 인기 상품 목록 반환
  Controller-->>User: 인기 상품 응답

```


> ### 장바구니 (POST - /cart/add, DELETE - /cart/delete, GET - /cart)
- 사용자는 구매 이전에 관심 있는 상품들을 장바구니에 적재할 수 있습니다.
- 이 기능을 제공하기 위해 `장바구니에 상품 추가/삭제` API 와 `장바구니 조회` API 가 필요합니다.
- 위 두 기능을 제공하기 위해 어떤 요구사항의 비즈니스 로직을 설계해야할 지 고민해 봅니다.

- 장바구니 추가 (POST - /cart/add)
```mermaid
sequenceDiagram
    actor User
    participant Controller
    participant Usecase
    participant Service
    participant Repository
    participant DB

    User->>Controller: 장바구니에 상품 추가 요청 (ProductId, Quantity)
    Controller->>Usecase: 추가 요청 전달
    Usecase->>Service: 장바구니 추가 로직 수행
    Service->>Repository: 사용자 조회 (UserId)
    Repository->>DB: 사용자 조회 (UserId)
    DB-->>Repository: 사용자 데이터
    Repository-->>Service: 사용자 존재 여부
    alt 사용자 존재
        Service->>Repository: 상품 조회 (ProductId)
        Repository->>DB: 상품 조회 (ProductId)
        DB-->>Repository: 상품 데이터
        Repository-->>Service: 상품 데이터
        alt 상품 존재
            alt 상품 재고 존재
                Service->>Repository: 장바구니에 상품 추가 (UserID, ProductID, Quantity)
                Repository->>DB: INSERT INTO Carts (user_id, product_id, quantity, added_at) VALUES (...)
                DB-->>Repository: 추가 완료
                Repository-->>Service: 추가 결과
                Service-->>Usecase: 추가 성공
            else 상품 재고 부족
                Service-->>User: 에러 반환(NOT_ENOUGH_STOCK)
            end
        else 상품 미존재
            Service-->>Usecase: 에러 반환 (PRODUCT_NOT_FOUND)
        end
    else 사용자 미존재
        Service-->>Usecase: 에러 반환 (USER_NOT_FOUND)
    end
    Usecase-->>Controller: 장바구니 추가 결과 반환
    Controller-->>User: 장바구니 추가 응답

```

- 장바구니 삭제 (DELETE - /cart/delete)
```mermaid
sequenceDiagram
    actor User
    participant Controller
    participant Usecase
    participant Service
    participant Repository
    participant DB

    User->>Controller: 장바구니에서 상품 삭제 요청 (ProductID)
    Controller->>Usecase: 삭제 요청 전달
    Usecase->>Service: 장바구니 삭제 로직 수행
    Service->>Repository: 사용자 조회 (UserId)
    Repository->>DB: 사용자 조회 (UserId)
    DB-->>Repository: 사용자 데이터
    Repository-->>Service: 사용자 존재 여부
    alt 사용자 존재
        Service->>Repository: 장바구니 조회(UserId)
        Repository->>Service: 장바구니 데이터
        Service->>Repository: 장바구니에 해당 상품 존재 확인 (CartId, ProductId)
        Repository->>DB: 장바구니에 해당 상품 존재 확인 (CartId, ProductId)
        DB-->>Repository: 장바구니 데이터
        Repository-->>Service: 상품 존재 여부
        alt 상품 존재
            Service->>Repository: 장바구니에서 상품 삭제 (CartId, ProductId)
            Repository->>DB: 장바구니에서 상품 삭제 (CartId, ProductId)
            DB-->>Repository: 삭제 완료
            Repository-->>Service: 삭제 결과
            Service-->>Usecase: 삭제 성공
        else 상품 미존재
            Service-->>Usecase: 에러 반환 (PRODUCT_NOT_FOUND_IN_CART)
        end
    else 사용자 미존재
        Service-->>Usecase: 에러 반환 (USER_NOT_FOUND)
    end
    Usecase-->>Controller: 장바구니 삭제 결과 반환
    Controller-->>User: 장바구니 삭제 응답
```

- 장바구니 조회 (GET - /cart)
```mermaid
sequenceDiagram
    actor User
    participant Controller
    participant Usecase
    participant Service
    participant Repository
    participant DB

    User->>Controller: 장바구니 조회 요청
    Controller->>Usecase: 조회 요청 전달
    Usecase->>Service: 장바구니 조회 로직 수행
    Service->>Repository: 사용자 조회 (UserId)
    Repository->>DB: 사용자 조회 (UserId)
    DB-->>Repository: 사용자 데이터
    Repository-->>Service: 사용자 존재 여부
    alt 사용자 존재
        Service->>Repository: 장바구니 조회 (UserId)
        Repository->>DB: 장바구니 조회 (UserId)
        DB-->>Repository: 장바구니 데이터
        Repository-->>Service: 장바구니 데이터
        Service->>Repository: 장바구니 항목 조회 (CartId)
        Repository->>DB: 장바구니 항목 조회 (CartId)
        DB-->>Repository: 장바구니 항목 데이터
        Repository-->>Service: 장바구니 항목 데이터
        Service->>Repository: 상품 조회 (ProductIds)
        Repository->>DB: 상품 조회 (ProductIds)
        DB-->>Repository: 상품 데이터
        Repository-->>Service: 상품 데이터
        Service-->>Usecase: 장바구니 상품 목록 반환
    else 사용자 미존재
        Service-->>Usecase: 에러 반환 (USER_NOT_FOUND)
    end
    Usecase-->>Controller: 장바구니 조회 결과 반환
    Controller-->>User: 장바구니 조회 응답
```

# MileStone
- https://github.com/users/F1rstID/projects/3

# ERD

# API Docs