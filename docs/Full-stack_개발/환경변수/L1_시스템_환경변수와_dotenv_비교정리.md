
### 전체 개요

환경변수(Environment Variable)는

**“코드 바깥에서 프로그램의 설정값을 주입하는 방법”**입니다.

→ 코드 수정 없이 환경만 바꿔서 동작을 제어할 수 있게 해주는 장치입니다.

---

#### 1. 일반 환경변수 vs .env + load_dotenv()의 차이

|**구분**|**일반 환경변수**|.env **+** load_dotenv()|
|---|---|---|
|**출처**|OS(운영체제)에 직접 설정됨 (export, set)|.env 파일에 저장됨|
|**형태**|영구적 or 세션 단위 환경|코드 실행 시점에 .env를 읽어서 임시로 os.environ에 등록|
|**로드 방식**|OS가 자동 관리|코드에서 load_dotenv()를 명시적으로 호출해야 함|
|**공유 범위**|모든 프로세스가 접근 가능|현재 Python 프로세스 내에서만 유효|
|**용도**|운영·배포 환경 (CI/CD, Docker, 서버)|개발환경 (로컬, PyCharm, VSCode)|
|**타입**|항상 문자열(str)|문자열(str)|
|**예시**|export MONGO_URI="mongodb://localhost:27017"|.env 안에 MONGO_URI="mongodb://localhost:27017"|

- load_dotenv()는 .env 파일의 내용을 **메모리로 불러와서**
    
    → 내부적으로 os.environ에 등록시켜주는 “헬퍼”일 뿐입니다.
    
- 따라서 실제 접근은 **항상** **os.getenv()** 로 동일하게 합니다.

---

#### 2. 환경변수를 읽는 표준적인 방법

(1) 단순 프로젝트 (스크립트, CLI)

```python
import os
from dotenv import load_dotenv

load_dotenv()  # .env 파일 읽어서 os.environ에 추가

MONGO_URI = os.getenv("MONGO_URI", "mongodb://127.0.0.1:27017")
DEBUG = os.getenv("DEBUG", "false").lower() == "true"
```

- .env가 있으면 거기서 읽고, 없으면 시스템 환경변수에서 읽음 (우선순위: 시스템 > .env).

(2) 대규모 프로젝트 (서비스, 웹앱)

대부분은 “설정 클래스를” 만들어 환경변수를 한곳에 모읍니다.

**예시 1: 직접 유틸 함수로**

```python
def _env_bool(key: str, default: bool) -> bool:
    val = os.getenv(key)
    return val and val.lower() in ("1", "true", "yes", "on")

def _env_int(key: str, default: int) -> int:
    try:
        return int(os.getenv(key, default))
    except ValueError:
        return default
```

```python
MONGO_URI = os.getenv("MONGO_URI", "mongodb://localhost:27017")
HEADLESS = _env_bool("SCRAPER_HEADLESS", True)
CHUNK = _env_int("SCRAPER_SINK_CHUNK", 1000)
```

**예시 2: Pydantic으로**

```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    MONGO_URI: str = "mongodb://localhost:27017"
    SCRAPER_HEADLESS: bool = True
    SCRAPER_SINK_CHUNK: int = 1000

    class Config:
        env_file = ".env"

settings = Settings()
```

→ .env와 시스템 환경변수를 모두 읽고 자동으로 타입 캐스팅 ("true" → True, "1000" → 1000)까지 해줍니다.

(가장 실용적이고 안전한 방법)

---

#### 3. 프로그램에서 환경변수를 다루는 규칙 (Best Practice)

**① 코드에 “직접 하드코딩하지 말 것”**

```python
# ❌ Bad
DB_URL = "mongodb://prod-server:27017"
```

→ 코드 변경 없이 환경만 바꿔서 설정을 제어해야 합니다.

**② os.getenv()를 직접 여러 곳에서 쓰지 말고 “중앙 설정 모듈”로 모을 것**

```python
# config.py
from dotenv import load_dotenv
from pydantic import BaseSettings

load_dotenv()

class Settings(BaseSettings):
    MONGO_URI: str
    SCRAPER_HEADLESS: bool = True
    SCRAPER_SINK_CHUNK: int = 1000

    class Config:
        env_file = ".env"

settings = Settings()
```

그 다음, 다른 모듈에서:

```python
from config import settings

client = AsyncMongoClient(settings.MONGO_URI)
```

→ 이렇게 하면 **한곳에서만 환경을 관리**하게 됩니다.

**③ .env는 개발용으로만, 배포 시에는 시스템 환경변수로 주입**

- 개발: .env → load_dotenv()
    
- Docker: env_file: .env
    
- 배포(CI/CD):

```python
export MONGO_URI=$PROD_MONGO_URI
python -m scraper2_hj3415 ingest-one 005930
```

**④ 환경변수는 항상 문자열로 들어오므로, 변환은 명시적으로 처리**

|**타입**|.env **예시**|**변환 함수**|
|---|---|---|
|bool|DEBUG=true|_env_bool("DEBUG", False)|
|int|CHUNK=1000|_env_int("CHUNK", 1000)|
|float|THRESHOLD=0.7|float(os.getenv("THRESHOLD", 0.7))|
|list|ALLOWED_HOSTS=a.com,b.com|.split(",") 등 직접 처리|

---

#### 4. 정리 — 환경변수 관리의 3단계 요약표

|**단계**|**설명**|**구현 방법**|
|---|---|---|
|1️⃣ 정의|설정 키를 .env 또는 배포 환경에 등록|.env 파일 or export|
|2️⃣ 로드|프로그램 시작 시 로드|load_dotenv() 또는 자동 로드|
|3️⃣ 파싱|문자열을 올바른 타입으로 변환|_env_bool, _env_int, Pydantic 등|

**예시: scraper2_hj3415 구조에서의 이상적 패턴**

```python
# scraper2_hj3415/di.py
from dotenv import load_dotenv
import os

load_dotenv()

def _env_bool(key: str, default: bool) -> bool:
    val = os.getenv(key)
    return val and val.lower() in ("1", "true", "yes", "on")

def _env_int(key: str, default: int) -> int:
    val = os.getenv(key)
    try:
        return int(val)
    except (TypeError, ValueError):
        return default

def get_settings():
    return {
        "mongo_uri": os.getenv("MONGO_URI", "mongodb://127.0.0.1:27017"),
        "mongo_db": os.getenv("MONGO_DB", "nfs_db"),
        "headless": _env_bool("SCRAPER_HEADLESS", True),
        "sink_chunk": _env_int("SCRAPER_SINK_CHUNK", 1000),
    }

settings = get_settings()
```

이제 다른 모듈에서:

```python
from scraper2_hj3415.di import settings
browser = await PlaywrightSession.create(headless=settings["headless"])
```

---

#### 결론 요약

|**항목**|**핵심 요약**|
|---|---|
|**환경변수란**|코드 밖에서 프로그램 설정을 제어하기 위한 값|
|**load_dotenv()**|.env 파일을 읽어 os.environ에 문자열로 추가|
|**자동 변환?**|❌ 없음. 항상 문자열로 로드됨|
|**올바른 처리법**|_env_bool, _env_int, 또는 Pydantic.BaseSettings로 타입 변환|
|**프로그램 구조**|중앙 config.py 또는 di.py에서 일괄 관리|
|**운영환경 차이**|개발은 .env, 배포는 시스템 환경변수|

