unittest에서 pytest로 넘어오실 때 바로 현업에 쓰기 쉬운 **8단계(주차) 커리큘럼**을 드릴게요. 
사용 환경(파이참, FastAPI, Beanie/MongoDB, httpx/async, 크롤링)까지 반영했습니다. 
각 단계는 “핵심 개념 → 실습 체크리스트 → 예시 코드/설정” 순서로 꾸렸어요.

---

### 0주차 — 설치 & 러너 전환

**핵심**

- pytest 설치, 테스트 발견 규칙(test_*.py, *_test.py)
    
- 파이참에서 기본 테스트 러너를 pytest로 설정

**체크리스트**

- pip install pytest pytest-cov
    
- PyCharm: Preferences → _Tools_ → _Python Integrated Tools_ → _Testing_ = **pytest**
    
- 프로젝트 루트에 pytest.ini 배치

**예시 pytest.ini**

```ini
[pytest]
addopts = -q --cov=app --cov-report=term-missing
testpaths = tests
python_files = test_*.py *_test.py
filterwarnings = ignore::DeprecationWarning

```

---

### 1주차 — pytest 기본 문법 & 실행 옵션

**핵심**

- 함수 기반 테스트, assert 인트로스펙션
    
- 선택 실행: -k, 마커: -m, 상세: -vv, 실패 즉시: -x

**체크리스트**

- 기존 unittest.TestCase 없이 함수 테스트 작성
    
- pytest -k "service and not slow" -vv로 실험
    
- 실패 메세지의 풍부한 diff 확인

**샘플**

```python
# tests/test_math.py
def add(a, b): return a + b

def test_add():
    assert add(2, 3) == 5
```

---

### 2주차 — 픽스처 & 파라미터화 (핵심)

**핵심**

- @pytest.fixture 생명주기(scope=function/session)
    
- @pytest.mark.parametrize로 입력 케이스 확장
    
- conftest.py로 공용 픽스처 공유

**체크리스트**

- 공용 conftest.py 만들기
    
- 임시 디렉터리 tmp_path 사용하기

**샘플**

```python
# tests/conftest.py
import pytest

@pytest.fixture
def sample_user():
    return {"id": 1, "name": "Alice"}

# tests/test_param.py
import pytest

@pytest.mark.parametrize("raw,expected", [(3, 9), (4, 16), (5, 25)])
def test_square(raw, expected):
    assert raw * raw == expected
```

---

### 3주차 — 모킹/패치, 캡처, 파일시스템

**핵심**

- monkeypatch로 환경/함수 패치, pytest-mock의 mocker(선택)
    
- 출력/로그 캡처: capsys, caplog
    
- 파일 작업: tmp_path

**체크리스트**

- 외부 API 키를 monkeypatch.setenv로 주입
    
- 로그 레벨 필터링해서 검증

**샘플**

```python
def get_api_key():
    import os
    return os.getenv("API_KEY")

def test_env_injection(monkeypatch):
    monkeypatch.setenv("API_KEY", "secret")
    assert get_api_key() == "secret"
```

---

### 4주차 — 비동기 테스트(FastAPI 포함)

**핵심**

- pytest-asyncio로 async def 테스트
    
- FastAPI + httpx AsyncClient + 앱 lifespan 관리

**체크리스트**

- pip install pytest-asyncio httpx asgi-lifespan
    
- 앱의 /health 같은 경량 엔드포인트 테스트

**샘플 (FastAPI)**

```python
# tests/test_api_health.py
import pytest
from httpx import AsyncClient
from asgi_lifespan import LifespanManager
from fastapi import FastAPI

app = FastAPI()
@app.get("/health")
async def health(): return {"ok": True}

@pytest.mark.asyncio
async def test_health():
    async with LifespanManager(app):
        async with AsyncClient(app=app, base_url="http://test") as ac:
            r = await ac.get("/health")
            assert r.status_code == 200
            assert r.json() == {"ok": True}
```

---

### 5주차 — DB 테스트: MongoDB / Beanie

**핵심**

- 테스트 전용 DB 생성/정리 픽스처
    
- Beanie 초기화/종료, 컬렉션 cleanup
    
- 팩토리/페이커로 데이터 준비

**체크리스트**

- pip install beanie motor faker
    
- DB 이름에 난수 붙여 충돌 방지, 테스트 후 drop
    
- 통합 테스트 마커 @pytest.mark.db

**샘플**

```python
# tests/conftest.py (일부)
import asyncio, pytest
from motor.motor_asyncio import AsyncIOMotorClient
from beanie import init_beanie
from yourapp.models import UserDoc  # Beanie Document들

@pytest.fixture(scope="session")
def event_loop():
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def motor_client():
    client = AsyncIOMotorClient("mongodb://localhost:27017")
    yield client
    client.close()

@pytest.fixture
async def beanie_db(motor_client, monkeypatch):
    db = motor_client[f"test_db_{__import__('uuid').uuid4().hex[:8]}"]
    await init_beanie(database=db, document_models=[UserDoc])
    yield db
    await db.client.drop_database(db.name)
```

---

### 6주차 — 고급 주제 & 플러그인

**핵심**

- 마커: slow, e2e, db 설계 및 -m 선택 실행
    
- 실패 기대/건너뛰기: xfail, skip
    
- 병렬 실행: pytest-xdist, 커버리지: pytest-cov
    
- HTTP 모킹: respx(httpx용) 또는 responses(requests용)
    
- 속성기반 테스트(선택): hypothesis

**체크리스트**

- pytest -m "not slow", pytest -n auto로 가속
    
- --cov=yourpkg --cov-report=term-missing로 누락 라인 확인

**예시 마커 설정 (pytest.ini)**

```ini
markers =
    slow: takes longer time
    db: needs database
    e2e: end-to-end tests
```

---

### 7주차 — 구조화, 컨벤션, CI

**핵심**

- 테스트 폴더링: tests/unit, tests/integration, tests/e2e
    
- conftest.py 계층 분리, 재사용 픽스처
    
- GitHub Actions/CI에서 pytest -n auto --cov 실행

**체크리스트**

- PR 시 느린 테스트 제외, main에서 전체 스위트
    
- 테스트 데이터 팩토리(직접 or factory_boy) 정리

**샘플 구조**

```
project/
  app/...
  tests/
    conftest.py
    unit/
      test_services.py
    integration/
      test_repos.py
      test_api_health.py
    e2e/
      test_payment_flow.py
  pytest.ini
```

---

### 8주차 — unittest → pytest 마이그레이션

**핵심 맵핑표**

- setUp/tearDown → **픽스처**
    
- @classmethod setUpClass/tearDownClass → **session/module 스코프 픽스처**
    
- self.assertEqual(a, b) → assert a == b
    
- subTest → @pytest.mark.parametrize
    
- patch() → monkeypatch 또는 pytest-mock

**체크리스트**

- 기존 unittest 테스트도 pytest로 **그대로 실행 가능**
    
- 점진 변환: 공용 초기화 로직을 픽스처로 올리고, 케이스는 파라미터화

**예시 변환**

```python
# before (unittest)
class TestAdd(unittest.TestCase):
    def test_add(self):
        self.assertEqual(add(2,3), 5)

# after (pytest)
def test_add():
    assert add(2, 3) == 5
```

---

### 필수 플러그인/도구 요약

- 실행/레포트: pytest, pytest-cov, pytest-xdist
    
- 비동기: pytest-asyncio
    
- FastAPI/HTTP: httpx, asgi-lifespan, (모킹) respx
    
- 모킹 보강(선택): pytest-mock
    
- 시간/날짜: freezegun
    
- 속성기반(선택): hypothesis
    

---

### 자주 쓰는 커맨드 치트시트

```
pytest -q                     # 조용히 빠르게
pytest -vv                    # 상세 출력
pytest -k "keyword"           # 키워드 매칭 필터
pytest -m "not slow"          # 마커 선택/제외
pytest -x                     # 첫 실패에서 중단
pytest --maxfail=2            # 2회 실패 시 중단
pytest --cov=app --cov-report=term-missing
pytest -n auto                # 멀티코어 병렬
```

---

### 다음 액션(바로 적용)

1. 위 pytest.ini 추가
    
2. 파이참 러너를 pytest로 전환
    
3. tests/ 폴더에 **가장 쉬운 API 헬스체크** 테스트부터 생성
    
4. 공용 픽스처를 conftest.py로 끌어올리기
    
5. DB 통합 테스트용 Beanie 초기화 픽스처 추가
    
