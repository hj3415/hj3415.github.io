**포트/어댑터 + 조립 지점 분리**. 
상위는 “추상(Port)”만 의존. 
하위(DB)는 “구현(Adapter)”만 가진다. 
조립은 한 곳에서만 한다.

```
contracts/                         # 공용 스키마(프레임워크 독립)
  pyproject.toml
  contracts/
    __init__.py
    __version__.py
    users.py
    items.py
    events.py
    common.py

fastapi-app/
  app/
    api/                           # DTO 입·출력만
    services/                      # 비즈니스 로직
      users_service.py
    ports/                         # ← 추상(Interface/Protocol) 정의
      users_repo.py                # UserRepo(Protocol | ABC)
    mappers.py                     # DTO ↔ 도메인 Entity 변환(있다면)
    di.py                          # ← 조립 지점(의존성 주입: 어떤 구현체 쓸지 결정)
  pyproject.toml                   # deps: contracts==x.y.z, db-service==x.y.z (runtime wiring용)

db-service/                        # 인프라 계층(구현 어댑터들)
  data_mongo/
    beanie_models.py               # Beanie Document
    mappers.py                     # Document ↔ DTO 변환(contracts 의존)
    repo_impl.py                   # ← Adapter: UsersRepoMongo(UserRepo 구현)
  # (필요 시) data_sql/ ...        # 다른 DB도 같은 인터페이스로 구현
  pyproject.toml                   # deps: contracts==x.y.z, beanie, motor ...

collector/                         # 크롤러/배치/수집기
  collector/
    pipelines/
      users_pipeline.py
    emitters/
      kafka_emitter.py
    ports/                         # ← 선택: Collector 자체 Port(얇은 Protocol)
      users_repo.py                # UserRepo(Protocol) 재정의하거나 fastapi 포트 재사용
    factory.py                     # ← 조립 지점: env로 구현체 선택
  pyproject.toml                   # deps: contracts==x.y.z, db-service==x.y.z
```

핵심 규칙:

- 상위 계층(FastAPI, Collector)은 **app/ports** **또는** **collector/ports****의 Port** 에만 의존.
    
- 하위 계층(DB 구현)은 **Port를 구현한 Adapter** 만 제공. 상위를 모름.
    
- **조립 지점은 단 하나**: fastapi-app/app/di.py, collector/factory.py.
    
- api/, services/, pipelines/, emitters/에서는 **구현체 import 금지**. 오직 Port 타입만 사용.

검증 포인트:

- DIP 유지: 상위는 Port만 의존. 하위는 Port 구현. 조립은 별도.
    
- DB 교체: DB_KIND 변경 혹은 di/factory만 수정.
    
- 테스트: 상위 테스트는 Fake Repo로 대체 가능.

요약:

- **Port(Protocol) 정의**: app/ports/, collector/ports/.
    
- **Adapter(DB 구현)**: db-service/**/repo_impl.py.
    
- **조립 지점 단일화**: app/di.py, collector/factory.py.
    
- **Mapper 분리**: 상위(mapper)와 인프라(mapper) 역할 구분.

---

이 구성은 **Clean Architecture + DIP(의존성 역전) + 패키지 분리 배포**를 동시에 만족시키는 형태입니다.
각 폴더가 “무엇을 책임지고, 누구에게 의존하는지”를 중심으로 정리하고,
흐름/와이어링/테스트 관점까지 한 번에 이해되도록 설명드릴게요.

---

### 최상위 개요

```
contracts/          # 공용 계약(프레임워크 독립) : DTO/이벤트/공통타입만
fastapi-app/        # 앱 레이어: API + 서비스 + 포트 + DI (런타임 결합 지점)
db-service/         # 인프라 레이어: 실제 저장소 구현(예: Mongo/SQL/...)
collector/          # 배치/크롤러/파이프라인: 계약 재사용, 필요 시 포트 정의
```

**의존 방향(한 줄 요약)**

contracts ← 아무도 의존하지 않음(최안쪽)

fastapi-app, db-service, collector → **contracts** 에만 공통 의존

fastapi-app 은 런타임에 **db-service** 구현을 선택해 주입(DI)

---

### contracts/ (공용 계약 레이어)

- **역할**: 데이터의 모양(스키마)만 정의. 프레임워크/DB에 독립.
    
- **구성**
    
    - users.py, items.py: Create/Read/Update DTO, 검색/페이징 입력 등
        
    - events.py: 이벤트(예: UserRegisteredEvent)
        
    - common.py: ID, Enum, 공통 BaseModel 등
        
    
- **핵심 포인트**
    
    - _이곳엔 비즈니스 로직/저장 로직 없음._
        
    - 모든 하위 시스템(fastapi-app, db-service, collector)이 **같은 계약으로 소통**.
        
    

---

### fastapi-app/ (애플리케이션 레이어)

**(1) api/ — 입/출력만**

- FastAPI 라우터: **입력 DTO 검증 → 서비스 호출 → 출력 DTO 반환**
    
- **비즈니스 규칙 없음** (컨트롤러 역할만)

```python
# app/api/users.py
@router.post("/users", response_model=UserRead)
async def create_user(user_in: UserCreate, svc: UsersService = Depends(get_user_service)):
    return await svc.create_user(user_in)
```

API는 Service만 본다:

```python
# fastapi-app/app/api/users.py
from fastapi import APIRouter, Depends
from contracts.users import UserCreate, UserRead
from app.di import provide_users_service, UsersService

router = APIRouter()

@router.post("/", response_model=UserRead)
async def create_user(dto: UserCreate, svc: UsersService = Depends(provide_users_service)):
    return await svc.create_user(dto)
```

**(2) services/ — 유즈케이스/도메인 로직**

- **포트(인터페이스)** 에 의존하고, 구체 구현은 모름
    
- 트랜잭션/정합성/도메인 규칙을 책임

```python
# app/services/users_service.py
class UsersService:
    def __init__(self, repo: UsersRepo):
        self.repo = repo

    async def create_user(self, dto: UserCreate) -> UserRead:
        # (예) 중복검사/암호화/정책 적용 등
        created = await self.repo.create(dto)
        return created
```

Service는 Port만 본다:

```python
# fastapi-app/app/services/users_service.py
from app.ports.users_repo import UserRepo
from contracts.users import UserCreate, UserRead

class UsersService:
    def __init__(self, repo: UserRepo):
        self.repo = repo

    async def create_user(self, dto: UserCreate) -> UserRead:
        return await self.repo.save(dto)
```

**(3) ports/ — 추상(Protocol/ABC)**

- **DIP 실현 핵심**: 서비스는 여기 정의된 **인터페이스**만 바라봄
    
- DB, 메시징, 외부 API 등 “어댑터”가 이 인터페이스를 구현

```python
# app/ports/users_repo.py
class UsersRepo(Protocol):
    async def create(self, dto: UserCreate) -> UserRead: ...
    async def get(self, user_id: str) -> UserRead | None: ...
    async def list(self, limit: int = 50) -> list[UserRead]: ...
```

간단 Port 예시 (Protocol 권장):

```python
# fastapi-app/app/ports/users_repo.py
from typing import Protocol, Optional
from contracts.users import UserCreate, UserRead, UserUpdate

class UserRepo(Protocol):
    async def save(self, dto: UserCreate) -> UserRead: ...
    async def get_by_id(self, id: str) -> Optional[UserRead]: ...
    async def update(self, id: str, dto: UserUpdate) -> Optional[UserRead]: ...
    async def delete(self, id: str) -> bool: ...
```


**(4) mappers.py**

**— DTO ↔ 도메인 엔티티(있다면)**

- 도메인 엔티티를 운영한다면 여기서 변환. DB 모델과 API DTO를 **느슨하게 연결**.
    
**(5) di.py — 조립/와이어링 지점**

- **실행 시점에** 어떤 구현을 쓸지 결정 (예: Mongo vs SQL)
    
- 환경변수/설정에 따라 구현체 스위칭

```python
# app/di.py
from db_service.data_mongo.repo_impl import UsersRepoMongo

def get_user_service() -> UsersService:
    repo = UsersRepoMongo(...)             # ← 구체 구현 선택
    return UsersService(repo)
```

FastAPI 조립(di):

```python
# fastapi-app/app/di.py
import os
from typing import Callable
from app.ports.users_repo import UserRepo
from app.services.users_service import UsersService

def provide_user_repo() -> UserRepo:
    kind = os.getenv("DB_KIND", "mongo")
    if kind == "mongo":
        from db_service.data_mongo.repo_impl import UsersRepoMongo
        return UsersRepoMongo()
    raise ValueError(f"Unsupported DB_KIND={kind}")

def provide_users_service() -> UsersService:
    return UsersService(repo=provide_user_repo())
```

**(6) fastapi-app/pyproject.toml**

- 런타임 의존성: contracts==x.y.z, db-service==x.y.z
    
- 의미: **API/서비스는 계약과 구현 패키지를 가져와서 조립**한다.
    

---

### db-service/ (인프라 레이어: 구현 어댑터)

- **역할**: ports 의 인터페이스를 **실제로 구현**하는 어댑터들.
    
- 연결되는 기술 스택: Mongo+Beanie, SQLAlchemy, Redis, S3 등 무엇이든 가능.
    
- **contracts 에만 의존**(DTO 변환을 위해). FastAPI/서비스에 직접 의존하지 않음.

**(1) data_mongo/beanie_models.py**

- Beanie Document 정의 (저장 전용 모델)

```python
class UserDoc(Document):
    email: str
    name: str
    created_at: datetime
```

**(2) data_mongo/mappers.py**

- Document ↔ DTO 변환

```python
def doc_to_dto(doc: UserDoc) -> UserRead:
    return UserRead(id=str(doc.id), email=doc.email, name=doc.name)
```

**(3) data_mongo/repo_impl.py**

- UsersRepo 구현체 (Mongo 버전)

```python
class UsersRepoMongo(UsersRepo):
    async def create(self, dto: UserCreate) -> UserRead:
        doc = UserDoc(**dto.model_dump())
        await doc.insert()
        return doc_to_dto(doc)
```

DB Adapter 예시:

```python
# db-service/data_mongo/repo_impl.py
from contracts.users import UserCreate, UserRead, UserUpdate
from .beanie_models import UserDocument
from .mappers import to_document, to_read, apply_update

class UsersRepoMongo:
    async def save(self, dto: UserCreate) -> UserRead:
        doc = to_document(dto); await doc.insert(); return to_read(doc)

    async def get_by_id(self, id: str):
        doc = await UserDocument.get(id); return to_read(doc) if doc else None

    async def update(self, id: str, dto: UserUpdate):
        doc = await UserDocument.get(id); 
        if not doc: return None
        apply_update(doc, dto); await doc.save(); return to_read(doc)

    async def delete(self, id: str) -> bool:
        doc = await UserDocument.get(id); 
        if not doc: return False
        await doc.delete(); return True
```

**(4) 다중 저장소 지원**

- data_sql/ 같은 폴더로 **같은 인터페이스**를 구현하면, DI에서 손쉽게 교체 가능.

**(5) db-service/pyproject.toml**

- 의존성: contracts==x.y.z, beanie, motor …
    
- **앱에 종속되지 않고, 오직 계약과 인프라 라이브러리만 의존**.
    

---

### collector/ (배치/크롤러/파이프라인)

- **역할**: 크롤링/ETL/주기적 작업.
    
- contracts 재사용으로 **앱과 같은 스키마**를 입력·출력에 사용.
    
- 필요 시 collector 내부 ports/ 로 자체 인터페이스를 얇게 정의하고, factory.py 에서 와이어링.

**(1) pipelines/users_pipeline.py**

- 사용자 데이터를 수집/가공 → db-service 저장소를 통해 저장하거나
    
    또는 emitters/ 로 카프카 등 브로커에 이벤트 발행.

```python
async def run():
    raw = await crawl_users()
    dto = UserCreate(**transform(raw))
    await users_repo.create(dto)          # db-service 구현 사용
    await kafka_emitter.emit(UserCreatedEvent.from_dto(dto))
```

Pipeline 사용:

```python
# collector/collector/pipelines/users_pipeline.py
from contracts.users import UserCreate
from collector.factory import get_user_repo

async def run_user_ingest(payload: dict):
    dto = UserCreate(**payload)
    repo = get_user_repo()
    await repo.save(dto)
```

**(2) emitters/kafka_emitter.py**

- events.py 계약을 메시지로 사용 (스키마 일관성 확보)

**(3) ports/users_repo.py**

- 필요하면 Collector가 직접 쓸 **얇은 포트**도 정의 가능
    
    (혹은 fastapi의 포트를 그대로 재사용해도 OK)

**(4) factory.py**

- **환경변수/설정으로 구현체 선택**(Mongo, SQL, Mock 등)

```python
def build_users_repo() -> UsersRepo:
    if os.getenv("DB_BACKEND") == "mongo":
        return UsersRepoMongo(...)
    return UsersRepoSql(...)
```

Collector 조립(factory):

```python
# collector/collector/factory.py
import os
from typing import Protocol
from contracts.users import UserCreate, UserRead

class UserRepo(Protocol):
    async def save(self, dto: UserCreate) -> UserRead: ...

def get_user_repo() -> UserRepo:
    kind = os.getenv("DB_KIND", "mongo")
    if kind == "mongo":
        from db_service.data_mongo.repo_impl import UsersRepoMongo
        return UsersRepoMongo()
    raise ValueError("Unsupported DB_KIND")
```

**(5) collector/pyproject.toml**

- 의존성: contracts==x.y.z, db-service==x.y.z
    
- 의미: **수집기는 계약과 저장소 구현만 끌어와 조립**.
    

---

### 실행/배포/버전 전략

- 각 패키지(contracts, db-service, fastapi-app, collector)는 **독립 배포** 가능.
    
- **버전 고정**으로 계약 일관성 유지:
    
    - fastapi-app/pyproject.toml → contracts==1.4.2, db-service==1.4.2
        
    - collector/pyproject.toml → 동일 버전 고정
        
    
- **릴리즈 순서 권장**:
    
    1. contracts 변경/배포 → 2) db-service 적응/배포 → 3) fastapi-app/collector 업데이트/배포

---

### 테스트 전략(핵심만)

- **contracts**: Pydantic 검증/시리얼라이즈 테스트
    
- **db-service**: 저장소 구현에 대한 **단위 테스트**(로컬 Mongo/테스트컨테이너)
    
- **fastapi-app**: 서비스 레벨 테스트 시 **포트에 대한 Fake/Mock** 주입 → 비즈니스 로직만 검증
    
- **e2e**: 앱+DB 통합, 또는 컬렉터 파이프라인 통합
    

---

### 이 구조의 장점

1. **강한 경계**: API/서비스/저장소/배치가 명확히 분리되어 변경 영향 최소화
    
2. **DIP 구현**: 서비스는 인터페이스만 의존 → 저장소 교체 용이
    
3. **스키마 일관성**: contracts 단일 소스 → 데이터/이벤트 형태가 시스템 전반에서 동일
    
4. **병렬 개발**: 팀별(앱/인프라/배치)로 독립 개발·배포 가능
    
5. **테스트 용이**: 각 레이어를 독립적으로 단위 테스트 가능
    

---

### 최소 예시(끝판 요약)

**포트**

```python
# app/ports/users_repo.py
class UsersRepo(Protocol):
    async def create(self, dto: UserCreate) -> UserRead: ...
```

**구현체**

```python
# db_service/data_mongo/repo_impl.py
class UsersRepoMongo(UsersRepo):
    async def create(self, dto: UserCreate) -> UserRead:
        doc = UserDoc(**dto.model_dump())
        await doc.insert()
        return doc_to_dto(doc)
```

**DI**

```python
# app/di.py
def get_user_service() -> UsersService:
    repo = UsersRepoMongo(...)  # or UsersRepoSql(...)
    return UsersService(repo)
```

**API**

```python
# app/api/users.py
@router.post("/users", response_model=UserRead)
async def create_user(user_in: UserCreate, svc: UsersService = Depends(get_user_service)):
    return await svc.create_user(user_in)
```
