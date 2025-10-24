## FastAPI - DB 저장소 - 데이터수집기 간에 어떤식의 통신 구조가 가장 효과적일까?

세 프로젝트(FastAPI 앱 · DB저장 서비스 · 데이터수집기) 사이에서 **데이터 명세를 한 곳**에서 관리해야 통신 오류가 줄고, 변경도 통제됩니다.

가장 깔끔한 해법은 **프레임워크에 독립적인 “계약(contracts) 패키지”**를 따로 만들어 그 안에 **Pydantic 모델(스키마)**을 선언하고,
세 프로젝트가 **같은 버전의 그 패키지**를 의존하도록 하는 겁니다.

아래처럼 정리해두면 관리가 편합니다.

---

**추천 구조**

```
contracts/                 # 공용 스키마 전용(프레임워크 독립)
  pyproject.toml
  contracts/
    __init__.py
    __version__.py
    users.py               # Pydantic DTO (Create/Read/Update, Event 등)
    items.py
    events.py              # (선택) 메시지/이벤트 스키마
    common.py              # 공통 타입(IDs, enums)

fastapi-app/
  app/
    api/                   # 여기선 contracts의 DTO를 그대로 사용
    services/
    mappers.py             # DB문서 <-> DTO 변환
  pyproject.toml           # deps: contracts==x.y.z

db-service/                # Mongo/Beanie 구현
  data_mongo/
    beanie_models.py       # Beanie Document (저장용)
    mappers.py             # Document <-> DTO 변환 (contracts 의존)
    repo_impl.py
  pyproject.toml           # deps: contracts==x.y.z

collector/                 # 크롤러/배치/수집기
  collector/
    pipelines/
    emitters/              # API/큐에 보낼 때 contracts DTO 인스턴스 생성
  pyproject.toml           # deps: contracts==x.y.z
```

**핵심 원칙**

- contracts에는 **Pydantic과 표준 라이브러리만** 사용(FastAPI/Beanie import 금지).
    
- 각 프로젝트는 contracts를 **읽기만** 하고, 자체 내부 모델(예: Beanie Document)과는 **매핑**으로 분리.
    

---

**예시 스키마 (contracts/users.py)**

```python
# contracts/users.py
from pydantic import BaseModel, EmailStr, Field
from typing import Optional

class UserId(BaseModel):
    value: str = Field(min_length=1)

class UserCreate(BaseModel):
    email: EmailStr
    name: str

class UserRead(BaseModel):
    id: str
    email: EmailStr
    name: str

class UserUpdate(BaseModel):
    name: Optional[str] = None
```

> FastAPI에서는 response_model=UserRead로 바로 쓰고,

> DB서비스에서는 UserCreate → Beanie Document로 매핑해 insert, 조회 후 UserRead로 매핑해 반환합니다.

> 수집기는 외부에서 긁은 raw dict를 UserCreate(**raw)로 검증 후 전송하면 됩니다.

---

### 매핑 위치 예시

  

**DB 서비스 (Beanie)**

```python
# data_mongo/beanie_models.py
from beanie import Document, Indexed
from pydantic import EmailStr

class UserDoc(Document):
    email: Indexed(EmailStr, unique=True)
    name: str
    class Settings:
        name = "users"
```

```python
# data_mongo/mappers.py
from contracts.users import UserCreate, UserRead
from .beanie_models import UserDoc

def to_doc(dto: UserCreate) -> UserDoc:
    return UserDoc(**dto.model_dump())

def to_read(doc: UserDoc) -> UserRead:
    return UserRead(id=str(doc.id), email=doc.email, name=doc.name)
```

**FastAPI 앱**

```python
# app/api/routes/users.py
from fastapi import APIRouter
from contracts.users import UserCreate, UserRead
from app.services.user_service import user_service

router = APIRouter(prefix="/users", tags=["users"])

@router.post("", response_model=UserRead)
async def create_user(data: UserCreate):
    return await user_service.create(data)
```

---

### 버전 전략(아주 중요)

- contracts는 **SemVer(예: 1.4.2)**로 관리하고, 각 서비스는 contracts==1.4.*처럼 **엄격히 핀(Pin)** 하세요.
    
- **호환성 원칙**
    
    - **비파괴 변경(Non-breaking)**: 필드 추가는 **Optional**로, 기본값 제공.
        
    - **파괴 변경(Breaking)**: 필드 이름/타입 변경·삭제는 **Major 버전 업**.
        
    - 필요한 경우 version: Literal["v1"] 같은 구분 필드를 이벤트 페이로드에 포함.

- **마이그레이션 기간** 동안은 alias를 활용해 구버전 필드도 잠시 허용 가능합니다.
    

---

### 다국어/타언어 고려가 있다면

- contracts의 Pydantic 모델에서 model_json_schema()로 **JSON Schema**를 뽑아 저장해두고,
    
    다른 언어(Typescript/Go 등) 클라이언트가 그 스키마로 **타입 생성**을 하게 하면 이상적입니다.
    
- 또는 Avro/Protobuf를 검토할 수 있지만, **Python 위주**라면 Pydantic+JSON Schema로 충분한 경우가 많습니다.
    

---

**결론**

- **Pydantic으로 “데이터 명세”를 통일**하는 것이 가장 실용적입니다.
    
- 그 명세는 **별도 공용 패키지(contracts)**로 분리해 **세 프로젝트가 동일 버전**을 의존하게 하세요.
    
- 각 프로젝트는 **자체 내부 모델(Beanie 등)**과는 **매핑 레이어**로 연결합니다.
    
- 이 구조가 변경 관리, 테스트, 교체(예: Mongo→다른 DB), 팀 협업 모두에서 **장기적으로 제일 안정적**입니다.
    

---

## 이 구조를 도시화해서 설명한다면?

**전체 개념도**

```
┌───────────────────────────────┐
│         📦 contracts          │
│  ───────────────────────────  │
│    Pydantic 모델(데이터 명세)     │
│     ├─ UserCreate             │
│     ├─ UserRead               │
│     └─ UserUpdate             │
│  (FastAPI, DB, Collector 공통) │
└──────────────┬────────────────┘
               │   (pip install contracts==1.0.0)
   ┌───────────┼───────────┬────────────┐
   ▼           ▼           ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│ FastAPI  │ │  DB App  │ │ Collector│
│  (API)   │ │ (Beanie) │ │ (Crawler)│
├──────────┤ ├──────────┤ ├──────────┤
│ api/     │ │ beanie_  │ │ emitters/│
│ schemas/ │ │ models.py│ │ pipelines│
│ routes/  │ │ mappers  │ │          │
└────┬─────┘ └────┬─────┘ └────┬─────┘
     │              │            │
     │ Pydantic DTO │            │
     │ import       │            │
     ▼              ▼            ▼
 요청/응답 검증    DB 저장/조회   외부데이터 수집후 검증
  (UserCreate,   (UserDoc↔DTO)    (DTO 검증 후 전송)
   UserRead)
```

---

### 계층별 역할 한눈에

|**계층**|**위치**|**주요 역할**|**예시**|
|---|---|---|---|
|**contracts**|/contracts/|**데이터 명세** (Pydantic 모델 정의)|UserCreate, UserRead|
|**FastAPI App**|/fastapi-app/|API 라우팅, 요청/응답 검증|/users 엔드포인트에서 DTO 사용|
|**DB App**|/db-service/|Beanie Document, DB 저장/조회|UserDoc ↔ DTO 매핑|
|**Collector**|/collector/|데이터 수집/전송, DTO 검증 후 송신|UserCreate(**raw) 검증 후 API POST|

---

### 데이터 흐름도 (데이터가 이동하는 경로)

```
[Collector] 
   └── 수집한 raw dict
         ↓ (Pydantic 검증)
      UserCreate 모델 인스턴스
         ↓ (HTTP 전송)

           ↓↓↓ API 호출 ↓↓↓

[FastAPI App]
   └── /users POST 엔드포인트
         ↓ (Pydantic으로 입력 검증)
      UserCreate -> 서비스 -> UserRead 반환
         ↓ (응답 직렬화)

           ↓↓↓ 내부 호출 ↓↓↓

[DB App]
   └── UserCreate DTO 수신
         ↓ (매퍼 변환)
      UserDoc (Beanie Document)
         ↓ (MongoDB 저장)
      저장 결과 -> UserRead DTO 반환
```

> 즉, **세 프로젝트가 모두 같은 Pydantic 모델(UserCreate, UserRead 등)** 을 공유하기 때문에
>
> 구조나 필드가 달라서 생길 수 있는 통신 오류가 없어집니다.


---

## 이런 구조(DB가 외부서비스)에서 FastAPI의 구조예시

> FastAPI는 **요청/응답 + 인증/권한 + 라우팅**만 담당.
>
> 데이터 저장/조회는 **db-service**가 담당하고, 둘은 HTTP/RPC로 통신.

```
fastapi-app/
  app/
    main.py                 # lifespan, 미들웨어, 라우팅 묶기
    core/
      config.py             # settings(.env), 외부 서비스 URL 등
      security.py           # JWT/OAuth2 등
      errors.py             # 예외 ↔ HTTP 변환
    api/
      deps.py               # 공통 DI(현재 사용자, 권한 등)
      routes/
        users.py
        items.py
    clients/                # ← 외부 서비스(= db-service) 클라이언트
      db_client.py          # httpx.AsyncClient 래퍼, 재시도/타임아웃
    schemas/                # ← contracts를 그대로 import해서 사용
      (대부분 비어도 됨: from contracts.users import UserCreate, UserRead)
  tests/
    api/
    e2e/
```

**핵심 포인트**

- **models/, db/, repositories/** 폴더 없음(데이터 계층은 바깥으로 분리).
    
- clients/db_client.py에서 **db-service API**를 호출하고, 실패/재시도/타임아웃을 표준화.
    
- **contracts** 패키지의 Pydantic DTO를 바로 response_model 등에 사용.
    

**간단 예시:**

```python
# app/api/routes/users.py
from fastapi import APIRouter, Depends
from contracts.users import UserCreate, UserRead
from app.clients.db_client import DBClient, get_db

router = APIRouter(prefix="/users", tags=["users"])

@router.post("", response_model=UserRead)
async def create_user(data: UserCreate, db: DBClient = Depends(get_db)):
    return await db.create_user(data)
```

이 구조의 장점

- FastAPI 앱이 **완전히 얇아짐**: 변경 영향이 작고, 배포/스케일이 단순.
    
- DB 교체/마이그레이션을 **db-service 안에서** 처리 가능.
    
- 팀 간 경계가 명확(플랫폼/인프라 vs API 팀).