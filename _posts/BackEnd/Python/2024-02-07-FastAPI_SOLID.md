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
### 단일 책임 원칙 (SRP)
각 라우터 함수는 하나의 엔드포인트에 대한 요청을 처리하고 하나의 작업 또는 책임만을 가지고 있어야 합니다.
즉, 각 라우터 함수는 단지 특정 작업을 수행해야 합니다. 예를 들어, 아이템을 조회하는 엔드포인트에 대한 라우터 함수는 단지 그 작업만을 수행해야 합니다.
```python
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    # item_id에 해당하는 아이템 조회 기능만을 수행하는 함수
    pass
```
<br/><br/>

### 개방-폐쇄 원칙 (OCP)
라우터 함수는 개방-폐쇄 원칙을 준수하여 확장에는 열려 있어야 하고 수정에는 닫혀 있어야 합니다.
이는 새로운 기능을 추가할 때 기존의 코드를 수정하지 않고 새로운 라우터 함수를 추가함으로써 구현됩니다.
```python
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    # item_id에 해당하는 아이템을 업데이트하는 기능
    pass
```
이와 같은 방식으로 새로운 기능을 추가할 때 기존의 코드를 수정하지 않고 새로운 라우터 함수를 추가함으로써 개방-폐쇄 원칙을 준수할 수 있습니다.
<br/><br/>

### 리스코프 치환 원칙 (LSP)
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

### 인터페이스 분리 원칙 (ISP)
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

> * from abc import ABC, abstractmethod는 Python의 abc 모듈에서 ABC 클래스와 abstractmethod 데코레이터를 가져오는 구문입니다.<br/>
> * ABC: 추상 기반 클래스(Abstract Base Class)의 약자로, 이를 상속받은 클래스들이 추상 메서드를 포함하고 있음을 나타냅니다. 추상 클래스는 직접 인스턴스화할 수 없으며, 서브클래스에서 구현해야 하는 메서드를 정의할 수 있습니다.<br/> 
> * abstractmethod: 메서드를 추상 메서드로 정의하는 데코레이터입니다. 추상 메서드는 구현이 없는 메서드이며, 서브클래스에서 반드시 재정의해야 합니다. 이를 통해 추상 클래스는 해당 메서드를 반드시 구현해야 함을 강제할 수 있습니다.

이러한 구문을 사용하여 추상 클래스를 정의하고 추상 메서드를 선언함으로써 상속 관계에서의 인터페이스를 정의할 수 있습니다.
위의 코드에서는 이러한 기능을 사용하여 인터페이스를 분리하는데 활용되었습니다.


### 응용방법
각 클래스를 별도의 파일로 분리하여 구현할 수 있습니다. 아래는 각 클래스를 별도의 파일로 분리한 예시입니다.

user.py (DTO)
```python
from pydantic import BaseModel

class UserDTO(BaseModel):
    id: int
    username: str
    email: str

    class Config:
        orm_mode = True

```

user_dao.py (DAO)
```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.orm import selectinload
from models import User

class UserDAO:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def get_user_by_id(self, user_id: int) -> User:
        result = await self.session.execute(
            selectinload(User).where(User.id == user_id)
        )
        user = result.scalars().first()
        return user

```

models.py (Domain)
```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.ext.asyncio import AsyncSession, AsyncEngine
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, index=True)
    email = Column(String, index=True)

```

user_service.py (Service)
```python
from sqlalchemy.ext.asyncio import AsyncSession
from dao import UserDAO
from dto import UserDTO

class UserService:
    def __init__(self, user_dao: UserDAO):
        self.user_dao = user_dao

    async def get_user_by_id(self, user_id: int) -> UserDTO:
        user = await self.user_dao.get_user_by_id(user_id)
        if not user:
            raise HTTPException(status_code=404, detail="User not found")
        return UserDTO(**user.dict())
```

main.py (FastAPI Router)
```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from database import get_session
from service import UserService
from dto import UserDTO

app = FastAPI()

@app.on_event("startup")
async def startup_db_client():
    async with async_engine.begin() as conn:
        await conn.run_sync(models.Base.metadata.create_all)

async def get_user_service(session: AsyncSession = Depends(get_session)) -> UserService:
    user_dao = UserDAO(session)
    return UserService(user_dao)

@app.get("/users/{user_id}", response_model=UserDTO)
async def read_user(user_id: int, user_service: UserService = Depends(get_user_service)):
    return await user_service.get_user_by_id(user_id)
```

database.py
```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.ext.asyncio import AsyncSession, AsyncEngine
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite+aiosqlite:///./test.db"
async_engine = create_async_engine(SQLALCHEMY_DATABASE_URL, echo=True, future=True)

async_session = sessionmaker(
    async_engine, class_=AsyncSession, expire_on_commit=False
)

def get_session() -> AsyncSession:
    return async_session()
```
각 파일은 단일 책임 원칙을 준수하고, 모듈 간의 의존성을 적절히 관리하여 SOLID 원칙을 따르고 있습니다. 이렇게 파일을 분리함으로써 코드의 가독성과 유지보수성이 향상될 수 있습니다.

<br/><br/><br/><br/>


## 글을 마치며