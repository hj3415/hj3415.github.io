### 추천 구조예시

```
collector/
  pipelines/
    users_http_pipeline.py        # httpx 기반
    users_playwright_pipeline.py  # playwright 기반
  sources/
    http_client.py                # httpx Client + retry + header template
    playwright_client.py          # login/state, wait 전략 공통화
  sinks/
    kafka_emitter.py              # Emitter 추상 인터페이스 구현
    outbox_repo.py                # (선택) outbox 패턴
  ports/
    users_repo.py                 # 도메인 쓰기/읽기 포트
    emitter.py                    # Emitter Port(Interface)
  commons/
    rate_limit.py                 # 레이트리미터, backoff
    parsers.py                    # bs4/lxml/selectolax 파서 유틸
    validators.py                 # pydantic 검증
    observability.py              # loguru/rich/metrics
  factory.py
```

1. 수집 단계 계층화 명시
    
    pipelines/ 아래에 소스별 파이프라인을 나누고, 공통 유틸은 commons/로 분리:

2. **Emitter도 Port로 추상화**
    
    Kafka 외 SQS/Kinesis/Redis Stream으로 바꿀 수 있게:

```python
# ports/emitter.py
from typing import Protocol, Mapping, Any
class Emitter(Protocol):
    def emit(self, topic: str, key: str | None, value: Mapping[str, Any]) -> None: ...
```

3. **Idempotency & Dedup**
    
    동일 레코드 중복 방지를 위해 **natural key**(예: user_id + source_updated_at) 기반 Upsert를 users_repo에 포함:
    

```python
class UsersRepo(Protocol):
    def upsert(self, user: UserDTO, natural_key: tuple[str, ...]) -> None: ...
```

3. 수집 측에도 **seen-cache**(in-memory/LRU/Redis) 지원 추천.
    
4. **Outbox/Transactional Emitter (선택)**
    
    DB 트랜잭션 내 Write → Outbox 기록 → 별도 데몬이 Outbox→Kafka 전송.
    
    “DB 반영은 됐는데 카프카는 실패” 같은 **at-least-once** 문제를 줄입니다.
    
5. **Backoff / Rate limit / Circuit breaker**
    
    - tenacity로 재시도(지수 백오프)
        
    - asyncio.Semaphore 또는 토큰버킷으로 동시 요청 수 제한
        
    - 반복 실패 시 **회로차단(circuit breaker)** 간단 구현
        
    
6. **Playwright와 httpx의 책임 분리**
    
    - Playwright는 “세션 생성/쿠키/렌더링/네트워크 인터셉트”만 담당
        
    - 가능한 한 **API 직공(httpx)** 으로 전환 (성능↑)
        
    - Network sniffer로 API 추출 → sources/http_client가 담당
        
    
7. **구성/비밀관리**
    
    - pydantic-settings로 Settings 관리(ENV→객체 변환)
        
    - API 키, 프록시, 타임아웃, 동시성, 리트라이 정책을 전부 설정화
        
    
8. **관측 가능성(Observability)**
    
    - loguru 구조화 로그(JSON) + 파일 로테이션
        
    - rich 진행률/테이블로 배치 가시성 확보
        
    - 메트릭(성공/실패/재시도 횟수, 처리량, 레이턴시) 카운터 노출
        
    
9. **테스트 전략**
    
    - 파이프라인 단위 테스트: ports를 **mocker/monkeypatch**로 대체
        
    - 파서 테스트: HTML fixture로 **bs4/selectolax** 파싱 정확도 검증
        
    - E2E(옵션): 로컬 Kafka + 테스트 DB(docker-compose)
        
    
10. **그레이스풀 종료**

- SIGINT/SIGTERM 핸들링 → 현재 작업 마무리 후 종료
    
- Playwright 브라우저/AsyncClient clean-up 철저

---

### 전체 흐름 비유로 보기

컬렉터(Collector)는 이렇게 생겼다고 보면 돼요

> **Source → Pipeline → Sink**

즉,

- **Source**: 데이터를 “어디서 가져올지”
    
- **Pipeline**: 데이터를 “어떻게 가공할지”
    
- **Sink**: 데이터를 “어디로 보낼지”
    

이 전체를 하나로 엮는 게 factory

공통 로직이나 유틸리티를 모아둔 게 commons

데이터 저장이나 외부 시스템 접근의 인터페이스가 ports

---

### 각 폴더의 역할 (쉽게)

|**이름**|**역할**|**비유**|**예시**|
|---|---|---|---|
|**source**|“데이터를 가져오는 입구”|신문기자가 현장에 나가서 인터뷰하는 곳|httpx로 API 호출, Playwright로 웹페이지 읽기|
|**pipeline**|“가져온 데이터를 가공·정리하는 통로”|편집기자가 기사 다듬는 과정|raw JSON → DTO 검증 → 저장소에 전달|
|**sink**|“가공된 데이터를 내보내는 출구”|신문을 인쇄하고 배포하는 과정|DB 저장, Kafka 전송, CSV 저장|
|**ports**|“입출구 규격(인터페이스)”|콘센트 규격 같은 것|UserRepo, Emitter 같은 추상 정의|
|**commons**|“공통 도구함”|기자들이 쓰는 공용 도구|로그, 파서(bs4/lxml), 유효성 검사, retry, 설정|

---

### 예시로 한 번 돌려보기

**상황**

> “네이버 금융에서 기업 정보를 수집해서 DB에 저장하고, Kafka에도 전송하고 싶다.”

---

#### 1. source/

> → 데이터를 어디서 _가져오는가_

```python
# sources/http_client.py
import httpx

def fetch_company(code: str):
    url = "https://navercomp.wisereport.co.kr/v2/company/cF4002.aspx"
    params = {...}
    r = httpx.get(url, params=params)
    return r.json()
```

- “데이터를 들고 오는 사람 (기자)”

→ httpx, Playwright, requests, selenium 같은 게 여기 들어갑니다.

---

#### 2. pipeline/

> → 들어온 데이터를 _어떻게 정리/검증/저장할지_

```python
# pipelines/users_http_pipeline.py
from typing import Iterable
from ports.users_repo import UsersRepo
from ports.emitter import Emitter
from contracts.users import UserDTO
from commons.validators import validate_user

def run(fetch_iter: Iterable[dict], repo: UsersRepo, emitter: Emitter, topic: str):
    for raw in fetch_iter:
        user = UserDTO.model_validate(raw)  # pydantic 검증
        validate_user(user)
        repo.upsert(user)
        emitter.emit(topic, key=str(user.id), value=user.model_dump())
```

- “데이터를 깔끔하게 정리하고 각 부서에 전달하는 편집기자”

→ 검증(pydantic), 변환, 저장, 이벤트 발행 등이 여기서 일어납니다.

---

#### 3. sink/

> → 최종 데이터를 _어디로 보낼지_(어댑터 역할 )

```python
# sinks/kafka_emitter.py
from kafka import KafkaProducer
from ports.emitter import Emitter

class KafkaEmitter(Emitter):
    def __init__(self, bootstrap):
        self.producer = KafkaProducer(bootstrap_servers=bootstrap)

    def emit(self, topic, key, value):
        self.producer.send(topic, key=key.encode(), value=json.dumps(value).encode())
```

- “정리된 기사를 신문사 서버에 넘기는 부서”

→ Kafka, Redis Stream, S3, DB 등으로 “내보내는” 역할이에요.

---

#### 4. ports/

> → 위 pipeline에서 사용하는 _인터페이스 규격 정의_

```python
# ports/users_repo.py
from typing import Protocol
from contracts.users import UserDTO

class UsersRepo(Protocol):
    def upsert(self, user: UserDTO) -> None: ...
```

```python
# ports/emitter.py
from typing import Protocol, Mapping, Any

class Emitter(Protocol):
    def emit(self, topic: str, key: str | None, value: Mapping[str, Any]) -> None: ...
```

- “신문사 내부의 규격서”

> ‘기사 데이터는 이런 구조로 넘겨야 한다’ 같은 규칙을 정의

이걸 이용하면 Mongo, PostgreSQL, MySQL 등 어떤 DB로 바꿔도

pipeline 쪽 코드는 바꿀 필요 없습니다.

---

#### 5. commons/

> → 모든 단계에서 공통으로 쓰는 “도구함”

```python
# commons/logging.py
from loguru import logger
logger.add("logs/collector.log", rotation="5 MB", encoding="utf-8")
```

```python
# commons/retry.py
from tenacity import retry, stop_after_attempt, wait_fixed

@retry(stop=stop_after_attempt(3), wait=wait_fixed(2))
def safe_get(url):
    ...
```

- “모든 기자들이 공용으로 쓰는 카메라, 녹음기, 편집 툴”

→ loguru, tenacity, rich, 파서, 설정 유틸 같은 것들이 여기에 들어갑니다.

---

#### 6. factory.py

> → 모든 걸 “조립”하는 메인 엔트리

```python
# factory.py
import os
from ports.users_repo import UsersRepo
from ports.emitter import Emitter
from db_service.mongo.users_repo_impl import UsersRepoMongo
from sinks.kafka_emitter import KafkaEmitter
from sources.http_client import HttpUserIterator   # httpx로 iterator 반환
from sources.playwright_client import PlaywrightUserIterator  # 필요 시

def make_pipeline_components():
    repo: UsersRepo = UsersRepoMongo(uri=os.environ["MONGO_URI"], db="app")
    emitter: Emitter = KafkaEmitter(bootstrap=os.environ["KAFKA_BOOTSTRAP"])
    source = os.environ.get("SOURCE", "http")
    if source == "playwright":
        fetch_iter = PlaywrightUserIterator(...)
    else:
        fetch_iter = HttpUserIterator(...)
    return fetch_iter, repo, emitter
```

- “신문사 편집장이 각 부서 기자, 편집, 배포팀을 연결해서 한 호를 완성하는 단계”

---

#### 7. 실행 엔트리

```python
# collector/__main__.py
from factory import make_pipeline_components
from pipelines.users_http_pipeline import run

if __name__ == "__main__":
    fetch_iter, repo, emitter = make_pipeline_components()
    run(fetch_iter, repo, emitter, topic="users.v1")
```

---

**정리 요약**

|**계층**|**하는 일**|**비유**|**도구 예시**|
|---|---|---|---|
|**source**|외부에서 데이터 가져오기|기자가 취재|httpx, Playwright|
|**pipeline**|데이터 정리·검증·조립|편집부|pydantic, mapper, validator|
|**sink**|외부 시스템으로 내보내기|인쇄·배포부|DB, Kafka, Redis|
|**ports**|입출구 규격(인터페이스)|내부 매뉴얼|Protocol, ABC|
|**commons**|공통 도구 모음|기자의 공용 장비|loguru, tenacity, rich, parser|
|**factory**|전체 조립|편집장|환경별로 어떤 구현체를 쓸지 결정|


#### 의존성 관리(pyproject.toml) 팁

- collector 는 **런타임 의존**으로 contracts==x.y.z, db-service==x.y.z 고정
    
- db-service 는 contracts에만 의존 (역의존 금지)
    
- 버전은 고정(핀) → 배포/롤백 안정성
    

---

#### 결론

지금 제시한 구조는 **DIP를 유지하면서 소스/싱크/스토리지 교체가 쉬운** 좋은 설계입니다.

위 보완점(Emitter Port 추가, outbox 선택, rate limit/retry, observability, 테스트·셋업)을 더하면

**대량/장기 운영에 강한 프로덕션급 컬렉터**로 바로 올라갑니다.

원하시면, 위 스켈레톤을 **실행 가능한 최소 예제(패키지 스캐폴딩 + 샘플 구현체)** 로 만들어 드릴게요.