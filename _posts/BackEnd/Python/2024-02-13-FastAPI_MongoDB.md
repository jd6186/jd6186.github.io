---
layout: post
title: "[Python] FastAPI MongoDB 연결 가이드"
tags: [BackEnd PYTHON]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
MongoDB는 문서 중심 NoSQL 데이터베이스로, 문자열 그대로를 사용하기 때문에 빠르게 확장 가능하며 개발자 친화적인 데이터베이스입니다.
FastAPI는 Python 기반 웹 API 프레임워크로, 빠르고 효율적인 API 개발을 지원합니다.<br/>
이 두 기술을 함께 사용해 웹 API를 만들어 보겠습니다.
<br/><br/><br/><br/>

## 본문
### 기본 조건
* Python 3.7 이상
* MongoDB 서버 (설치 및 실행)

### MongoDB 연결 가이드
아래 로직을 따라 MongoDB와 FastAPI를 연동하여 사용자 정보를 저장하고 조회하는 코드를 작성해보겠습니다.<br/>
이 코드는 FastAPI와 MongoDB를 통합하여 간단하고 효율적인 API 서비스를 구축하는 방법을 구현해놨습니다.<br/>

---
**main.py**

```python
from fastapi import FastAPI, Body

import log_util
from dto import CreateKepcoBillingDailyRequestDto
import service

app = FastAPI()


@app.post("/kepco")
async def create_room(create_dto: CreateKepcoBillingDailyRequestDto = Body(...)):
    log_util.info(f"create_room")
    response = service.create_kepco(create_dto)
    log_util.info(f"create_room success: {response}")
    return {"content": True}


@app.get("/kepco")
async def get_rooms():
    log_util.info(f"create_dto")
    response = service.get_kepco_list()
    log_util.info(f"response : {response}")
    return {"content": response}


@app.get("/kepco/{kepco_id}")
async def get_room_detail(kepco_id: str):
    log_util.info("get_room_detail called")
    response = service.get_kepco_by_id(kepco_id)
    log_util.info(f"response : {response}")
    return {"content": response}

```
* POST /kepco: 데이터를 생성 생성합니다.
* GET /kepco: 데이터를 검색해 가져옵니다.
* GET /kepco/{kepco_id}: 특정 데이터를 가져옵니다.
<br/><br/>

---
**dto.py**

```python
from typing import Optional

from pydantic import BaseModel


class CreateKepcoBillingDailyRequestDto(BaseModel):
    F_AP_QT_ALL: Optional[float]
    F_AP_QT_ALL_SUM: Optional[float]
    F_AP_QT_B: Optional[float]
    F_AP_QT_B_SUM: Optional[float]
    F_AP_QT_M: Optional[float]
    F_AP_QT_M_SUM: Optional[float]
    F_AP_QT_S: Optional[float]
    F_AP_QT_S_SUM: Optional[float]
    KWH_BILL_ALL: Optional[float]
    KWH_BILL_ALL_SUM: Optional[float]
    KWH_BILL_B: Optional[float]
    KWH_BILL_B_SUM: Optional[float]
    KWH_BILL_M: Optional[float]
    KWH_BILL_M_SUM: Optional[float]
    KWH_BILL_S: Optional[float]
    KWH_BILL_S_SUM: Optional[float]
    MAX_PWR: Optional[float]

```
이 파일은 데이터 전송 객체(DTO)를 정의합니다. DTO는 API 요청 및 응답 데이터를 유효성 검사합니다.

* CreateRoomDto: 방 생성에 필요한 데이터의 DTO입니다.
  <br/><br/>

---
**service.py**

```python
import dao
import log_util
from dto import CreateKepcoBillingDailyRequestDto


def get_kepco_list():
    rooms = dao.find_kepco_list()
    log_util.info(f"Retrieved rooms : {rooms}")
    return rooms


def get_kepco_by_id(room_id: str):
    return dao.find_kepco_by_id(room_id)


def create_kepco(create_dto: CreateKepcoBillingDailyRequestDto):
    dao.insert_kepco(create_dto)
```
유저 생성과 유저 조회 함수를 정의합니다.
* get_rooms: 모든 방을 가져옵니다.
* get_room_by_id: 특정 ID로 방을 가져옵니다.
* create_room: 새로운 방을 생성합니다.
<br/><br/>

---
**dao.py**

```python
from datetime import timedelta, timezone, datetime
from bson import ObjectId

import log_util
from database import client
from dto import CreateKepcoBillingDailyRequestDto
from models import KepcoBillingDaily

collection = 'kepco_billing_daily'
collection_temp = 'kepco_billing_daily_temp'


def find_kepco_list():
    log_util.info(f"find_kepco_list START --------------------------------------")

    # Mongo 쿼리는 이런 식으로 활용하면 됨
    yesterday = "{:%Y-%m-%d %H:%M:%S}".format(datetime.now(timezone(timedelta(hours=9))) - timedelta(days=1))
    search_pipeline = [
        {"$sort": {"_id": -1}},
        {'$match': {'F_AP_QT_ALL': {"$gt": 10}}},
        {"$limit": 10}
    ]
    cursor = client[collection].aggregate(search_pipeline)
    if not cursor:
        return None
    result_list = [KepcoBillingDaily(result) for result in cursor]
    log_util.info(f"result_list: {result_list}")
    return result_list


def find_kepco_by_id(room_id: str):
    log_util.info(f"find_kepco_by_id START --------------------------------------")
    room_id = ObjectId(room_id)
    response = client[collection].find_one({"_id": room_id})
    if response:
        return KepcoBillingDaily(response)


def insert_kepco(create_dto: CreateKepcoBillingDailyRequestDto):
    log_util.info(f"insert_kepco START --------------------------------------")
    create_dto = create_dto.model_dump()
    # 데이터 삽입
    log_util.info(f"create test; {create_dto}")
    response = client[collection_temp].insert_one(create_dto)

    # 삽입 결과 확인
    if response.acknowledged:
        log_util.info("데이터 삽입 성공!")
    else:
        log_util.error("데이터 삽입 실패:", response.error)


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
class BaseMongoModel:
    def __init__(self, obj: dict):
        for key, value in obj.items():
            if key == "_id":
                self.id = str(value)
            elif isinstance(value, float):
                self.__setattr__(key, float(value))
            elif isinstance(value, bool):
                self.__setattr__(key, bool(value))
            elif isinstance(value, str):
                self.__setattr__(key, str(value))


class KepcoBillingDaily(BaseMongoModel):
    id: str = None
    F_AP_QT_ALL: float = None
    F_AP_QT_ALL_SUM: float = None
    F_AP_QT_B: float = None
    F_AP_QT_B_SUM: float = None
    F_AP_QT_M: float = None
    F_AP_QT_M_SUM: float = None
    F_AP_QT_S: float = None
    F_AP_QT_S_SUM: float = None
    KWH_BILL_ALL: float = None
    KWH_BILL_ALL_SUM: float = None
    KWH_BILL_B: float = None
    KWH_BILL_B_SUM: float = None
    KWH_BILL_M: float = None
    KWH_BILL_M_SUM: float = None
    KWH_BILL_S: float = None
    KWH_BILL_S_SUM: float = None
    MAX_PWR: float = None


```
model 객체를 생성 시 자동으로 변환이 가능하도록 BaseMongoModel을 상속받아 구현합니다.

* RoomCollection: 방의 모델입니다.
<br/><br/>

---
**database.py**

```python
from pymongo.mongo_client import MongoClient
import certifi

import log_util

host = ''
query_param = 'ssl=true&authSource=admin'
port = 27017
username = ''
password = ''
ca = certifi.where()
uri = f"mongodb://{username}:{password}@{host}/?{query_param}"

dbname = ''
client = MongoClient(uri, tlsCAFile=ca)[dbname]
client.command('ping')
log_util.info("Pinged your deployment. You successfully greenos connected to MongoDB!")
```
maxPoolSize 옵션은 연결 풀의 최대 크기를 설정합니다. 기본값은 100입니다.<br/>
이 파일은 MongoDB와의 연결을 설정합니다. 인증 및 데이터베이스 선택을 처리합니다.
<br/><br/>

---
**app_test.py**

```python
import json

import pytest
from starlette.testclient import TestClient

import log_util
from dto import CreateKepcoBillingDailyRequestDto
from main import app


class TestRoom:
    client = TestClient(app)


    def test_create_room_success(self):
        """
        Test creating a room with valid data.
        """
        data = CreateKepcoBillingDailyRequestDto(
            F_AP_QT_ALL=1.0,
            F_AP_QT_ALL_SUM=1.0,
            F_AP_QT_B=1.0,
            F_AP_QT_B_SUM=1.0,
            F_AP_QT_M=1.0,
            F_AP_QT_M_SUM=1.0,
            F_AP_QT_S=1.0,
            F_AP_QT_S_SUM=1.0,
            KWH_BILL_ALL=1.0,
            KWH_BILL_ALL_SUM=1.0,
            KWH_BILL_B=1.0,
            KWH_BILL_B_SUM=1.0,
            KWH_BILL_M=1.0,
            KWH_BILL_M_SUM=1.0,
            KWH_BILL_S=1.0,
            KWH_BILL_S_SUM=1.0,
            MAX_PWR=1.0
        )
        response = self.client.post("/kepco", json=data.model_dump()) # json으로 전달해야 현재는 수신 가능
        log_util.info(f"test_create_room_success : {response.json()}")
        assert response.status_code == 200


    def test_get_all_rooms(self):
        """
        Test retrieving all rooms.
        """
        response = self.client.get("/kepco")
        log_util.info(f"test_get_all_rooms : {response.json()}")
        assert response.status_code == 200


    @pytest.mark.parametrize("id, expected_status_code", [
        ("000000000000000000000000", False),
        ("65cd50dec4b04b0b65598952", True)
    ])
    def test_get_room_by_id_with_parametrization(self, id, expected_status_code):
        response = self.client.get(f"/kepco/{id}")
        content = response.json()
        log_util.info(f"test_get_room_by_id_with_parametrization ({id}) : {content['content']} : {expected_status_code}")

        assert expected_status_code == (content['content'] is not None)


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
pymongo
pytest-html
certifi
pydantic
httpx
```
이 파일에는 프로젝트에 필요한 모든 종속성이 명시되어 있습니다.
<br/><br/>

---
**테스트 실행**
```bash
pytest -o log_cli=true
```
pytest를 사용하여 테스트를 실행합니다. log_cli=true 옵션을 사용하여 테스트 로그를 표시합니다.

---
**log.ini**

```ini
[loggers]
keys=root,sLogger

[handlers]
keys=consoleHandler,fileHandler

[formatters]
keys=fileFormatter,consoleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_sLogger]
level=DEBUG
handlers=consoleHandler,fileHandler
qualname=sLogger
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=consoleFormatter
args=(sys.stdout,)

[handler_fileHandler]
class=FileHandler
level=DEBUG
formatter=fileFormatter
args=('logfile.log',)

[formatter_fileFormatter]
format=[%(asctime)s] %(levelname)s : %(message)s
datefmt=

[formatter_consoleFormatter]
format=[%(asctime)s] %(levelname)s : %(message)s
datefmt=
```
log 관련 설정을 모아 둔 파일입니다.

---
**log_util.py**

```python
import logging.config

# set up logging
logging.config.fileConfig("log.ini")
logger = logging.getLogger('sLogger')


def info(message):
    logger.info(message)


def debug(message):
    logger.debug(message)


def error(message):
    logger.error(message)


def warning(message):
    logger.warning(message)


def critical(message):
    logger.critical(message)
```
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
**MongoDB**<br/>
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