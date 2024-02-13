---
layout: post
title: "[Python] FastAPI와 JWT"
tags: [BackEnd PYTHON]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
오늘은 FastAPI와 JWT를 이용하여 안전하고 효과적인 사용자 인증 시스템을 구현하는 방법을 알아보겠습니다.<br/>
JWT 토큰 생성, 적용 및 라우터 그룹에 토큰 검증 적용하는 방법을 상세히 설명하였으니 FastAPI를 활용한 Application 개발 시 확인 후 적용해보세요 ^^
<br/><br/><br/><br/>


## 본문
### JWT 토큰 생성 방법
JWT 토큰은 세 개의 부분으로 구성됩니다.

헤더: 토큰의 형식과 서명 알고리즘을 포함합니다.<br/>
페이로드: 사용자 정보를 포함합니다.<br/>
서명: 토큰의 무결성을 검증하는 데 사용됩니다.<br/>
다음은 Python 라이브러리 PyJWT를 사용하여 JWT 토큰을 생성하는 코드 예시입니다.

```python
import jwt

# 사용자 정보
user_info = {
    "id": 1234,
    "username": "bard",
    "email": "bard@example.com",
}

# 서명 키
secret_key = "my-secret-key"

# 토큰 생성
token = jwt.encode(user_info, secret_key, algorithm="HS256")

print(token)
```
<br/><br/>

### JWT 토큰 적용 방법
JWT 토큰은 HTTP 요청 헤더에 Authorization 키를 사용하여 전송됩니다.<br/>
다음은 FastAPI에서 JWT 토큰을 적용하는 코드 예시입니다.

```python
from fastapi import Depends, FastAPI, HTTPException, Security

# JWT 라이브러리
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

# 토큰 검증 함수
def verify_token(token: str, credentials_exception: HTTPException):
    try:
        # 토큰 디코딩
        payload = jwt.decode(token, "my-secret-key", algorithms=["HS256"])
    except jwt.DecodeError:
        raise credentials_exception

    # 사용자 정보 추출
    user_info = payload.get("user")

    if not user_info:
        raise credentials_exception

    return user_info

# OAuth2
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")

# FastAPI 앱
app = FastAPI()

# 토큰 검증을 통한 사용자 정보 추출
@app.get("/user", dependencies=[Security(oauth2_scheme)])
def get_user(user_info: dict = Depends(verify_token)):
    return user_info
```
<br/><br/>

### JWT 토큰을 적용할 라우터와 적용하지 않을 라우터를 구분하는 방법
Depends 데코레이터를 사용하여 특정 라우터에만 JWT 토큰 검증을 적용할 수 있습니다.<br/>
다음은 토큰 검증을 적용하지 않을 라우터 예시입니다.

```python
@app.post("/token")
def refresh_token(form_data: OAuth2PasswordRequestForm = Depends()):
    # 로그인 처리
    ...

```
<br/>

router 그룹단위로 JWT 토큰 검증을 적용할 수도 있습니다.<br/>
다음은 router 그룹에 JWT 토큰 검증을 적용하는 코드 예시입니다.<br/>
Guest 영역은 토큰 검증을 적용하지 않고, 이외 영역은 토큰 검증을 적용합니다.

```python
import jwt
import datetime
from fastapi import APIRouter, Depends
from fastapi.security import OAuth2PasswordBearer, HTTPAuthorizationCredentials, HTTPBearer

from src.api.game import controller as game_controller
from src.api.game_room_group import controller as game_room_group_controller
from src.api.guest import controller as guest_controller
from src.api.user import controller as user_controller
from src.api.user_group import controller as user_group_controller
from src.api.file import controller as file_controller


def cors_setting(app, CORSMiddleware):
    origins = [
        "*"
    ]
    app.add_middleware(
        CORSMiddleware,
        allow_origins=origins,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )


def decode_jwt_token(token: str):
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
    except jwt.ExpiredSignatureError as e:
        EXCEPTION_TYPE.raise_exception(EXCEPTION_TYPE.ERROR_402)
    except jwt.InvalidTokenError as e:
        EXCEPTION_TYPE.raise_exception(EXCEPTION_TYPE.ERROR_422)


def verify_token(credentials: [HTTPAuthorizationCredentials, None] = Depends(HTTPBearer())):
    # 토큰 검증 로직 수행
    token = credentials.credentials
    decode_jwt_token(token)
    return token


def router_setting(app, master_router):
    guest_router = APIRouter(prefix='/guest')
    user_router = APIRouter(prefix='/user', dependencies=[Depends(verify_token)])
    user_group_router = APIRouter(prefix='/user-group', dependencies=[Depends(verify_token)])
    game_router = APIRouter(prefix='/game', dependencies=[Depends(verify_token)])
    file_router = APIRouter(prefix='/file', dependencies=[Depends(verify_token)])
    game_room_group_router = APIRouter(prefix='/game-room-group', dependencies=[Depends(verify_token)])

    guest_router.include_router(guest_controller.router)
    user_router.include_router(user_controller.router)
    user_group_router.include_router(user_group_controller.router)
    game_router.include_router(game_controller.router)
    game_room_group_router.include_router(game_room_group_controller.router)
    file_router.include_router(file_controller.router)

    master_router.include_router(guest_router)
    master_router.include_router(user_router)
    master_router.include_router(user_group_router)
    master_router.include_router(game_router)
    master_router.include_router(game_room_group_router)
    master_router.include_router(file_router)
    app.include_router(master_router)
```
<br/><br/><br/><br/>

## 글을 마치며
이 글에서는 FastAPI와 JWT를 연동하여 사용자 인증을 구현하는 방법을 설명했습니다.<br/>
JWT 토큰은 사용자 인증, 권한 관리 등 다양한 목적으로 활용될 수 있습니다. 더 많은 정보는 다음 링크를 참조하시기 바랍니다.

FastAPI 공식 문서: https://fastapi.tiangolo.com/ <br/>
PyJWT 공식 문서: https://pyjwt.readthedocs.io/en/stable/