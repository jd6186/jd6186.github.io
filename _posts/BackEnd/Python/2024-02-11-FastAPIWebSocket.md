---
layout: post
title: "[Python] FastAPI WebSocket 사용 가이드"
tags: [BackEnd PYTHON]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
FastAPI는 일반 API 제작하는 것 뿐만 아니라, WebSocket을 사용하는 방법도 간단하게 제공합니다.
이 가이드에서는 FastAPI에서 WebSocket을 구현하는 방법과 회원들이 연결된 소켓을 관리하는 방법을 알아보겠습니다.
<br/><br/><br/><br/>

## 본문
### FastAPI 프로젝트 시작
먼저 FastAPI 프로젝트를 시작합니다. 만약 FastAPI를 처음 사용하시는 경우, 다음과 같이 간단한 Hello World 앱을 만들어보세요:

```Python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

이 코드를 main.py 파일에 복사하고 다음 명령 중 본인이 사용하시는 ASGI 서버를 실행합니다
> uvicorn main:app --reload <br/>
> hypercorn main:app --reload

로컬에서 앱이 서비스되는 URL은 여기에서 확인할 수 있습니다. 브라우저로 해당 URL을 열어보면 JSON 응답인 {"message": "Hello World"}를 볼 수 있습니다.
<br/><br/>

### WebSocket 경로 추가
FastAPI에서 WebSocket을 사용하려면 다음과 같이 WebSocket 경로를 추가해야 합니다:
```Python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
   await websocket.accept()
   while True:
      data = await websocket.receive_text()
      await websocket.send_text(f"Message text was: {data}")
```
위 코드는 /ws 경로로 WebSocket 연결을 허용하며, 클라이언트로부터 메시지를 받아서 다시 전송합니다.
> WebSocket 클라이언트 작성: FastAPI에서 WebSocket을 사용하기 위해서는 클라이언트도 필요합니다. WebSocket 클라이언트를 작성하고 FastAPI 서버와 연결해보세요.

<br/><br/>

### 다중 웹소캣 연결 관리(EX-단체 채팅방)
회원들이 연결된 소켓을 관리하려면 다음과 같은 접근 방법을 고려할 수 있습니다
1. 세션 관리: 각 클라이언트의 연결을 세션으로 추적하고 관리합니다.
2. 인증 및 권한 부여: 회원들의 연결을 관리할 때 인증 및 권한 부여를 구현하여 허가된 사용자만 연결할 수 있도록 합니다.
3. 상태 관리: 연결된 소켓의 상태를 추적하고 필요한 경우 연결을 종료하거나 다시 연결합니다.

> 2024-05-01 수정

```Python
    from fastapi import APIRouter, WebSocket
    
    router = APIRouter()
    chat_rooms = {}
   
   
    @app.websocket("/chat-api/{chat_room_id}")
    async def chat_room_websocket_endpoint(chat_room_id: int, websocket: WebSocket):
        # Socket Manager 정보 조회 후 없으면 생성
        logger.info(f"__________________________________________ Chat Room Setting START __________________________________________")
        if chat_room_id not in chat_rooms:
            chat_rooms[chat_room_id] = {
            "chat_room_id": chat_room_id
            , "socket_group": []
            }
            socket_group = chat_rooms[chat_room_id]["socket_group"]
        if websocket not in socket_group:
            socket_group.append(websocket)
    
        await websocket.accept()
        logger.info(f"__________________________________________ Chat Room Setting END __________________________________________")
        try:
            while True:
                if websocket.client_state == WebSocketState.DISCONNECTED:
                    raise WebSocketDisconnect
                message = await websocket.receive_text()
                logger.info(f"__________________________________________ GROUP WEBSOCKET REQUEST MESSAGE START __________________________________________")
                logger.info(f"{message}")
                logger.info(f"__________________________________________ GROUP WEBSOCKET REQUEST MESSAGE END __________________________________________")
                if message is None:
                    continue
                if message == "ping":
                    await single_send(websocket, "pong")
                else:
                    result = await chat_room_main_logic(chat_room_id, message)
                    if result is not None:
                        await broadcast(result, chat_rooms[chat_room_id]["socket_group"])
        except Exception as e:
            exception_type_name = type(e).__name__
            logger.error(f"########################### GROUP Disconnect Title : {exception_type_name}")
        finally:
            # 연결이 끊겼을 때, 채팅방 내에서 소캣 정보 제거 후 명시적으로 websocket.close()를 호출하여 연결을 정리
            if websocket in socket_group:
                socket_group.remove(websocket)
            await websocket.close()
   
   
    # 채팅을 친 당사자에게만 메시지 전송
    async def single_send(websocket: WebSocket, message: str):
        logger.info(f"__________________________________________ SINGLE_SEND MESSAGE START __________________________________________")
        await websocket.send_text(message)
        logger.info(f"__________________________________________ SINGLE_SEND MESSAGE END __________________________________________")

   
    # 채팅방 내 모든 소켓에 메시지 전송
    async def broadcast(message: str, socket_group: List[WebSocket]):
        for websocket in socket_group:
            logger.info(
            f"__________________________________________ BROADCAST MESSAGE START __________________________________________")
            await websocket.send_text(message)
            logger.info(
            f"__________________________________________ BROADCAST MESSAGE END __________________________________________")
    
   
    # 채팅 관련 로직 처리 함수
    async def chat_room_main_logic(chat_room_id, message):
        try:
            chat_data, result = await chat_room_router(json.loads(message))
            # 채팅방 종료 시 채팅방 정보 삭제
            if chat_data.chat_type_code == GameRoomGroupApiTypeCode.CLOSE_GAME_ROOM_GROUP.value:
                chat_rooms.pop(chat_room_id)
            return result
        except Exception as e:
            logger.error(f"########################### GROUP MainLogicError : {e}")
```
이러한 접근 방법을 활용하여 회원들이 연결된 소켓을 효과적으로 관리할 수 있습니다.
<br/><br/>

### Router를 통한 WebSocket 경로 관리
Router를 앱에 추가하여 해당 경로를 사용할 수 있도록 합니다. 다음과 같이 Router를 앱에 추가할 수 있습니다.
```Python
from fastapi import FastAPI
from . import my_router  # 여기서 my_router는 생성한 Router 모듈입니다.

app = FastAPI()

app.include_router(my_router)
```
<br/><br/><br/><br/>

## 글을 마치며
이러한 접근 방법을 활용하여 회원들이 연결된 소켓을 효과적으로 관리할 수 있습니다.
이를 통해 채팅서비스, 실시간 게임, 실시간 데이터 처리 등 다양한 서비스를 구현할 수 있습니다.<br/>
인증과 관련된 내용은 제 [FastAPI와 JWT](https://jd6186.github.io/FastAPI_JWT) 포스팅을 참고하시면 더 많은 정보를 얻을 수 있습니다. 다들 즐코하세요~ 🚀
<br/><br/>

* 참고자료
  * [FastAPI 공식 문서](https://fastapi.tiangolo.com/ko/)
