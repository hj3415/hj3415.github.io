이미 **httpx를 통한 API 직공 크롤링** 커리큘럼을 공부하셨다면, 
HTTP 기본 구조·비동기 프로그래밍·데이터 파싱 같은 핵심 이론은 겹칩니다.

따라서 이번엔 그 중복 부분은 빼고,
**Playwright로만 가능한 영역(렌더링·상호작용·동적 페이지 제어)** 중심의 **추가 커리큘럼**으로 구성해드릴게요.

---

**목표**

> “API가 없거나, JS 렌더링/로그인/클릭이 필요한 페이지에서 데이터를 자동으로 수집할 수 있도록
> Playwright를 실무 수준으로 익힌다.”

---

**Playwright 전용 보완 커리큘럼 (httpx 학습 이후 과정)**

|**단계**|**주제**|**핵심 목표**|
|---|---|---|
|1️⃣|Playwright 기본 구조|브라우저 자동화의 핵심 개념과 실행 흐름 익히기|
|2️⃣|DOM 제어 & 셀렉터|JS로 렌더된 요소 접근, 클릭·입력·대기 패턴 익히기|
|3️⃣|대기(wait)와 안정화|wait_for_* 시리즈, 네트워크 대기 전략|
|4️⃣|동적 데이터 수집|JS로 갱신되는 텍스트·표·차트 추출|
|5️⃣|로그인/폼/인증 자동화|로그인·2FA·세션 유지·쿠키 복원|
|6️⃣|Network 인터셉트|API 요청 가로채기 → API 직공으로 전환|
|7️⃣|headless 운영 & 성능최적화|서버에서 안정적 실행, 속도 조정|
|8️⃣|비동기/병렬 처리|asyncio + Playwright 조합으로 여러 페이지 동시 수집|
|9️⃣|실전 프로젝트|JS 렌더링 사이트(예: 네이버 리포트, 증권사 차트 등) 크롤러 제작|

---

### 1. Playwright 기본 구조

**공부 목표:**

Playwright의 “페이지를 코드로 조작한다” 개념을 익히기.

**핵심 개념**

- **브라우저 인스턴스** (browser)
    
- **컨텍스트** (탭/세션 단위)
    
- **페이지 객체** (page)
    
- **headless / headed 모드**
    
- **sync / async** 방식 차이
    
**실습 예제**

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.title())
    browser.close()
```

---

### 2. DOM 제어 & 셀렉터

**공부 목표:**

렌더링된 HTML 요소를 찾아 클릭, 입력, 추출하는 법 익히기.

**학습 내용**

- page.locator(selector) / page.query_selector
    
- CSS / XPath 셀렉터 선택
    
- .click(), .fill(), .text_content(), .get_attribute()
    
- nth(), has_text(), has_selector() 응용
    
**실습**

```python
page.goto("https://quotes.toscrape.com/")
for quote in page.locator(".quote").all():
    print(quote.locator(".text").text_content())
```

---

### 3. 대기(wait)와 안정화 전략

**공부 목표:**

비동기 렌더링에서 **“요소가 나타날 때까지 기다리는 법”** 익히기.

**학습 내용**

- page.wait_for_selector()
    
- page.wait_for_load_state("networkidle")
    
- page.wait_for_timeout(ms)
    
- 이벤트 기반 대기 (expect(page).to_have_selector)

**실습**

```python
page.goto("https://wisereport.co.kr/v2/company/...")
page.wait_for_selector("#compBody")  # JS 렌더 후 존재
html = page.content()
```

---

### 4. 동적 데이터 추출

**공부 목표:**

렌더링 이후 DOM에서 JS 생성 데이터를 가져오기

**학습 내용**

- .content() 로 전체 HTML 추출
    
- .evaluate(js_expr) 로 JS 변수 직접 접근
    
- .screenshot() / .pdf() 출력

**실습**

```python
table_html = page.locator("#financeTable").inner_html()
# 또는 JS 변수 접근
data = page.evaluate("window.__FINANCE_DATA__")
```

---

### 5. 로그인 & 세션 관리

**공부 목표:**

쿠키 기반 인증, 로그인 폼 자동화, 세션 재사용

**학습 내용**

- .fill() / .press("Enter")
    
- .context.storage_state(path="state.json") 저장
    
- .context = browser.new_context(storage_state="state.json") 복원
    
- 다중 로그인 처리
    
**실습**

```python
page.goto("https://example.com/login")
page.fill("#id", "myid")
page.fill("#pw", "mypass")
page.click("#submit")
page.context.storage_state(path="auth_state.json")
```

---

### 6. Network 인터셉트 (★API 직공과의 연결고리)

**공부 목표:**

Playwright가 호출하는 네트워크 요청을 **가로채서 분석/복제**하는 기술.

**학습 내용**

- page.on("request"), page.on("response")
    
- request.url, request.post_data, response.json()
    
- 특정 요청 필터링 (if "ajax" in url)
    
- API 엔드포인트를 추출하여 httpx로 재활용
    
**실습**

```python
def on_response(response):
    if "cF4002.aspx" in response.url:
        print("API 발견:", response.url)
        print(response.json())

page.on("response", on_response)
page.goto("https://navercomp.wisereport.co.kr/v2/company/c1040001.aspx?cmp_cd=005930")
```

- 이렇게 찾은 API를 이후엔 **Playwright 없이 httpx로 직접 요청** → “API 직공 전환”.

---

### 7. Headless 운영 & 성능최적화

**공부 목표:**

Playwright를 **서버 환경**이나 **Docker**에서 안정적으로 돌리기.

**학습 내용**

- headless=True
    
- slow_mo 로 디버깅 속도 제어
    
- 네트워크 대역폭 절약 (route().abort("image"))
    
- 크롤링 시 브라우저 누수 방지 (context close)

---

### 8. 비동기 / 병렬 크롤링

**공부 목표:**

여러 페이지를 동시에 열어 빠르게 수집.

**학습 내용**

- asyncio.create_task()
    
- 여러 브라우저 컨텍스트에서 병렬 실행
    
- connection reuse, 세션 분리
    
**실습**

```python
import asyncio
from playwright.async_api import async_playwright

async def fetch(code):
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        await page.goto(f"https://navercomp.wisereport.co.kr/v2/company/c1040001.aspx?cmp_cd={code}")
        await page.wait_for_selector("#compBody")
        html = await page.content()
        await browser.close()
        return code, html

asyncio.run(asyncio.gather(*(fetch(c) for c in ["005930","000660","035420"])))
```

---

### 9. 실전 프로젝트 예시

|**프로젝트**|**설명**|
|---|---|
|**Wisereport 자동 스크린샷 크롤러**|종목별 페이지 렌더 → 차트/표 캡처 → 이미지 저장|
|**네이버 증권 로그인 후 관심종목 수집기**|로그인 세션 저장 + 포트폴리오 데이터 추출|
|**JS 차트 데이터 추출기**|Network 이벤트 가로채서 Chart.js 데이터 JSON 추출|
|**Vue/React 기반 동적 테이블 수집기**|HTML 로딩 후 .evaluate()로 렌더 데이터 파싱|

---

### 실무에 꼭 필요한 추가 토픽

|**주제**|**설명**|
|---|---|
|**Stealth 모드**|안티봇 방지 (stealth.min.js, UA·navigator 수정)|
|**Proxy / User-Agent 회전**|context.new_page()마다 다르게|
|**스크롤 시 무한 로딩 처리**|page.evaluate("window.scrollTo(0, document.body.scrollHeight)") 반복|
|**예외 처리**|try/except TimeoutError, retry, wait_for_selector timeout 설정|
|**로깅/모니터링**|playwright.debug() / console.log() 캡처|

---

### 학습 순서 요약 (Playwright 전용 추가 커리큘럼)

|**주차**|**주제**|**결과물**|
|---|---|---|
|1주차|기본 구조 + 셀렉터 제어|페이지 이동 및 텍스트 추출|
|2주차|Wait & 렌더링 안정화|JS 렌더링 완료 후 HTML 추출|
|3주차|로그인 & 세션 유지|state.json을 통한 자동 로그인|
|4주차|Network 인터셉트|Wisereport API 요청 실시간 탐지|
|5주차|비동기 병렬 실행|여러 종목 페이지 동시 크롤링|
|6주차|Headless + 최적화|서버/배치용 크롤러 완성|
|7주차|실전 프로젝트|“Wisereport 렌더링 크롤러” 완성|

---

### 결론 요약

> **Playwright 학습은 httpx 이후 단계로, “API가 없거나 렌더링이 필요한” 데이터를 다루는 기술**입니다.
>
> **httpx** → API 직공 (빠르고 효율적인 데이터 수집)
>
> **Playwright** → JS 렌더링, 로그인, 상호작용 자동화
>
> 즉, Playwright는 “브라우저 환경을 코드로 제어하는 httpx의 확장판”이라고 보면 됩니다.
