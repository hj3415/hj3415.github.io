httpx를 활용해서 **API 직공(직접 API 크롤링)** 을 익히는 건,
단순 “웹 스크래핑”을 넘어서 **웹 데이터 자동화·비동기 프로그래밍·HTTP 프로토콜**을 제대로 이해하는 단계예요.

아래는 당신처럼 “실전 데이터 크롤링을 제대로 배우려는 사람”을 위한
**“httpx 기반 API 직공 크롤링 커리큘럼”** 입니다.

(단계별 → 이론 → 실습 → 실전 적용 예시 포함)

---

**전체 로드맵**

|**단계**|**주제**|**핵심 목표**|
|---|---|---|
|1️⃣|HTTP & API 기본 이해|HTTP 구조, Request/Response, Header, QueryParam 이해|
|2️⃣|requests → httpx 전환|httpx 기본 문법과 동기/비동기 패턴 학습|
|3️⃣|API 직공 원리|Network 탭 분석, API URL/params/header 파악, 응답 처리|
|4️⃣|httpx 고급 기능|비동기, 세션관리, 재시도, 예외처리, rate-limit 제어|
|5️⃣|데이터 파싱|JSON 파싱, HTML fragment 처리, pandas/DataFrame 변환|
|6️⃣|대규모 병렬 크롤링|asyncio + httpx로 수백 API 요청 병렬 처리|
|7️⃣|실무 보완|헤더 위장, 쿠키 유지, 리퍼러 제어, robots/정책 이해|
|8️⃣|실전 프로젝트|① wisereport ② 공공데이터포털 ③ 뉴스·API형 웹 서비스|
|9️⃣|확장 학습|aiofiles, async DB, asyncio 스케줄링, FastAPI 연동|

---

### 1. HTTP & API 기본 이해

**공부 목표:**

“API 직공”은 결국 **HTTP 요청을 직접 구성**하는 일이기 때문에,

HTTP 구조를 알아야 합니다.

**공부 내용**

- HTTP Request 구조: GET, POST, PUT, DELETE
    
- URL / Query Parameter / Header / Cookie / Body
    
- 응답(Response): Status Code(200, 404, 500), MIME type
    
- JSON vs HTML vs XML 응답 형태 구분
    
- API 문서 읽는 법 (Swagger, Postman, OpenAPI)

**실습 예제**

```python
import requests
r = requests.get("https://jsonplaceholder.typicode.com/posts/1")
print(r.status_code, r.headers['content-type'])
print(r.json())
```

---

### 2. requests → httpx 전환

**공부 목표:**

requests와 거의 똑같지만 더 빠르고 현대적인 httpx 문법 익히기

**공부 내용**

- httpx.get(), httpx.post()
    
- params, headers, timeout
    
- 동기(httpx.Client) vs 비동기(httpx.AsyncClient)
    
- .json(), .text, .content
    
**실습**

```python
import httpx
r = httpx.get("https://api.github.com/users/octocat")
print(r.json())
```

비동기 버전

```python
import asyncio, httpx

async def main():
    async with httpx.AsyncClient() as client:
        r = await client.get("https://api.github.com/users/octocat")
        print(r.json())

asyncio.run(main())
```

---

### 3. API 직공 원리 (핵심 파트)

**공부 목표:**

웹사이트에서 “API가 어떻게 호출되는지” 찾아내고 직접 호출하는 법 익히기.

**공부 내용**

- Chrome DevTools → Network 탭 분석
    
    - XHR / Fetch / JSON 응답 찾기
        
    - Request URL, Query params, Headers, Payload 복사하기
        
    
- “JS 렌더링”과 “API 호출”의 구분법
    
- 헤더 필수 항목: User-Agent, Referer, Accept
    
- JSON 응답 직접 호출 실습
    
**실습: wisereport / 네이버 금융**

- JS 내부에서 호출하는 /company/cF4002.aspx 찾아 직접 호출
    
- 응답 .json() 파싱
    
- 파라미터 반복 (cmp_cd=005930, 000660 등)

---

### 4. httpx 고급 기능

**공부 목표:**

httpx의 강점 — 병렬, 세션, 재시도, 커넥션 풀 등을 자유자재로 다루기

**공부 내용**

- Client 객체로 세션 재사용 (with httpx.Client())
    
- AsyncClient로 비동기 처리 (asyncio.gather)
    
- 타임아웃(timeout=10.0)과 예외(r.raise_for_status())
    
- 재시도/백오프 (직접 구현 or tenacity)
    
- Rate Limit 제어 (Semaphore)
    
**실습**

- 100개 API를 비동기로 요청 후 평균 응답 시간 비교
    
- 실패 시 3회 재시도(backoff)

---

### 5. 데이터 파싱

**공부 목표:**

API 응답(JSON 또는 HTML fragment)을 가공하는 기술 익히기

**공부 내용**

- JSON 파싱 (.json())
    
- HTML fragment 처리 (parsel.Selector, BeautifulSoup, Selectolax)
    
- pandas로 데이터프레임 변환
    
- CSV / Excel / Parquet 저장

**실습**

```python
import pandas as pd
import httpx

r = httpx.get("https://api.coincap.io/v2/assets")
data = r.json()['data']
df = pd.DataFrame(data)
print(df.head())
```

---

### 6. 대규모 병렬 크롤링 (Async 핵심)

**공부 목표:**

300개, 1000개 API를 동시에 호출할 수 있게 만들기

**공부 내용**

- asyncio.gather() / Semaphore / create_task
    
- “동시 요청 수 제한” 구현
    
- 응답 JSON 병합 → DataFrame으로 저장

**실습**

```python
import asyncio, httpx, pandas as pd

codes = ["005930","000660","035420"]

async def fetch(client, code):
    params = {"cmp_cd": code, "rpt": 1, "finGubun": "IFRSL"}
    r = await client.get("https://navercomp.wisereport.co.kr/v2/company/cF4002.aspx", params=params)
    return code, r.json()

async def main():
    async with httpx.AsyncClient(timeout=10) as client:
        data = await asyncio.gather(*(fetch(client, c) for c in codes))
    df = pd.DataFrame([{"code": c, **d} for c,d in data])
    print(df.head())

asyncio.run(main())
```

---

### 7. 실무 보완 (안티봇 회피 & 안정화)

**공부 목표:**

실제 운영 시 차단이나 오류 없이 돌릴 수 있게 만들기.

**공부 내용**

- 헤더 랜덤화 (User-Agent, Referer, Accept-Language)
    
- 요청 지연 (random.uniform)
    
- robots.txt 확인
    
- 프록시 사용 (proxies=)
    
- 에러/Timeout 처리 (try/except, r.is_error)
    

---

### 8. 실전 프로젝트

|**프로젝트**|**내용**|
|---|---|
|**wisereport 기업 300개 재무데이터 수집기**|/company/cF4002.aspx API 호출 → JSON 파싱 → CSV 저장|
|**공공데이터포털 API 수집**|API Key 발급 → 파라미터 조합 → 비동기 요청|
|**네이버 뉴스/증시 요약 크롤러**|HTML+JSON 혼합 구조 파싱|
|**날씨/환율 API 모니터링 봇**|주기적 비동기 호출 + 로깅|

---

### 9. 확장 학습 (심화)

**공부 목표:**

httpx를 실전 서비스로 통합하기

**공부 내용**

- httpx + FastAPI (API 집계 서버 만들기)
    
- httpx + sqlite / asyncpg (비동기 DB 저장)
    
- tenacity, rich, loguru로 안정성 강화
    
- asyncio task 스케줄러 (apscheduler, aiotaskq)
    
- 실제 크롤링 파이프라인 설계

---

**추천 학습 순서 (한눈에 보기)**

|**주차**|**학습 주제**|**결과물**|
|---|---|---|
|1주차|HTTP/requests 기초|JSON 응답 파싱 스크립트|
|2주차|httpx 문법/비동기 패턴|여러 URL 병렬 요청|
|3주차|Network 탭 분석 & API 직공|wisereport API 한 종목 추출|
|4주차|병렬 크롤링 & 예외 처리|300개 종목 동시 크롤러|
|5주차|데이터 가공 & 시각화|pandas로 표/차트 출력|
|6주차|안정화 & 정책 고려|header/지연/로깅/백오프 적용|
|7주차|프로젝트 완성|wisereport 전체 자동 수집기 완성|
