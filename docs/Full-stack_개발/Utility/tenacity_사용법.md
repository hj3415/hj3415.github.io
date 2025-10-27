**tenacity**는 파이썬에서 **“자동 재시도(retry)” 로직을 간단하게 구현할 수 있게 해주는 라이브러리**입니다.

예를 들어 네트워크 요청, DB 연결, 외부 API 호출처럼 가끔 실패할 수 있는 작업에 유용하죠.

---

#### 1. 기본 개념

tenacity는 “한 번 실패했다고 바로 중단하지 말고, 정해진 규칙에 따라 다시 시도하자”는 철학을 코드로 구현한 겁니다.

예를 들어 이런 식이죠:

- “요청이 실패하면 3초 간격으로 최대 5번 다시 시도해라.”
    
- “HTTP 500 오류면 재시도하되, 404면 중단해라.”
    
- “매번 딜레이를 1초, 2초, 4초… 식으로 늘려가라(지수 백오프).”
    

---

#### 2. 설치

```
pip install tenacity
```

---

#### 3. 간단한 예시

```python
from tenacity import retry, stop_after_attempt, wait_fixed
import random

@retry(stop=stop_after_attempt(5), wait=wait_fixed(2))
def unstable_task():
    x = random.random()
    print(f"Trying... x={x}")
    if x < 0.7:
        raise Exception("Temporary failure")
    print("Success!")

unstable_task()
```

**실행 결과 예시**

```
Trying... x=0.31  ❌ 실패 → 2초 대기
Trying... x=0.42  ❌ 실패 → 2초 대기
Trying... x=0.79  ✅ 성공!
```

**설명**

- @retry(...) 데코레이터가 자동으로 재시도 로직을 추가합니다.
    
- stop_after_attempt(5) → 최대 5번 시도
    
- wait_fixed(2) → 각 시도 사이에 2초씩 대기
    

---

#### 4. 주요 옵션 정리

|**카테고리**|**옵션**|**설명**|
|---|---|---|
|**종료 조건**|stop_after_attempt(n)|n번 시도 후 중단|
||stop_after_delay(t)|t초가 지나면 중단|
|**대기 전략**|wait_fixed(t)|매번 t초씩 기다림|
||wait_random(min, max)|min~max 사이에서 랜덤 대기|
||wait_exponential(multiplier=1, max=60)|1초 → 2초 → 4초 → 8초… 지수적으로 증가|
|**예외 조건**|retry_if_exception_type(ErrorType)|특정 예외만 재시도|
||retry_if_result(lambda x: x is None)|반환값에 따라 재시도|
|**로깅/콜백**|before, after, before_sleep|시도 전후에 커스텀 동작 실행 가능|

---

#### 5. 예외 필터링 예시

```python
from tenacity import retry, retry_if_exception_type, stop_after_attempt

@retry(
    retry=retry_if_exception_type(ConnectionError),
    stop=stop_after_attempt(3)
)
def fetch_data():
    print("Fetching...")
    raise ConnectionError("Network problem")

fetch_data()
```

→ ConnectionError이면 재시도하지만, 다른 에러는 바로 멈춥니다.

---

#### 6. 실제 예시: httpx 연동

```python
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

@retry(
    retry=retry_if_exception_type(httpx.RequestError),
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=1, max=10)
)
async def fetch_with_retry(url: str):
    async with httpx.AsyncClient() as client:
        r = await client.get(url)
        r.raise_for_status()
        return r.json()
```

- **네트워크 불안정한 환경에서 매우 유용**

(예: 외부 API 크롤러, 금융 시세 수집기 등)

---

#### 7. 핵심 요약

|**항목**|**설명**|
|---|---|
|라이브러리|tenacity|
|핵심 기능|함수 실패 시 자동 재시도|
|주요 요소|stop_*, wait_*, retry_* 조합|
|사용 형태|@retry(...) 데코레이터|
|활용 예|외부 API 호출, 네트워크 요청, DB 연결 등|
