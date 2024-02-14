---
layout: post
title: "[Python] FastAPI WebSocket 사용 가이드"
tags: [BackEnd PYTHON]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
MongoDB는 문서 중심 NoSQL 데이터베이스로, 빠르고 확장 가능하며 개발자 친화적입니다.
FastAPI는 Python 기반 웹 API 프레임워크로, 빠르고 효율적인 API 개발을 지원합니다.<br/>
이 두 기술을 함께 사용해 웹 API를 만들어 보겠습니다.
<br/><br/><br/><br/>

## 본문
### 기본 조건
* Python 3.7 이상
* MongoDB 서버 (설치 및 실행)
* API 설치
  ```bash
  pip install fastapi motor
  ```

### MongoDB 연결 가이드
아래 로직을 따라 MongoDB와 FastAPI를 연동하여 사용자 정보를 저장하고 조회하는 코드를 작성해보겠습니다.<br/>
이 코드는 FastAPI와 MongoDB를 통합하여 간단하고 효율적인 API 서비스를 구축하는 방법을 구현해놨습니다.<br/>

---
**main.py**

```python
from fastapi import FastAPI, Depends
import logging
from pymongo import MongoClient
import service
from database import get_db
from dto import CreateRoomDto
from models import RoomCollection

app = FastAPI()
logging.basicConfig(level=logging.INFO)


@app.post("/rooms")
async def create_room(create_dto: CreateRoomDto, client: MongoClient = Depends(get_db)):
    logging.info("create_room called")
    response = service.create_room(create_dto, client)
    logging.info(f"create_room success: {response}")
    return {"success": True}


@app.get("/rooms", response_model=list[RoomCollection])
async def get_rooms(client: MongoClient = Depends(get_db)):
    logging.info("get_rooms called")
    response = service.get_rooms(client)
    logging.info(f"response : {response}")
    return {"rooms": response}


@app.get("/rooms/{room_id}", response_model=RoomCollection)
async def get_room_detail(room_id: str, client: MongoClient = Depends(get_db)):
    logging.info("get_room_detail called")
    response = service.get_room_by_id(room_id, client)
    logging.info(f"response : {response}")
    return {"rooms": response}
```
* /rooms: 새로운 방을 생성합니다.
* /rooms: 모든 방을 가져옵니다.
* /rooms/{room_id}: 특정 방을 가져옵니다.
<br/><br/>

---
**service.py**

```python
import logging

from pymongo import MongoClient

from dto import CreateRoomDto

import dao


def get_rooms(client: MongoClient):
  rooms = dao.find_rooms(client)
  logging.info(f"Retrieved rooms : {rooms}")
  return rooms


def get_room_by_id(room_id: str, client: MongoClient):
  return dao.find_room_by_id(room_id, client)


def create_room(create_dto: CreateRoomDto, client: MongoClient):
  dao.insert_room(create_dto, client)
```
유저 생성과 유저 조회 함수를 정의합니다.
* get_rooms: 모든 방을 가져옵니다.
* get_room_by_id: 특정 ID로 방을 가져옵니다.
* create_room: 새로운 방을 생성합니다.
<br/><br/>

---
**dao.py**

```python
import logging
from datetime import timedelta, timezone, datetime
from bson import ObjectId
from pymongo import MongoClient
from dto import CreateRoomDto
from models import RoomCollection

collection = 'room'


def find_rooms(client: MongoClient):
    date = "{:%Y-%m-%d %H:%M:%S}".format(datetime.now(timezone(timedelta(hours=9))) - timedelta(days=2))

    # Mongo 쿼리는 이런 식으로 활용하면 됨
    pipeline = [
        {"$sort": {"createdAt": -1}}, # 최신순으로 정렬
        {'$match': {'createdAt': {'$lt': date}}}, # 2일 이전 데이터만 조회
        {"$group": {"_id": "room_id", "room_data": {"$first": "$$ROOT"}}}, # 중복 제거
        {"$project": {"_id": 0, "room_id": "$_id", "room_data": 1}} # 필요한 필드만 조회
    ]
    cursor = client[collection].aggregate(pipeline)
    logging.info(f"cursor: {cursor}")
    result_list = [RoomCollection(**result["room_data"]).dict() for result in cursor]
    logging.info(f"result_list: {result_list}")
    if result_list:
        return [RoomCollection(**result).dict() for result in result_list]


def find_room_by_id(room_id: str, client: MongoClient):
    logging.info(f"room_id: {room_id}")
    room_id = ObjectId(room_id)
    result = client[collection].find_one({"_id": room_id})
    logging.info(f"mongo result : {result}")
    if result:
        return RoomCollection(**result).dict()


def insert_room(create_dto: CreateRoomDto, client: MongoClient):
    create_dto = create_dto.model_dump()
    logging.info(f"create_dto: {create_dto}")
    print(f"create_dto: {create_dto}")
    # 데이터 삽입
    result = client[collection].insert_one(create_dto)

    # 삽입 결과 확인
    if result.acknowledged:
        print("데이터 삽입 성공!")
    else:
        print("데이터 삽입 실패:", result.error)
```
위 코드는 FastAPI와 MongoDB를 연동하여 사용자 정보를 저장하고 조회하는 코드입니다.<br/>
유저 생성과 유저 조회 함수를 정의합니다. 이때, models에 정의된 User에서 MongoDB의 '_id'를 Python Code에서 'id'로 변환하여 정상적으로 조회됩니다.

* find_rooms: 모든 방을 가져오는 쿼리를 실행합니다.
* find_room_by_id: 특정 ID로 방을 조회합니다.
* insert_room: 새로운 방을 데이터베이스에 삽입합니다.
<br/><br/>

---
**models.py**

```python
from typing import Annotated, Optional
from pydantic import BaseModel, BeforeValidator, Field

StringConvertType = Annotated[str, BeforeValidator(str)]
IntegerConvertType = Annotated[int, BeforeValidator(int)]


class RoomCollection(BaseModel):
  id: Optional[StringConvertType] = Field(alias="_id", default=None)
  category: str = None
  description: str = None
  name: str = None
  order: int = None
  room_type: StringConvertType = Field(alias="roomType", default=None)
  square_meter: IntegerConvertType = Field(alias="squareMeter", default=None)
```
python 문법 중 '_'를 사용하여 변수명을 지정하면, 해당 변수는 private 변수로 취급되어 외부에서 접근할 수 없습니다.<br/>
위와 같이 모델을 관리하면 MongoDB의 '_id'가 Python Code에서 'id'로 변환되어 정상적으로 조회됩니다.

* RoomCollection: 방의 모델입니다.
<br/><br/>

---
**dto.py**

```python
from pydantic import BaseModel, Field


class CreateRoomDto(BaseModel):
  category: str = Field(max_length=100)
  description: str = Field(max_length=100)
  name: str = Field(max_length=100)
  order: int
  room_type: str = Field(max_length=100)
  square_meter: int
```
이 파일은 데이터 전송 객체(DTO)를 정의합니다. DTO는 API 요청 및 응답 데이터를 유효성 검사합니다.

* CreateRoomDto: 방 생성에 필요한 데이터의 DTO입니다.
<br/><br/>

---
**database.py**

```python
import logging
from pymongo.mongo_client import MongoClient
import certifi

host = 'atlas-online-archive-6102f5b81406e867b86462a6-po4vl.a.query.mongodb.net'
query_param = 'ssl=true&authSource=admin&retryWrites=true&w=majority'
port = 27017
username = 'greendata'
password = 'Ab1234567*'
ca = certifi.where()
uri = f"mongodb://{username}:{password}@{host}/?{query_param}"
dbname = 'greenos'
client = MongoClient(uri, tlsCAFile=ca)


def get_db():
  try:
    db = client[dbname]
    db.command('ping')
    logging.info("Pinged your deployment. You successfully greenos connected to MongoDB!")
    return db
  except:
    logging.error("Failed to connect to the database")
```
maxPoolSize 옵션은 연결 풀의 최대 크기를 설정합니다. 기본값은 100입니다.<br/>
이 파일은 MongoDB와의 연결을 설정합니다. 인증 및 데이터베이스 선택을 처리합니다.
<br/><br/>

---
**room_test.py**

```python
import pytest
import logging
from starlette.testclient import TestClient
from main import app


class TestRoom:
  client = TestClient(app)

  def test_create_room_success(self):
    """
    생성 TEST
    """
    data = {
      "category": "test_category",
      "description": "test_description",
      "name": "test_name",
      "order": 1,
      "room_type": "test_room_type",
      "square_meter": 100
    }
    response = self.client.post("/rooms", data=data)
    logging.info(f"test_create_room_success : {response.json()}")
    assert response.status_code == 200

  def test_get_all_rooms(self):
    """
    검색 테스트
    """
    response = self.client.get("/rooms")
    logging.debug(f"test_get_all_rooms : {response.json()}")
    assert response.status_code == 200

  @pytest.mark.parametrize("id, expected_status_code", [
    ("invalid_id", 404),  # Testing invalid ID
    ("실제 있는 ObjectId", 200)  # Testing valid ID
  ])
  def test_get_room_by_id_with_parametrization(self, id, expected_status_code):
    """
    상세조회 테스트 (파라미터화)
    """
    response = self.client.get(f"/rooms/{id}")
    logging.info(f"test_get_room_by_id_with_parametrization ({id}) : {response.status_code}")
    assert response.status_code == expected_status_code
```
이 파일은 방 관련 API에 대한 테스트를 정의한 내용입니다.

* test_create_room_success: 새로운 방을 성공적으로 생성하는지 테스트합니다.
* test_get_all_rooms: 모든 방을 성공적으로 검색하는지 테스트합니다.
* test_get_room_by_id_with_parametrization: 특정 ID로 방을 검색하는지 테스트합니다. 파라미터화된 테스트로 유효한 ID와 무효한 ID를 사용하여 테스트합니다.
<br/><br/>

---
**requirement.txt**

```txt
fastapi
hypercorn
httpx
pytest
pymongo
certifi
```
이 파일에는 프로젝트에 필요한 모든 종속성이 명시되어 있습니다.
<br/><br/>

---
**테스트 실행**
```bash
pytest -o log_cli=true
```
pytest를 사용하여 테스트를 실행합니다. log_cli=true 옵션을 사용하여 테스트 로그를 표시합니다.
<br/><br/><br/><br/>

### RDBMS와의 차이점
1. 스키마<br/>
MongoDB는 **RDBMS와 다르게 스키마가 없습니다.** 따라서 데이터베이스에 데이터를 저장할 때 스키마를 정의하지 않아도 됩니다.
또한, MongoDB는 **JSON 형태의 데이터를 저장**하므로, 데이터베이스에 저장된 데이터를 JSON 형태로 조회할 수 있습니다.
이러한 특징을 활용하여 데이터베이스에 저장된 데이터를 쉽게 관리할 수 있습니다.
<br/><br/>

2. 데이터베이스 커넥션 관리<br/>
**RDBMS (SQLAlchemy 사용 시)**<br/>
Session Maker 생성: sessionmaker 객체를 만들고 변수에 저장합니다.<br/>
Dependency 함수: get_session과 같은 dependency 함수를 만들어 요청마다 sessionmaker를 사용하여 새 세션을 생성합니다.<br/>
새로운 연결: 각 요청은 session을 배정받아 관리됩니다.<br/><br/>
**MongoDB (Motor 사용 시)**<br/>
연결 풀: Motor는 기본적으로 비동기 연결 풀을 사용합니다.<br/>
새로운 연결 없음: 요청마다 새 연결을 생성하지 않고 애플리케이션 전체에서 AsyncIOMotorClient 인스턴스를 직접 사용합니다.<br/>
효율성: 이 방식은 효율적이며 불필요한 연결 오버헤드가 생기는 것을 방지해줍니다.
<br/><br/><br/><br/>

## 글을 마치며
이번 포스트에서는 FastAPI와 MongoDB를 결합하여 API 서비스를 구축하는 방법을 살펴보았습니다.<br/>
FastAPI를 사용하여 방 생성 및 조회 API를 작성하고 MongoDB와 연동하여 데이터 관리를 수행했으며, Pydantic을 활용하여 데이터 모델링과 유효성 검사를 간편하게 처리했습니다.<br/>
pytest를 활용하여 API 기능을 테스트하고 안정성을 확보했습니다. 이를 통해 FastAPI와 MongoDB를 연결하는 방법에 대해 확인할 수 있었습니다.<br/>
여러분도 FastAPI와 MongoDB를 활용하여 다양한 프로젝트를 구현하고 개발 경험을 쌓아나가기를 기대하겠습니다.<br/>
다들 즐코하세요~ 🚀
<br/><br/>

* 참고자료
  * [FastAPI 공식 문서](https://fastapi.tiangolo.com/ko/)
  * [MongoDB 공식 FastAPI 연동 문서](https://www.mongodb.com/developer/languages/python/python-quickstart-fastapi/)