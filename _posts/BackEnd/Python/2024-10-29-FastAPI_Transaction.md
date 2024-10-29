---
layout: post
title: "Python, FastAPI 에서 트랜잭션 관리하는 방법(결제 프로세스 예시)"
tags: [BackEnd, Python, FastAPI, SQLAlchemy, Transaction, Payment, Order]
---

# Intro
안녕하세요 Noah입니다.

오늘은 FastAPI와 Python에서 트랜잭션 관리하는 방법에 대해 알아보겠습니다.

트랜잭션은 데이터베이스 연산이 여러 작업으로 구성될 때 전체 작업이 성공해야만 변경 사항을 반영하고, 실패 시에는 이전 상태로 복구(rollback)하여 데이터 일관성을 유지하는 중요한 역할을 합니다.

Python에서는 FastAPI와 함께 SQLAlchemy를 사용하여 트랜잭션을 쉽게 관리할 수 있습니다. SQLAlchemy의 `Session` 객체를 통해 트랜잭션을 시작하고 `commit()` 또는 `rollback()`을 사용하여 트랜잭션을 제어할 수 있습니다.

아래에서 트랜잭션을 설정하고 사용하는 예제와 함께 트랜잭션 관리 방법을 자세히 설명하겠습니다.

이제부터 시작해보겠습니다.
<br/><br/><br/><br/>

# 목차

<br/><br/><br/><br/>

## 기본 Transaction 관리 방법
### 1. 트랜잭션 설정
SQLAlchemy의 `Session`을 사용하여 트랜잭션을 관리합니다. `session.commit()`을 통해 변경 사항을 반영하고, 예외 발생 시 `session.rollback()`을 호출하여 트랜잭션을 취소할 수 있습니다.

```python
from sqlalchemy.orm import Session
from models import User, Order

def create_user_and_place_order(db: Session, user_data, order_data):
    new_user = User(**user_data)
    db.add(new_user)
    new_order = Order(**order_data)
    db.add(new_order)

    try:
        db.commit()  # 모든 데이터가 성공적으로 저장되면 트랜잭션을 커밋
    except Exception as e:
        db.rollback()  # 에러 발생 시 롤백
        raise e

```

<br/><br/>

### 2. Isolation Level 설정 예시
트랜잭션 격리 수준을 설정하여 트랜잭션 간 데이터 일관성 문제를 제어할 수 있습니다. 

예를 들어, `SERIALIZABLE`로 설정하면 동시에 같은 데이터를 읽거나 쓸 수 없도록 가장 높은 수준의 일관성을 제공합니다.

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# 트랜잭션 격리 수준을 설정하여 세션 생성
engine = create_engine("postgresql://user:password@localhost/dbname", isolation_level="SERIALIZABLE")
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

```

<br/><br/>

### 3. 트랜잭션 테스트 및 롤백 확인
테스트 코드에서도 트랜잭션을 사용할 수 있으며, 테스트 완료 후 기본적으로 롤백되도록 설정하여 데이터베이스 상태를 원상 복구할 수 있습니다.

```python
from sqlalchemy.orm import Session
from fastapi.testclient import TestClient
from main import app, get_db

client = TestClient(app)

def test_create_user_and_place_order(db: Session):
    user_data = {"name": "John", "email": "john@example.com"}
    order_data = {"amount": 100}

    response = client.post("/create_user_order", json={"user": user_data, "order": order_data})
    assert response.status_code == 200

```

테스트가 종료되면 데이터베이스는 롤백되며 실제로 변경 사항이 반영되지 않습니다.
<br/><br/><br/><br/>

## 결제 프로세스에서 트랜잭션 관리해보기
결제 프로세스를 설계하고 구현할 때는 여러 단계가 연관되며 모든 작업이 성공적으로 완료되어야 데이터 일관성을 유지할 수 있습니다.
<br/><br/>

### 1. 결제와 관련된 테이블 설계
결제 프로세스에서 필요한 주요 테이블은 사용자, 주문, 결제, 결제 상태, 포인트/할인 정보 등을 포함합니다.

#### 테이블 설계
1. **Users** (사용자 정보)
- `user_id`: 사용자 ID (PK)
- `name`: 사용자 이름
- `balance`: 사용자 잔액
1. **Orders** (주문 정보)
- `order_id`: 주문 ID (PK)
- `user_id`: 사용자 ID (FK)
- `total_amount`: 총 주문 금액
1. **Payments** (결제 정보)
- `payment_id`: 결제 ID (PK)
- `order_id`: 주문 ID (FK)
- `amount`: 결제 금액
- `status`: 결제 상태 (예: "PENDING", "COMPLETED")
- `created_at`: 결제 시도 일자
  <br/><br/>

### 2. 트랜잭션을 구분하여 관리하는 이유
결제는 여러 단계로 나뉘며, 각 단계가 성공적으로 처리되지 않으면 이후의 작업을 중단하고 이전 상태로 되돌려야 합니다.

#### 트랜잭션 구분 방식
1. **주문 생성 트랜잭션**: 주문을 생성하여 결제 전 준비 상태로 저장.
2. **결제 초기화 트랜잭션**: 결제 상태를 "PENDING"으로 설정.
3. **결제 승인 및 트랜잭션 처리 트랜잭션**: 결제가 성공되면 상태를 "COMPLETED"로 변경.
   <br/><br/>

### 3. 트랜잭션 관리 코드 예시
아래는 결제 트랜잭션 관리 예제 코드입니다.

```python
from sqlalchemy.orm import Session
from models import Order, Payment

def create_order(db: Session, order_data):
    new_order = Order(**order_data)
    db.add(new_order)
    db.commit()
    return new_order

def initiate_payment(db: Session, order_id, amount):
    payment = Payment(order_id=order_id, amount=amount, status="PENDING")
    db.add(payment)
    db.commit()
    return payment

def complete_payment(db: Session, payment_id):
    payment = db.query(Payment).filter(Payment.payment_id == payment_id).first()
    payment.status = "COMPLETED"
    db.commit()

```

<br/><br/>

### 4. 트랜잭션 흐름 요약
1. **주문 생성**: 주문 정보를 데이터베이스에 저장.
2. **결제 초기화**: 결제 금액을 계산하여 `PENDING` 상태로 저장.
3. **결제 승인**: 결제가 성공하면 상태를 "COMPLETED"로 설정.

이러한 단계적 관리로 각 트랜잭션이 독립적으로 처리되며, 특정 단계에서 문제가 발생하면 이전 데이터가 보호됩니다.

하지만 모든 상황에서 저렇게 간단하게 처리되진 않겠죠? 아래 예시를 통해 복잡한 트랜잭션을 살펴보겠습니다.
<br/><br/><br/><br/>


## 복잡한 Transaction 관리 방법
복잡한 트랜잭션 처리는 주로 **하나의 API가 여러 트랜잭션을 필요로 할 때** 발생합니다.

예를 들어, 결제 API에서는 사용자의 포인트, 할인 쿠폰, 결제 상태 업데이트, 결제 시스템 호출 등 여러 단계가 필요하며, 각 단계가 개별 트랜잭션으로 처리될 수 있습니다.

이런 복잡한 프로세스에서는 전체 트랜잭션 관리가 매우 중요하며, 한 단계라도 실패하면 모든 작업을 롤백해야 합니다.

여기서는 예시로 **포인트 결제와 외부 결제 시스템 호출을 포함한 복잡한 결제 API**를 설계해 보겠습니다.

각 단계별로 트랜잭션이 진행되며, 일부 작업은 독립 트랜잭션으로 처리되어야 하고, 일부는 전체 트랜잭션으로 관리됩니다.
<br/><br/>

### 1. 예제 시나리오: 결제 API의 복잡한 트랜잭션 처리
**결제 API 단계:**

1. 사용자 포인트 차감
2. 할인 쿠폰 적용 여부 확인 및 차감
3. 결제 요청 상태를 "PENDING"으로 설정
4. 외부 결제 시스템에 요청 (실패 가능성이 존재)
5. 결제 성공 시 결제 완료 상태 "COMPLETED"로 설정하고 트랜잭션 로그 기록

각 단계는 트랜잭션 관리가 필요하며, 특히 외부 결제 시스템에서 실패 시에는 모든 작업을 롤백해야 합니다.
<br/><br/>

### 2. 코드 예시: 결제 API의 트랜잭션 처리
우선 각 단계를 독립적으로 처리하기 위해 Unit of Work 패턴과 SQLAlchemy의 Session을 사용해 트랜잭션을 관리할게요~

#### 프로젝트 구조
```plaintext
payment_service/
├── app/
│   ├── controllers/
│   │   └── payment_controller.py
│   ├── services/
│   │   └── payment_service.py
│   ├── repositories/
│   │   ├── user_repository.py
│   │   ├── coupon_repository.py
│   │   ├── payment_repository.py
│   │   └── transaction_log_repository.py
│   ├── models/
│   │   ├── user.py
│   │   ├── coupon.py
│   │   ├── payment.py
│   │   └── transaction_log.py
│   ├── db/
│   │   └── database.py
│   ├── core/
│   │   └── config.py
│   ├── main.py
│   └── dependencies.py
└── requirements.txt
```

* controllers: FastAPI endpoint 정의 (API 요청의 진입점).
* services: 비즈니스 로직을 처리.
* repositories: 데이터베이스와의 상호작용을 담당.
* models: SQLAlchemy 모델 정의.
* db: 데이터베이스 연결 설정.
* core: 환경 설정 및 설정 파일.
* main.py: FastAPI 애플리케이션 진입점.
* dependencies.py: 의존성 주입을 관리하여 DB 세션과 서비스 계층 등을 제공.


#### 파일별 역할 및 코드 예시
##### `app/repositories/user_repository.py` (및 다른 Repository 파일)
**역할**: 데이터베이스와의 상호작용을 담당하며, 각 모델에 대한 CRUD 메소드를 제공합니다.

```python
from sqlalchemy.orm import Session
from app.models.user import User

class UserRepository:
    def __init__(self, db: Session):
        self.db = db

    def get_user(self, user_id: int):
        return self.db.query(User).filter(User.id == user_id).first()

# 비슷한 구조로 coupon_repository.py, payment_repository.py, transaction_log_repository.py도 작성

```

##### `app/models/user.py` (및 다른 Model 파일)
**역할**: 데이터베이스 테이블을 나타내는 SQLAlchemy 모델을 정의합니다.

```python
from sqlalchemy import Column, Integer, Float
from app.db.database import Base

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    points = Column(Float, default=0)

# 비슷한 구조로 coupon.py, payment.py, transaction_log.py도 작성

```

##### `app/db/database.py`
**역할**: 데이터베이스 연결을 설정합니다.

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

```

##### `app/core/config.py`
**역할**: 환경 설정 및 데이터베이스 URL, 외부 API 키 등을 관리합니다.

```python
import os

class Settings:
    DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./test.db")
    # 외부 결제 시스템 관련 설정을 포함할 수 있음

settings = Settings()

```

##### `app/dependencies.py`
**역할**: DB 세션 등 공통 의존성을 관리합니다.

```python
from app.db.database import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

##### `app/main.py`
**역할**: FastAPI 애플리케이션을 초기화하고 라우터를 등록합니다.

```python
from fastapi import FastAPI
from app.controllers import payment_controller

app = FastAPI()

app.include_router(payment_controller.router, prefix="/api")

```

##### `app/controllers/payment_controller.py`
**역할**: 결제 API 엔드포인트를 정의하며, 요청을 서비스 계층으로 전달합니다.

```python
python
코드 복사
from fastapi import APIRouter, HTTPException, Depends
from sqlalchemy.orm import Session
from app.services.payment_service import PaymentService
from app.dependencies import get_db

router = APIRouter()

@router.post("/payments/")
def process_payment(user_id: int, amount: float, db: Session = Depends(get_db)):
    payment_service = PaymentService(db)
    try:
        payment = payment_service.process_payment(user_id=user_id, amount=amount)
        return {"message": "Payment processed successfully", "payment_id": payment.id}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

```

##### `app/services/payment_service.py`
**역할**: 결제 로직을 관리하며, 트랜잭션 처리 및 외부 결제 시스템 호출을 포함한 비즈니스 로직을 수행합니다.

```python
from sqlalchemy.orm import Session
from app.repositories.user_repository import UserRepository
from app.repositories.coupon_repository import CouponRepository
from app.repositories.payment_repository import PaymentRepository
from app.repositories.transaction_log_repository import TransactionLogRepository
from app.models.payment import PaymentStatus
import requests

class PaymentService:
    def __init__(self, db: Session):
        self.db = db
        self.user_repo = UserRepository(db)
        self.coupon_repo = CouponRepository(db)
        self.payment_repo = PaymentRepository(db)
        self.log_repo = TransactionLogRepository(db)

    def process_payment(self, user_id: int, amount: float):
        try:
            # Step 1: 사용자 포인트 차감
            user = self.user_repo.get_user(user_id)
            if user.points < amount:
                raise ValueError("Insufficient points.")
            user.points -= amount

            # Step 2: 할인 쿠폰 적용 및 차감
            coupon = self.coupon_repo.get_coupon(user_id)
            if coupon:
                amount -= coupon.discount_amount
                self.coupon_repo.delete_coupon(coupon)

            # Step 3: 결제 정보 초기화 및 상태 설정
            payment = self.payment_repo.create_payment(user_id, amount, PaymentStatus.PENDING)
            self.db.commit()  # 초기 상태 커밋
            self.db.refresh(payment)

            # Step 4: 외부 결제 시스템 호출
            external_payment_result = self.external_payment_system(payment.id, amount)

            if external_payment_result.get("status") != "success":
                raise ValueError("External payment failed.")

            # Step 5: 트랜잭션 로그 생성
            payment.status = PaymentStatus.COMPLETED
            self.log_repo.create_log(payment_id=payment.id, status="COMPLETED")

            self.db.commit()  # 모든 단계가 성공한 경우에만 최종 커밋
            return payment

        except Exception as e:
            self.db.rollback()  # 오류 발생 시 롤백
            raise ValueError(f"Payment process failed: {str(e)}")

    def external_payment_system(self, payment_id: int, amount: float):
        """
        실제 외부 결제 시스템과 상호작용을 수행하는 메서드.
        실제 환경에서는 이 메서드가 외부 API를 호출하고 결제 결과를 반환합니다.
        """
        try:
            # 외부 결제 API 호출 예시
            response = requests.post(
                "https://api.external-payment.com/charge",
                json={"payment_id": payment_id, "amount": amount},
                headers={"Authorization": "Bearer YOUR_API_KEY"},
                timeout=10
            )
            response.raise_for_status()  # 응답이 성공적이지 않으면 예외 발생

            # 응답 파싱
            payment_response = response.json()
            return payment_response

        except requests.exceptions.RequestException as e:
            # 네트워크 오류, 타임아웃 등의 예외 처리
            raise ValueError(f"External payment request failed: {str(e)}")

```
<br/><br/>

#### 트랜잭션 처리 단계
1. **사용자 포인트 차감**
    - 포인트 잔액을 차감하고, 사용자 정보를 업데이트합니다.
    - 이 작업이 실패하면 결제 전체가 롤백됩니다.
2. **할인 쿠폰 적용 및 차감**
    - 할인 쿠폰의 만료 여부를 확인하고, 만료되지 않았으면 쿠폰을 삭제하여 차감 처리합니다.
    - 이 작업이 실패해도 전체 트랜잭션이 롤백됩니다.
3. **결제 정보 초기화 및 상태 설정**
    - `Payment` 엔터티를 생성하고 상태를 "PENDING"으로 설정하여 결제를 초기화합니다.
    - 결제 상태가 아직 확정되지 않았으므로 외부 결제가 성공해야만 이후 상태를 "COMPLETED"로 업데이트합니다.
4. **외부 결제 시스템 호출**
    - 외부 결제 시스템 호출이 실패할 경우 HTTPException을 발생시키고 트랜잭션 전체를 롤백합니다.
5. **트랜잭션 로그 생성**
    - 결제가 성공적으로 완료되면 트랜잭션 로그에 기록하여 결제 내역을 추적합니다.

#### 중요 부분 요약 설명
- **독립 트랜잭션**: 일부 작업은 즉각적으로 반영되며, 예를 들어 포인트 차감, 쿠폰 차감 등은 한 번 수행하면 결과가 유지됩니다.
- **전체 트랜잭션 롤백**: 결제 중 하나의 단계라도 실패하면 모든 작업이 롤백되어 데이터 무결성이 보장됩니다.
- **예외 처리**: try-except 블록을 통해 예외가 발생할 경우 rollback()을 호출하여 상태를 원래대로 되돌립니다.
  <br/><br/><br/><br/>

# Outro
이번 글에서는 Python, FastAPI에서 트랜잭션을 관리하는 방법을 실무에서 사용할 법한 결제 프로세스를 예시로 하여 자세히 알아보았습니다.

트랜잭션의 기본 개념부터 FastAPI와 SQLAlchemy에서 트랜잭션을 관리하는 설정, 그리고 실무에서 자주 사용되는 트랜잭션 관리 예시까지 다룰려고 노력했는데 도움이 되었으면 좋겠습니다 ^^

트랜잭션이 데이터 일관성을 유지하는데 얼마나 중요한 역할을 하는지 인지하셨으면 좋겠구요.

FastAPI와 SQLAlchemy 활용을 통해 안정적이고 견고한시스템을 구축하고자 하는 분들에게 도움이 되었기를 바랍니다.

긴 글 읽어주셔서 감사합니다! 궁금한 점이나 더 알고 싶은 내용이 있다면 언제든지 댓글로 남겨주세요.
<br/><br/><br/><br/>
