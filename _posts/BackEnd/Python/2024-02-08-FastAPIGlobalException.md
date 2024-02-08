---
layout: post
title: "[Python] FastAPI 'Global Exception'와 'Filter' 구현 가이드"
tags: [BackEnd PYTHON]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
오늘은 FastAPI 사용 시 Filter 및 Global Exception을 핸들링 할 수 있는 방법에 대해 알아보겠습니다.
<br/><br/><br/><br/>

## 본문
### Global Exception
Global Exception을 사용하는 이유는 API 서버에서 발생하는 모든 예외를 한 곳에서 통제하기 위함입니다.<br/>
이를 통해 예외 처리 로직을 중복으로 작성하지 않아도 되며, 예상하지 못한 예외 발생을 관리할 수 있습니다.<br/>
FastAPI 애플리케이션을 설정하고, exception_handler 어노테이션을 활용해 Handler를 제작합니다.<br/>
exception_handler 어노테션에 핸들링 하길 원하는 Exception을 파라미터로 작성합니다. 그 후 함수를 제작하여 해당 Exception을 처리하시면 되겠습니다.
```python
from fastapi import FastAPI, APIRouter, Request, HTTPException
from fastapi.responses import JSONResponse
from fastapi.logger import logger
from src.core.code.error_type_code import EXCEPTION_TYPE
from src.core.common.dto import ResponseDTO

############################## App & CORS Setting ##############################
app = FastAPI(docs_url="/api/docs", openapi_url="/api/openapi", redoc_url=None)

############################## Global Exception Handler ##############################
@app.exception_handler(HTTPException)
async def http_exception_handler(request, error_object):
    logger.error(f"시스템 오류가 발생했습니다. Error status_code: {error_object.status_code}, Message : {error_object.detail}")
    dto = ResponseDTO(
        result_data="시스템 오류 발생",
        status_code=error_object.status_code,
        status_message=error_object.detail
    )
    return JSONResponse(
        content=dto.dict()
    )


@app.exception_handler(Exception)
async def global_exception_handler(request, error_object):
    logger.error(f"예상하지 못한 오류가 발생했습니다.")
    dto = ResponseDTO(
        result_data="시스템 오류 발생",
        status_code=EXCEPTION_TYPE.ERROR_500.value["code"],
        status_message=EXCEPTION_TYPE.ERROR_500.value["message"]
    )
    return JSONResponse(
        content=dto.dict()
    )
```


### Filter
Filter를 사용하는 이유는 API 서버에 들어오는 모든 요청을 통제하기 위함입니다.<br/>
이를 통해 요청에 대한 로직을 중복으로 작성하지 않아도 되며, 예상하지 못한 요청을 관리할 수 있습니다.<br/>
FastAPI 애플리케이션을 설정하고, middleware 어노테이션을 활용해 Filter를 제작합니다.<br/>
middleware 어노테이션에 핸들링 하길 원하는 Request를 파라미터로 작성합니다. 그 후 함수를 제작하여 해당 Request를 처리하시면 되겠습니다.
```python
from fastapi import FastAPI, APIRouter, Request, HTTPException
from fastapi.logger import logger

############################## App & CORS Setting ##############################
app = FastAPI(docs_url="/api/docs", openapi_url="/api/openapi", redoc_url=None)

############################## Filter ##############################
@app.middleware("http")
async def application_filter(request: Request, call_next):
    # 단순 health-check 요청은 별도 작업없이 Return
    request_url = str(request.url)
    if '/health-check' in request_url:
        return
    # api 요청이 아니라면 반려
    if 'api' not in request_url:
        return
    
    # 정상 요청인 경우 로그 출력 후 다음 단계로 진행
    logger.debug("############# request_url : ", request_url)
    response = await call_next(request)
    return response
```
<br/><br/><br/><br/>


## 글을 마치며
오늘은 FastAPI에서 Global Exception과 Filter를 구현하는 방법에 대해 알아보았습니다.<br/>
이를 통해 예외 및 요청에 대한 통제를 효율적으로 관리할 수 있게 되셨을 겁니다. 궁금하신 점이나 추가적인 질문이 있으시다면 언제든지 댓글로 남겨주세요.<br/>
아래는 참고자료로 FastAPI 공식 문서를 첨부하겠습니다. 모두 FastAPI를 통해 효율적인 개발을 진행하시길 바랍니다.<br/>
감사합니다.
[FastAPI 공식 문서](https://fastapi.tiangolo.com/advanced/middleware/)

