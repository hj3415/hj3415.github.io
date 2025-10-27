이제 정말 **실무 중심으로 loguru를 “제대로” 쓰는 법**을 배우려는 단계군요.

그럼 이 답변에서는 이렇게 정리해드릴게요

---

**전체 구성**

1. **loguru 핵심 개념**
2. **필수 기능 (이건 꼭 알아야 함)**
3. **실전 적용 예시 (Collector / API / Batch 기준)**
4. **실전 세팅 스니펫**

---

#### 1. loguru 핵심 개념

**기본 목표**

> loguru는 파이썬의 **logging 모듈보다 훨씬 간단하게**
>
> “보기 좋은 로그 + 자동 파일 관리 + 예외 로깅”을 제공하는 라이브러리.

설정이 필요 없습니다.

**import만 하면 바로 동작.**

```python
from loguru import logger

logger.info("서버 시작")
logger.warning("응답 지연 발생")
logger.error("DB 연결 실패")
```

출력:

```
2025-10-27 11:24:01.234 | INFO     | __main__:main:10 - 서버 시작
2025-10-27 11:24:02.001 | WARNING  | __main__:main:11 - 응답 지연 발생
2025-10-27 11:24:03.521 | ERROR    | __main__:main:12 - DB 연결 실패
```

✔ 시간 / 로그레벨 / 파일 / 함수 / 라인번호 자동 표시

✔ 컬러 출력

✔ f-string처럼 {}로 포매팅

---

#### 2. 필수적으로 알아야 할 기능 6가지

|**번호**|**기능**|**설명**|**예시**|
|---|---|---|---|
|1|logger.add()|로그를 파일에도 저장|logger.add("app.log", rotation="10 MB")|
|2|로그 레벨|로그 중요도 구분|logger.debug(), logger.info(), …|
|3|포맷 지정|출력 형식 설정|`logger.add(sys.stderr, format=”{time}|
|4|예외 자동 로깅|try/except 대신 @logger.catch|@logger.catch|
|5|필터링|로그 레벨별 분리 출력|filter=lambda record: record["level"].name == "ERROR"|
|6|멀티스레드/멀티프로세스|안전한 비동기 로깅|enqueue=True 옵션|

---

**1) 로그 파일 저장 (logger.add)**

```python
from loguru import logger

logger.add(
    "logs/app.log",          # 저장 경로
    rotation="10 MB",        # 용량 기준 회전 (또는 "1 week")
    retention="7 days",      # 7일 후 삭제
    compression="zip",       # 오래된 로그 압축
    encoding="utf-8",        # 한글 깨짐 방지
    enqueue=True             # 비동기 안전 (Thread/Async/Multiprocess)
)
```

이거 한 줄이면,

- 자동으로 파일 분리 (app.log, app.log.1, app.log.2, …)
    
- 오래된 로그는 zip 압축
    
- 비동기 코드에서도 안전하게 동작
    

---

**2) 로그 레벨 구분**

|**레벨**|**의미**|**예시**|
|---|---|---|
|DEBUG|디버그용 세부 로그|내부 상태 추적|
|INFO|일반 정보|“작업 시작”, “성공”|
|SUCCESS|성공 단계 (INFO보다 한 단계 위)|“파일 저장 완료”|
|WARNING|경고|“응답 느림”|
|ERROR|오류|“API 실패”|
|CRITICAL|치명적 오류|“DB 연결 불가”|

```python
logger.debug("Debug info: {}", data)
logger.info("Start task")
logger.success("Saved: {}", filename)
logger.warning("Slow response")
logger.error("Request failed")
logger.critical("System crash!")
```

---

**3) 포맷 커스터마이즈**

```python
logger.remove()  # 기본 콘솔 제거
logger.add(sys.stderr, format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level}</level> | {message}")
```

출력:

```
2025-10-27 11:30:00 | INFO | 작업 시작
```

---

**4. 예외 자동 로깅 (@logger.catch)**

이건 진짜 loguru의 “킬러 기능”입니다.

```python
from loguru import logger

@logger.catch
def run():
    1 / 0   # 예외 발생

run()
```

출력:

```
2025-10-27 11:31:55.712 | ERROR | Caught an exception:
Traceback (most recent call last):
  File "app.py", line 6, in run
ZeroDivisionError: division by zero
```

try/except로 감쌀 필요 없이 **모든 예외를 자동 포착 + 스택트레이스 출력 + 파일 저장**까지 가능.

---

** 5) 필터와 다중 출력 (여러 로그를 동시에)

```
logger.add("logs/info.log", level="INFO", filter=lambda r: r["level"].name == "INFO")
logger.add("logs/error.log", level="ERROR", filter=lambda r: r["level"].name == "ERROR")
```

→ INFO 로그는 info.log로

→ ERROR 로그는 error.log로 자동 분류됩니다.

---

**6)멀티프로세스 / 비동기 안전**

크롤러, FastAPI, Celery, asyncio 등에서 공통 이슈는

“여러 프로세스가 동시에 로그를 쓸 때 깨짐”.

```
logger.add("logs/app.log", enqueue=True)
```

enqueue=True를 주면 내부적으로 큐를 사용해서

**동시에 여러 스레드/프로세스가 써도 절대 로그 깨지지 않습니다.**

이건 실전에서 매우 중요합니다.

비동기든 동기든 기본 활성화 추천.

---

#### 3. 실전 적용 예시

**A. 크롤러 (Collector) 예시**

```python
from loguru import logger
import httpx, time

logger.add("logs/collector.log", rotation="10 MB", retention="7 days", encoding="utf-8", enqueue=True)

@logger.catch
def fetch(url: str):
    try:
        with httpx.Client(timeout=10) as client:
            r = client.get(url)
            r.raise_for_status()
            logger.success("✅ 수집 성공: {}", url)
            return r.text
    except Exception as e:
        logger.error("❌ 수집 실패: {} ({})", url, e)
        return None

def run():
    urls = ["https://example.com", "https://naver.com"]
    start = time.time()
    for url in urls:
        fetch(url)
    logger.info("전체 완료 ({:.2f}s)", time.time() - start)

if __name__ == "__main__":
    run()
```

collector.log 에:

```
2025-10-27 11:40:00.123 | SUCCESS  | __main__:fetch:10 - ✅ 수집 성공: https://example.com
2025-10-27 11:40:01.532 | INFO     | __main__:run:20 - 전체 완료 (1.41s)
```

---

**B. FastAPI 서비스 예시**

```python
from fastapi import FastAPI
from loguru import logger

app = FastAPI()
logger.add("logs/api.log", rotation="1 day", retention="3 days", enqueue=True)

@app.middleware("http")
async def log_requests(request, call_next):
    logger.info("요청: {} {}", request.method, request.url.path)
    response = await call_next(request)
    logger.info("응답: {} {}", response.status_code, request.url.path)
    return response
```

→ 요청·응답을 자동으로 로그로 남김.

---

**C. 배치/ETL 파이프라인 예시**

```python
from loguru import logger
import pandas as pd

logger.add("logs/batch.log", rotation="5 MB", retention="7 days")

def process_csv(path: str):
    logger.info("CSV 로딩 중: {}", path)
    df = pd.read_csv(path)
    logger.info("총 {}행 로드됨", len(df))
    # ...
    logger.success("처리 완료!")

if __name__ == "__main__":
    process_csv("data/input.csv")
```

---

#### 4. 실전 세팅 스니펫 (Best Practice)

```python
# logging_setup.py
import sys
from loguru import logger

def setup_logger(level="INFO"):
    logger.remove()
    logger.add(
        sys.stderr,                              # 콘솔 출력
        level=level,
        format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level:<8}</level> | {message}"
    )
    logger.add(
        "logs/app.log",                          # 파일 저장
        rotation="10 MB",
        retention="7 days",
        encoding="utf-8",
        enqueue=True,
        level=level,
        format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {message}"
    )
    return logger
```

사용:

```python
from logging_setup import setup_logger

logger = setup_logger()

logger.info("앱 시작")
logger.warning("응답 느림")
logger.error("DB 연결 실패")
```

---

#### 5. 정리 요약

|**목적**|**기능**|**예시**|
|---|---|---|
|콘솔 출력|기본 사용|logger.info("작업 시작")|
|파일 저장|add()|logger.add("logs/app.log", rotation="10 MB")|
|자동 예외 로깅|@logger.catch|함수 위에 붙이기|
|로그 레벨 분리|filter|INFO/ERROR 각각 파일|
|안전성 확보|enqueue=True|비동기/멀티프로세스 환경|
|유지보수 쉬운 설정|logging_setup.py|프로젝트 공통 로거|

---

### 로거설정에서 맨처음 logger.remove()를 하는 이유와 format 형식 설명

logger.remove()와 format=...은 loguru 설정에서 제일 중요한 두 부분입니다.

하나씩 아주 쉽게 풀어볼게요.

---

#### 1. logger.remove() — 기본 핸들러 제거

**기본 동작**

loguru는 import하면 **자동으로 콘솔(log sink)** 하나가 등록됩니다.

즉, from loguru import logger 하는 순간 이미 이런 설정이 되어 있습니다.

```
# 내부적으로 이미 이렇게 동작 중
logger.add(sys.stderr, format="{time} {level} {message}")
```

그래서 아무 설정 없이 아래처럼 써도 바로 콘솔에 찍히죠

```
from loguru import logger
logger.info("Hello")
```

출력:

```
2025-10-27 13:01:22.345 | INFO     | Hello
```

**문제점**

하지만 **커스텀 포맷 / 파일 저장**을 설정할 때,
이 기본 콘솔 핸들러까지 같이 출력되면 **중복 로그**가 발생합니다.

```
logger.info("시작")
# 콘솔 기본 핸들러 + 내가 추가한 핸들러 2개 → 로그가 2번 찍힘
```

**해결법**

그래서 **logger.remove()** 로 기본 핸들러를 제거하고
**내가 원하는 출력 설정만 남기기 위해** 씁니다.

```
logger.remove()  # 기본 콘솔 제거
```

이제는 **logger.add()** 로 내가 직접 지정한 콘솔, 파일 핸들러만 존재합니다.

> logger.remove() = “기본 로그 출력기를 없애고 내가 원하는 출력기만 쓰겠다.”

---

#### 2. format= — 로그 출력 형식 지정

loguru는 각 로그를 “템플릿 문자열”로 구성해서 출력합니다.

이게 바로 format= 옵션이에요.

**예시**

```
format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {message}"
```

출력:

```
2025-10-27 13:05:40 | INFO     | 서버 시작
```

**각 항목의 의미**

|**항목**|**의미**|**예시 출력**|
|---|---|---|
|{time}|로그가 찍힌 시각|2025-10-27 13:05:40|
|{level}|로그 레벨|INFO, ERROR, DEBUG 등|
|{message}|실제 로그 내용|"서버 시작", "DB 연결 실패"|
|<8|왼쪽 정렬, 8자 폭 확보|INFO____ 처럼 정렬을 깔끔하게 맞춤|
|YYYY-MM-DD HH:mm:ss|날짜 포맷|ISO 형식 (연-월-일 시:분:초)|
|<green> ... </green>|(콘솔 전용) 색상 지정|콘솔에서 초록색으로 표시됨|

**그래서 콘솔용 format:**

```
format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level:<8}</level> | {message}"
```

→ 초록색 시간 + 레벨별 색상 + 메시지 출력

출력 예시:

```
2025-10-27 13:06:10 | INFO     | 크롤러 시작
2025-10-27 13:06:11 | ERROR    | DB 연결 실패
```

**파일용 format:**

```
format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {message}"
```

→ 파일은 색 코드( 등)이 필요 없으므로 일반 텍스트로.

---

#### 3. 전체 설정을 요약하면

```python
import sys
from loguru import logger

def setup_logger(level="INFO"):
    # (1) 기본 핸들러 제거
    logger.remove()

    # (2) 콘솔 출력용 핸들러 추가 (컬러 + 보기 좋은 형식)
    logger.add(
        sys.stderr,
        level=level,
        format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level:<8}</level> | {message}"
    )

    # (3) 파일 출력용 핸들러 추가 (색 제거 + 파일 회전)
    logger.add(
        "logs/app.log",
        rotation="10 MB",
        retention="7 days",
        encoding="utf-8",
        enqueue=True,
        level=level,
        format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {message}"
    )

    return logger
```

---

#### 최종 정리

|**항목**|**의미**|
|---|---|
|logger.remove()|loguru가 기본으로 등록해둔 콘솔 출력을 제거|
|logger.add(sys.stderr, ...)|콘솔로 로그 출력 (컬러 가능)|
|logger.add("logs/app.log", ...)|로그를 파일로 저장 (회전/보존 설정)|
|format="..."|로그 메시지 형식 지정|
|<green>...</green>|콘솔용 색상 태그|
|{level:<8}|로그 레벨을 8칸 고정 폭으로 왼쪽 정렬|

---

### format: - 모듈명:함수:줄번호 형식 지정 방법

“어디서 로그가 찍혔는지 (파일명, 함수명, 줄번호)”를 보고 싶을 때

format 문자열에 몇 가지 **내장 속성**을 추가하면 됩니다

---

#### 1. loguru format 주요 필드 요약

|**필드**|**설명**|**예시 출력**|
|---|---|---|
|{time}|로그 시각|2025-10-27 13:42:11|
|{level}|로그 레벨|INFO, ERROR, …|
|{message}|실제 로그 메시지|"DB 연결 성공"|
|{file}|파일명 (확장자 포함)|collector.py|
|{name}|모듈명 (파일명에서 .py 제거)|collector|
|{function}|로그를 호출한 함수명|run|
|{line}|코드 줄 번호|42|
|{process}|프로세스 ID|12345|
|{thread}|스레드 이름|MainThread|

---

#### 2. 실전 예시 — 파일·함수·줄번호 포함

```python
from loguru import logger
import sys

logger.remove()
logger.add(
    sys.stderr,
    format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
           "<level>{level:<8}</level> | "
           "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - {message}"
)

def run_task():
    logger.info("작업 시작")

run_task()
```

출력:

```
2025-10-27 13:45:55 | INFO     | collector:run_task:12 - 작업 시작
```

즉,

- {name} → collector (파일명에서 .py 제외)
    
- {function} → run_task
    
- {line} → 12 (코드 줄 번호)
    

---

#### 3. 더 자세히 보고 싶다면 {file} 포함 버전

```python
logger.add(
    sys.stderr,
    format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
           "<level>{level:<8}</level> | "
           "{file.path}:{line} in {function}() - {message}"
)
```

출력:

```
2025-10-27 13:47:02 | INFO     | /Users/demian/collector/main.py:23 in run() - 데이터 수집 완료
```

- 파일 전체 경로 + 줄번호 + 함수명까지 표시됩니다.

---

#### 4. 파일 출력용 (색상 제거 버전)

색상 태그(<green> 등)는 콘솔 전용이기 때문에 파일에는 일반 텍스트 포맷으로 넣는 게 좋아요.

```python
logger.add(
    "logs/app.log",
    rotation="10 MB",
    retention="7 days",
    encoding="utf-8",
    enqueue=True,
    format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {file}:{line} | {function} - {message}"
)
```

출력 예:

```
2025-10-27 13:48:22 | INFO     | collector.py:28 | run_task - 데이터 수집 완료
```

---

#### 5. 실전 팁

- {name} → 모듈 단위 로그 구분에 좋음 (ex. collector / db / api)
    
- {file} → 프로젝트 내 여러 파일에서 로그가 찍힐 때 위치 파악에 유용
    
- {line} → 디버깅 시 어디서 호출됐는지 바로 확인 가능
    
- {function} → 함수 단위 로깅 시 맥락 표시용

> 대부분의 실무에서는 {name}:{function}:{line} 조합이 가장 깔끔하고 많이 쓰입니다.

---

#### 예시: Best Practice 종합 버전

```python
import sys
from loguru import logger

def setup_logger(level="INFO"):
    logger.remove()
    logger.add(
        sys.stderr,
        level=level,
        format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
               "<level>{level:<8}</level> | "
               "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - {message}"
    )
    logger.add(
        "logs/app.log",
        rotation="10 MB",
        retention="7 days",
        encoding="utf-8",
        enqueue=True,
        level=level,
        format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {name}:{function}:{line} - {message}"
    )
    return logger
```

출력 (콘솔):

```
2025-10-27 13:50:05 | INFO     | collector:fetch_data:42 - 데이터 수집 완료
```

출력 (파일):

```
2025-10-27 13:50:05 | INFO     | collector:fetch_data:42 - 데이터 수집 완료
```

---

**한 줄 요약**

> 파일명·함수·줄번호는 {name}, {function}, {line} (또는 {file.path}) 필드로 표시할 수 있습니다.
>
> 보통 실무에서는 "| {name}:{function}:{line} - {message}"
>
> 이 조합이 가장 깔끔하고 유용합니다.

---

### 실무에서 loguru 사용하는 법 

- 여러모듈에서 중구난방으로 logger를 add하면 문제가 생기지 않을까?

정확히 짚으셨습니다.

logger.add()로 추가된 핸들러(sink)는 **프로세스가 살아 있는 동안 전역적으로 유지**되며,
**명시적으로 logger.remove() 하지 않는 한 자동으로 사라지지 않습니다.**

---

#### 1. loguru의 logger는 “전역 싱글톤(singleton)”

from loguru import logger 하면 
모든 모듈이 **동일한 전역 로거 인스턴스**를 가져옵니다.

즉

```python
# a.py
from loguru import logger
logger.add("a.log")
logger.info("A에서 로그")

# b.py
from loguru import logger
logger.info("B에서 로그")  # a.log에도 같이 기록됨
```

→ loguru는 logging.getLogger(__name__)처럼 이름별 logger를 만드는 구조가 아니라
**전역 logger 하나를 모든 모듈이 공유하는 구조**예요.

---

#### 2. “언제까지 남는가?”

> 한 번 logger.add()로 추가된 sink는
> 프로그램(프로세스)이 **종료될 때까지 계속 유지**됩니다.

- 모듈 import 횟수와 관계없음
    
- 함수나 클래스가 끝나도 제거되지 않음
    
- 새로 add하면 기존 sink들과 **모두 공존**

즉, loguru는 **프로그램 전체에서 공유되는 단일 파이프라인**이에요.

---

#### 3. 그래서 생길 수 있는 문제**

**예시**

```python
# module_a.py
from loguru import logger
logger.add("a.log")

# module_b.py
from loguru import logger
logger.add("b.log")
```

이제 main.py에서 두 모듈을 임포트하면,

```python
import module_a
import module_b
logger.info("Test")
```

→ 결과적으로 로그가 **a.log와 b.log 두 곳 모두**에 기록됩니다.
(각 모듈이 전역 logger에 sink를 추가했기 때문)

시간이 지나면 이렇게 누적돼서 로그가 중복되거나 꼬일 수 있습니다.

---

#### 4. 그래서 실무에서는 “중앙 로거 설정 파일”을 둡니다

**권장 구조**

```
project/
 ├── core/
 │    └── logging_setup.py
 ├── collector/
 │    └── ...
 ├── db/
 │    └── ...
 └── main.py
```

```python
# core/logging_setup.py
from loguru import logger
import sys

def setup_logger(level="INFO"):
    logger.remove()  # 기존 모든 핸들러 제거 (중복 방지)
    logger.add(sys.stderr, level=level, format="...")
    logger.add("logs/app.log", rotation="10 MB", retention="7 days", enqueue=True)
    return logger
```

그 다음, **어디서나 이 logger를 가져다 써야 합니다.**

```python
# collector/scraper.py
from core.logging_setup import setup_logger
logger = setup_logger()

logger.info("스크래핑 시작")
```

이렇게 하면

- 모든 패키지에서 **같은 logger 인스턴스** 사용
    
- 설정 중복 없음
    
- 모듈별 logger.add() 호출로 인한 “엉킴” 방지
    

---

#### 5. 만약 외부 패키지가 logger.add()를 호출한다면?

예를 들어,

```python
# 외부 패키지 내부 코드
from loguru import logger
logger.add("external.log")
```

이러면 내 프로젝트의 전역 logger에도 해당 sink가 붙어버립니다.
즉, 내 로그가 외부 패키지의 log 파일에도 찍힐 수 있어요.

**해결책**

1. 외부 패키지 import 전에 logger.remove()로 초기화 
2. 내 프로젝트에서 새로 add()만 유지

예:

```python
from loguru import logger
logger.remove()          # 외부 패키지 설정 제거
logger.add("logs/app.log")
```

이러면 외부 패키지의 add()보다 우선 적용됩니다.

---

#### 6. 정리 요약

|**항목**|**설명**|
|---|---|
|logger의 수명|프로그램(프로세스) 종료 시까지|
|logger.add() 효과 범위|전역 (모든 모듈 공유)|
|logger.remove()|모든 기존 sink 제거|
|여러 모듈에서 add() 호출 시|로그가 중복으로 여러 sink에 전송됨|
|권장 패턴|중앙 setup_logger() 함수 한 곳에서만 add() 수행|
|외부 패키지가 add() 호출한 경우|초기화(remove())로 무효화 가능|

---

#### 실전 요약

> loguru는 “전역 단일 logger”이다.
>
> add()한 핸들러는 프로그램 종료까지 유지된다.
>
> 여러 모듈에서 add()를 호출하면 로그가 중복된다.
>
> 따라서 중앙 설정에서 logger.remove() 후 한 번만 add() 하는 게 정석이다.
>
> 외부 패키지가 add()를 한다면 import 전에 초기화로 방어한다.

---
