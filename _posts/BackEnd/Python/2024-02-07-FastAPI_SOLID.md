---
layout: post
title: "[Python] FastAPI와 SOLID 원칙"
tags: [BackEnd PYTHON]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
FastAPI는 Python으로 빠르고 현대적인 API를 개발하기 위한 프레임워크입니다.<br/>
이 프레임워크를 사용하면 솔리드하고 효율적인 웹 API를 만들 수 있습니다.<br/>
이를 위해서는 객체지향 프로그래밍의 디자인 원칙인 SOLID를 적절히 적용해야 합니다.<br/>
오늘은 FastAPI와 SOLID 원칙을 함께 사용하여 더욱 효율적인 API를 개발하는 방법에 대해 알아보겠습니다.
<br/><br/><br/><br/>

## 본문
### SRP - 단일 책임 원칙 (Single Responsibility Principle)
각 라우터 함수는 하나의 엔드포인트에 대한 요청을 처리하고 하나의 작업 또는 책임만을 가지고 있어야 합니다.
즉, 각 라우터 함수는 단지 특정 작업을 수행해야 합니다. 예를 들어, 아이템을 조회하는 엔드포인트에 대한 라우터 함수는 단지 그 작업만을 수행해야 합니다.
```python
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    # item_id에 해당하는 아이템 조회 기능만을 수행하는 함수
    pass
```
<br/><br/>

### OCP - 개방-폐쇄 원칙 (Open-Closed Principle)
라우터 함수는 개방-폐쇄 원칙을 준수하여 확장에는 열려 있어야 하고 수정에는 닫혀 있어야 합니다.
이는 새로운 기능을 추가할 때 기존의 코드를 수정하지 않고 새로운 라우터 함수를 추가함으로써 구현됩니다.<br/>
예를 들어, 아래의 Printer 클래스는 개방-폐쇄 원칙을 준수합니다.
```python
class Printer:
    def print_message(self, message):
        # 프린터로 메시지를 출력하는 기능
        pass

class ColorPrinter(Printer):
    def print_message(self, message, color):
        # 색상을 지정하여 메시지를 출력하는 기능
        pass
```
위의 Printer 클래스는 확장에 열려 있습니다.<br/>
여기서 확장에 열려있다는 말은 새로운 기능을 추가할 때 Printer 클래스를 수정할 필요 없이 ColorPrinter와 같은 새로운 서브클래스를 만들어 확장할 수 있다는 뜻입니다.<br/>
이와 같은 방식으로 새로운 기능을 추가할 때 기존의 코드를 수정하지 않고, 원본 객체를 상속받은 새로운 객체를 추가함으로써 개방-폐쇄 원칙을 준수할 수 있습니다.
<br/><br/>

### LSP - 리스코프 치환 원칙 (Liskov Substitution Principle)
상속 관계에 있는 클래스들 간의 관계를 정의하고, 상속을 올바르게 사용할 수 있도록 하는 원칙을 의미합니다.
간단히 말해서, LSP는 상위 클래스가 있을 때 그것을 상속받는 하위 클래스가 상위 클래스를 대체 가능해야 한다는 것입니다.<br/>
이것은 하위 클래스를 사용하는 코드에서 상위 클래스 대신에 하위 클래스를 사용해도 문제가 없어야 함을 의미합니다.
이를 통해 코드의 유연성과 확장성이 향상됩니다.
```python
class BaseItem:
    def calculate_price(self):
        pass

class Item(BaseItem):
    def __init__(self, price: float, tax: float = None):
        self.price = price
        self.tax = tax

  def calculate_price(self):
      if self.tax:
          return self.price * (1 + self.tax)
      else:
          return self.price

class DiscountedItem(BaseItem):
    def __init__(self, price: float, discount: float):
        self.price = price
        self.discount = discount

    def calculate_price(self):
        return self.price * (1 - self.discount)
```
위 코드에서 Item 클래스와 DiscountedItem 클래스는 BaseItem 클래스를 상속받아 calculate_price() 메서드를 오버라이딩하고 있습니다.
이를 통해 리스코프 치환 원칙을 준수하고 있습니다.
<br/><br/>

### ISP - 인터페이스 분리 원칙 (Interface Segregation Principle)
클라이언트는 자신이 사용하지 않는 인터페이스에 의존하지 않아야 합니다. 따라서 여러 개의 특화된 인터페이스가 범용 인터페이스 하나보다 낫습니다.<br/>
파이썬에서는 인터페이스라는 명시적인 개념이 없지만, 추상 클래스나 덕 타이핑을 사용하여 비슷한 효과를 얻을 수 있습니다. 아래는 인터페이스 분리 원칙을 적용한 코드의 예시입니다.
```python
from abc import ABC, abstractmethod

# 소리를 내는 인터페이스
class Soundable(ABC):
    @abstractmethod
    def make_sound(self):
        pass

# 날 수 있는 인터페이스
class Flyable(ABC):
    @abstractmethod
    def fly(self):
        pass

# 헤엄을 칠 수 있는 인터페이스
class Swimmable(ABC):
    @abstractmethod
    def swim(self):
        pass

# 각각의 동물 클래스들은 필요한 인터페이스를 구현합니다.
class Bird(Soundable, Flyable):
    def make_sound(self):
        return "Tweet tweet"

    def fly(self):
        return "Flapping wings and flying"

class Fish(Soundable, Swimmable):
    def make_sound(self):
        return "Blub blub"

    def swim(self):
        return "Moving fins and swimming"

class Dog(Soundable):
    def make_sound(self):
        return "Woof woof"

# 클라이언트 코드
def perform_actions(animal):
    if isinstance(animal, Soundable):
        print(animal.make_sound())
    if isinstance(animal, Flyable):
        print(animal.fly())
    if isinstance(animal, Swimmable):
        print(animal.swim())

# 각 동물이 필요한 인터페이스만을 구현하여 클라이언트 코드에서 필요한 기능을 사용합니다.
bird = Bird()
fish = Fish()
dog = Dog()

perform_actions(bird)
perform_actions(fish)
perform_actions(dog)
```
이 코드에서 각 동물 클래스는 필요한 인터페이스만을 구현하고 있습니다. 
클라이언트 코드에서는 각 동물이 어떤 기능을 제공하는지 확인한 후 필요한 기능을 사용할 수 있습니다. 
이렇게 하면 각 동물 클래스가 자신이 필요로 하는 인터페이스에만 의존하게 되어 ISP를 준수하게 됩니다.
* from abc import ABC, abstractmethod는 Python의 abc 모듈에서 ABC 클래스와 abstractmethod 데코레이터를 가져오는 구문입니다.<br/>
* ABC: 추상 기반 클래스(Abstract Base Class)의 약자로, 이를 상속받은 클래스들이 추상 메서드를 포함하고 있음을 나타냅니다. 추상 클래스는 직접 인스턴스화할 수 없으며, 서브클래스에서 구현해야 하는 메서드를 정의할 수 있습니다.<br/> 
* abstractmethod: 메서드를 추상 메서드로 정의하는 데코레이터입니다. 추상 메서드는 구현이 없는 메서드이며, 서브클래스에서 반드시 재정의해야 합니다. 이를 통해 추상 클래스는 해당 메서드를 반드시 구현해야 함을 강제할 수 있습니다.

이러한 구문을 사용하여 추상 클래스를 정의하고 추상 메서드를 선언함으로써 상속 관계에서의 인터페이스를 정의할 수 있습니다.
위의 코드에서는 이러한 기능을 사용하여 인터페이스를 분리하는데 활용되었습니다.
<br/><br/>

### DIP - 의존성 역전 원칙 (Dependency Inversion Principle)
상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 되며, 둘 모두 추상화에 의존해야 한다는 원칙입니다. 
또한, 추상화는 세부 사항에 의존해서는 안 되며, 세부 사항은 추상화에 의존해야 한다는 원칙도 포함하고 있습니다.
이 원칙을 준수하면서 의존성을 주입함으로써 상위 모듈이 하위 모듈에 의존하지 않도록 할 수 있습니다.<br/>
말만 들으면 어려워보이는 다음 예시를 통해 이해해보도록 합시다.
```python
from abc import ABC, abstractmethod

class Car(ABC):
    @abstractmethod
    def start(self):
        pass

    @abstractmethod
    def drive(self):
        pass

    @abstractmethod
    def stop(self):
        pass

class Benz(Car):
    def start(self):
        print("Benz: Engine started")

    def drive(self):
        print("Benz: Driving")

    def stop(self):
        print("Benz: Stopped")

class Ferrari(Car):
    def start(self):
        print("Ferrari: Engine started")

    def drive(self):
        print("Ferrari: Driving")

    def stop(self):
        print("Ferrari: Stopped")

class BMW(Car):
    def start(self):
        print("BMW: Engine started")

    def drive(self):
        print("BMW: Driving")

    def stop(self):
        print("BMW: Stopped")
```
위 코드에서는 Car 상위 클래스를 추상화로 사용하여 다양한 하위 자동차 브랜드 클래스를 구현하고 있습니다.
이를 통해 자동차 브랜드 클래스들은 모두 Car 클래스에 의존하게 되며, 추상화에만 의존하게 됩니다.
<br/><br/>

### 실제 서비스에 SOLID 원칙 응용 예시
실제 서비스에서도 SOILD 원칙을 적용한 예시들을 볼 수 있습니다.
> 코드는 FastAPI를 사용하고, MySQL 데이터베이스에 SqlAlchemy를 통해 연결하겠습니다.

1. main.py
```python
from fastapi import FastAPI
from routers import item_router

app = FastAPI()
app.include_router(item_router.router)
```

2. dtos/response_dto.py
```python
from pydantic import BaseModel
from typing import Union
from fastapi import status

class BaseResponse(BaseModel):
    success: bool
    message: str = None

class SuccessResponse(BaseResponse):
    data: dict = None

class ErrorResponse(BaseResponse):
    error_code: int
```

3. routers/item_router.py
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from dependencies import get_db, get_item_service
from schemas import ItemCreate
from response_dto import SuccessResponse, ErrorResponse
from services import ItemService

router = APIRouter()

@router.post("/items/", response_model=SuccessResponse)
def create_item(item: ItemCreate, db: Session = Depends(get_db), service: ItemService = Depends(get_item_service)):
    try:
        created_item = service.create_item(db=db, item=item)
        return SuccessResponse(success=True, message="Item created successfully", data=created_item)
    except Exception as e:
        return ErrorResponse(success=False, message="Failed to create item", error_code=status.HTTP_500_INTERNAL_SERVER_ERROR)

@router.get("/items/{item_id}", response_model=Union[SuccessResponse, ErrorResponse])
def read_item(item_id: int, db: Session = Depends(get_db), service: ItemService = Depends(get_item_service)):
    try:
        db_item = service.get_item(db=db, item_id=item_id)
        if db_item:
            return SuccessResponse(success=True, message="Item retrieved successfully", data=db_item)
        else:
            return ErrorResponse(success=False, message="Item not found", error_code=status.HTTP_404_NOT_FOUND)
    except Exception as e:
        return ErrorResponse(success=False, message="Failed to retrieve item", error_code=status.HTTP_500_INTERNAL_SERVER_ERROR)
```

4. service.py
```python
from sqlalchemy.orm import Session
from models import Item
from schemas import ItemCreate

def create_item(db: Session, item: ItemCreate):
    db_item = Item(**item.dict())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

def get_item(db: Session, item_id: int):
    return db.query(Item).filter(Item.id == item_id).first()
```

5. models.py
```python
from sqlalchemy import Column, Integer, String
from database import Base

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    description = Column(String, index=True)
    price = Column(Float)
    tax = Column(Float)
```

6. schemas.py
```python
from pydantic import BaseModel

class ItemBase(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

class ItemCreate(ItemBase):
    pass
```

7. database.py
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "mysql+mysqlconnector://user:password@localhost/db_name"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

#### SOLID 원칙 적용 설명:
* SRP (단일 책임 원칙):
    * 각 파일과 클래스는 하나의 기능 또는 책임만을 가지고 있습니다.
    * main.py는 앱의 진입점으로서의 역할만 수행합니다.
    * routers/item_router.py는 Item과 관련된 라우팅과 요청 처리를 담당합니다.
    * service.py는 비즈니스 로직을 처리합니다.
    * models.py는 데이터 모델을 정의합니다.
    * schemas.py는 Pydantic 모델을 정의합니다.
    * database.py는 데이터베이스 설정과 연결을 담당합니다.
* OCP (개방-폐쇄 원칙):
  * 새로운 기능이 추가될 때 기존의 코드를 수정하지 않고 확장할 수 있도록 설계되었습니다. 예를 들어, 새로운 API 엔드포인트를 추가하려면 새로운 라우터 함수만 추가하면 됩니다.
* LSP (리스코프 치환 원칙):
  * 서브 클래스인 ItemCreate이 부모 클래스인 ItemBase를 대체 가능합니다.
* ISP (인터페이스 분리 원칙):
  * 인터페이스를 명시적으로 정의한 부분은 없지만, FastAPI의 의존성 주입을 통해 인터페이스 분리를 구현하였습니다. 종속성을 주입함으로써 의존성이 하위 모듈에서 상위 모듈로 향하지 않고 상위 모듈이 하위 모듈에 의존하지 않습니다.
* DIP (종속성 역전 원칙):
  * 응답 데이터 전송을 위한 DTO(Data Transfer Object)를 구현
<br/><br/><br/><br/>

## 글을 마치며
오늘은 FastAPI와 SOLID 디자인 원칙을 적용하여 효율적이고 견고한 API를 개발하는 방법에 대해 알아보았습니다. 
SOLID 원칙을 준수하면서 코드를 작성하면 유지 보수가 쉽고 확장성 있는 애플리케이션을 구축할 수 있습니다. 하지만 하시는 업무에 따라 지킬 수 없는 상황일 수도 있으니 상황에 맞게 적용하시길 바랍니다. 

앞으로도 이러한 디자인 원칙을 염두에 두고 개발을 진행하여 더 나은 소프트웨어를 만들어 가시길 바라겠습니다.

감사합니다.
