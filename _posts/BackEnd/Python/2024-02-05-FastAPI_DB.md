---
layout: post
title: "[Python] FastAPI와 DataBase를 SQLAlchemy를 활용해 연동하는 방법"
tags: [BackEnd PYTHON]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
이 글에서는 Python, FastAPI, SQLAlchemy를 사용하여 MySQL 데이터베이스에 연결하는 방법을 단계별로 안내합니다.<br/>
또한, DB 세션을 효율적으로 관리하는 방법에 대한 가이드라인도 제공합니다.<br/>
<br/><br/><br/><br/>


## 본문
1. DB 설정을 위한 환경 설정
    - 라이브러리 설치:
      - ```
        pip install fastapi sqlalchemy pymysql
        ```
    - MySQL 설정:
      - MySQL 서버를 설치하고 실행합니다. 
      - 사용자 계정 및 데이터베이스를 만들어 준비해둡니다.
2. 모델 정의<br/>
    - models.py:
      - ```Python
        from sqlalchemy import Column, Integer, String, ForeignKey
        from sqlalchemy.orm import relationship
        
        Base = declarative_base()
        
        class User(Base):
            __tablename__ = "users"
    
            id = Column(Integer, primary_key=True)
            name = Column(String(255))
            email = Column(String(255))
        class Post(Base):
            __tablename__ = "posts"
    
            id = Column(Integer, primary_key=True)
            title = Column(String(255))
            content = Column(String(255))
            user_id = Column(Integer, ForeignKey("users.id"))
    
            user = relationship("User", foreign_keys=[user_id])
        ```
   - 설명:
     - Base는 모든 모델의 기본 클래스입니다.
     - User와 Post는 각각 사용자와 게시물 테이블을 나타내는 모델입니다.
     - Column은 테이블의 각 열을 정의합니다.
     - relationship은 두 모델 간의 관계를 정의합니다.

3. 데이터베이스 연결<br/>
   - database.py:
     - ```Python
         from sqlalchemy import create_engine

         engine = create_engine("mysql+pymysql://username:password@localhost/database_name")

         Base.metadata.create_all(engine)
       ```
   - 설명:
     - create_engine 함수는 데이터베이스 연결 문자열을 사용하여 엔진을 생성합니다.
     - metadata.create_all 메서드는 모델을 기반으로 테이블을 생성합니다.

4. FastAPI 애플리케이션<br/>
   - 폴더구조
     - ![folder.png](..%2F..%2F..%2Fassets%2Fimg%2F2024-02-05-FastAPI_DB%2Ffolder.png)

   - main.py:
     - ```Python
         from fastapi import FastAPI
    
         from app.routers import users
    
         app = FastAPI()
    
         app.include_router(users.router, prefix="/users")
       ```

   - routers/users.py:
     - ```Python
         from fastapi import APIRouter, Depends
         from pydantic import BaseModel
    
         from app.services import UserService
    
         router = APIRouter()
    
         class UserIn(BaseModel):
             name: str
             email: str
    
         class UserOut(BaseModel):
             id: int
             name: str
             email: str
    
         @router.get("/", response_model=List[UserOut])
         async def get_users(service: UserService = Depends()):
             return await service.get_all_users()
    
         @router.post("/", response_model=UserOut)
         async def create_user(user_in: UserIn, service: UserService = Depends()):
             return await service.create_user(user_in)
       ```

   - services/users.py:
     - ```Python
         from app.dao import UserDAO
    
         class UserService:
             def __init__(self, dao: UserDAO):
                 self.dao = dao
    
             async def get_all_users(self):
                 return await self.dao.get_all_users()
    
             async def create_user(self, user_in: UserIn):
                 return await self.dao.create_user(user_in)
       ```

   - dao/users.py:
     - ```Python
         from sqlalchemy.orm import Session
    
         from app.models import User
    
         class UserDAO:
             def __init__(self, session: Session):
                 self.session = session
    
             async def
       ```

5. DB 세션 관리 가이드라인
   - 세션 범위: 각 요청마다 새 세션을 생성하고 종료하는 것이 좋습니다. 세션 범위를 늘려야 하는 경우, with 블록을 사용하여 명확하게 정의해야 합니다.
   - 커밋 및 롤백: 데이터 변경 후에는 commit을 사용하여 변경 사항을 저장해야 합니다. 오류 발생 시 rollback을 사용하여 변경 사항을 되돌릴 수 있습니다.
   - 연결 풀: 성능 향상을 위해 SQLAlchemy 연결 풀을 사용할 수 있습니다.

<br/><br/><br/><br/>

## 글을 마치며
이 글에서는 Python, FastAPI, SQLAlchemy를 사용하여 MySQL 데이터베이스에 연결하는 방법을 단계별로 안내했습니다. 또한, DB 세션을 효율적으로 관리하는 방법에 대한 가이드라인도 작성해봤습니다.

- 글 핵심 내용 
  - Solid원칙에 따라 라우터, 서비스, DAO, 모델 분리하여 코드의 역할과 책임을 명확하게 구분했습니다. 
  - DTO를 활용하여 클라이언트로부터 받는 데이터와 모델 간에 분리했습니다. 
  - Depends(Dependency Injection)를 활용해 코드의 결합성을 낮추고 테스트 가능성을 높였습니다.
- 추가 개선 가능한 목록
  - 테스트 코드 작성: 단위 테스트 및 통합 테스트를 작성하여 코드의 안정성을 높일 수 있습니다.
  - 환경 변수 관리: 환경 변수를 사용하여 데이터베이스 연결 정보 및 기타 설정을 관리할 수 있습니다. 
  - 에러 처리: 예외 처리를 추가하여 예상치 못한 오류 발생 시 사용자에게 적절한 메시지를 제공할 수 있습니다.
  - SQL 관리: ORM 사용 시 저는 DDL 및 기본 Insert문을 App 버전별로 관리하기 때문에 이를 별도로 관리하는 것을 선호합니다.
  <br/>

> 제가 위에서 보여드린 코드는 기본적인 예시이며, 실제 프로젝트에서는 추가적인 기능 및 보안 고려 사항을 추가해야 합니다.<br/>

<br/>


이상으로 글을 마치겠습니다.<br/>
다음에는 FastAPI를 활용해 테스트를 구현하는 벙법에 대해 알아보겠습니다.<br/>
이 글이 도움이 되었기를 바라겠습니다. 감사합니다!
<br/><br/><br/>

- 참고 사이트
  - SQLAlchemy 공식 문서: https://docs.sqlalchemy.org/en/14/
  - FastAPI 공식 문서: https://fastapi.tiangolo.com/
  - PyMySQL 공식 문서: https://pymysql.readthedocs.io/en/latest/
  - SOLID 원칙: https://en.wikipedia.org/wiki/SOLID
  - Dependency Injection: https://en.wikipedia.org/wiki/Dependency_injection