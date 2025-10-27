이미 httpx로 데이터 요청, Playwright로 렌더링 제어를 이해하셨으니,
이번엔 그 위에서 “HTML을 정제·추출하는 파서”인 BeautifulSoup(bs4) 과 lxml을 배우면
크롤링 파이프라인이 완성됩니다.

---

**학습 목표**

HTML 문서를 정확하게 파싱하고, 필요한 데이터를 CSS/XPath로 안정적으로 추출할 수 있게 된다.

(즉, “요청 → 응답 → HTML 파싱 → 데이터 추출”의 중간 핵심 단계 완성)

---

### 전체 커리큘럼 

**BeautifulSoup + lxml**

|단계|주제|핵심 목표|
|---|---|---|
|1️⃣|HTML 파싱 기초|HTML 구조 이해 및 파서의 역할 이해|
|2️⃣|BeautifulSoup 기본 문법|bs4로 HTML 탐색/추출 기본기 다지기|
|3️⃣|CSS 셀렉터와 find/find_all|요소 선택·탐색 패턴 완전히 익히기|
|4️⃣|속성(attribute)·텍스트 처리|태그 속성값, 텍스트 정제, 공백 처리|
|5️⃣|lxml 파서 & XPath|고성능 파서(lxml)과 XPath 선택 문법 학습|
|6️⃣|BeautifulSoup + lxml 결합|파서 옵션과 속도 비교, 실무적 선택 기준|
|7️⃣|실전: 데이터 추출 파이프라인|HTML → 구조적 데이터(JSON/DataFrame) 변환|
|8️⃣|실전 프로젝트|뉴스/블로그/상품/공공사이트 크롤링 실습|

---

#### 1. HTML 파싱 기초


**공부 목표:**

BeautifulSoup이나 lxml이 하는 일이 *“HTML 문자열을 트리 구조로 바꾸는 것”*임을 이해하기.


**학습 내용**

- HTML 문서 구조 (<html\>, <head\>, <body\>, <div\>, <a\>, <table\>)

- 트리 구조(DOM)의 개념

- 태그, 속성(attribute), 텍스트 노드의 구분

- HTML 파서의 역할 (텍스트 → 객체 트리)


**실습 예제**

```python
html = "<html><body><h1>Title</h1><p>내용</p></body></html>"

from bs4 import BeautifulSoup

soup = BeautifulSoup(html, "html.parser")

print(soup.h1.text)     # Title
```

---

#### 2. BeautifulSoup 기본 문법

**공부 목표:**

BeautifulSoup의 트리 탐색 문법 익히기

**학습 내용**

- BeautifulSoup(html, "html.parser")

- 태그 접근 (soup.title, soup.body.h1)

- .find() / .find_all()

- .select() (CSS 선택자)

- .text, .get_text(), .attrs, .get("href")

**실습**

```python
html = """<div class='post'><a href='/abc'>링크</a><p>본문</p></div>"""

soup = BeautifulSoup(html, "html.parser")

  
print(soup.find("a")["href"])    # 속성값

print(soup.select_one(".post p").text)  # CSS 선택자

```

---

#### 3. CSS 셀렉터 완전정복

**공부 목표:**

.select(), .select_one()으로 CSS 선택자 문법 자유자재로 사용

**학습 내용**

- 기본: tag, .class, #id

- 조합: div \> p, .post a, ul \> li:nth-child(2)

- 속성 선택: a\[href*=naver\], img\[src\$=.png\]

- 논리 조합: .active, .selected

**실습**

```python
html = """
<ul id="menu">
    <li class="item">Home</li>
    <li class="item active">Blog</li>
</ul>
"""

soup = BeautifulSoup(html, "html.parser")

print(soup.select_one("#menu li.active").text)  # Blog
```

---

#### 4. 속성값·텍스트 처리

**공부 목표:**

데이터 정제의 80%는 “공백/텍스트 다듬기”에서 발생 — 이것을 마스터하기.

**학습 내용**

- .get("속성명"), .attrs 딕셔너리

- .text vs .get_text(strip=True)

- .stripped_strings (공백제거된 텍스트 순회)

- .contents, .children으로 하위 요소 접근

- 문자열 전처리 (strip(), replace())

**실습**

```python
for s in soup.stripped_strings:
    print(s)    # 줄바꿈·공백 없는 텍스트
```

---

#### 5. lxml 파서 & XPath

**공부 목표:**

bs4보다 훨씬 빠른 lxml의 XPath 선택 기능을 익힌다.

**학습 내용**

- lxml.etree.HTML() / .fromstring()

- .xpath() 문법 (HTML 요소 선택)

- XPath 기본 문법  

    - /html/body/div\[1\]/a/text()
    - //a\[@class='link'\]
    - //table//tr\[td\[2\]>1000\]

- lxml의 HTML 파서 속도 vs BeautifulSoup 비교

**실습**

```python
from lxml import html

doc = html.fromstring("<div><a href='x'>링크</a></div>")
print(doc.xpath("//a/@href"))# ['x']
```  

---

#### 6. BeautifulSoup + lxml 결합 사용

**공부 목표:**

BeautifulSoup를 lxml 파서와 함께 쓰는 이유와 방법 익히기.

**학습 내용**

- BeautifulSoup 기본 파서: "html.parser" (파이썬 내장)

- "lxml" 파서: 더 빠르고 정확 (태그 오류 자동 보정)

- "html5lib": 가장 관대한 파서 (브라우저처럼)

- 속도·정확도 비교

- soup = BeautifulSoup(html, "lxml") 추천 패턴

**실습**

soup = BeautifulSoup(html, "lxml")

---

#### 7. 데이터 추출 파이프라인 설계

**공부 목표:**

HTML → 구조적 데이터(JSON, DataFrame)로 전환하는 실전 로직 익히기.

**학습 내용**

- find_all 반복 → dict/list 생성

- JSON 변환 (json.dumps)

- pandas DataFrame 변환

- 중첩 구조 파싱 (테이블, 리스트, 카드 레이아웃)

**실습**

```python
import pandas as pd

rows = []

for tr in soup.select("table tr")[1:]:
    tds = [td.get_text(strip=True) for td in tr.select("td")]
    rows.append(tds)

df = pd.DataFrame(rows, columns=["이름","가격","재고"])

print(df)
```

#### 8. 실전 프로젝트

|프로젝트|설명|
|---|---|
|뉴스 기사 제목/링크 수집기|뉴스 포털 HTML에서 기사 타이틀, URL 추출|
|네이버 쇼핑 랭킹 수집기|상품명, 가격, 리뷰 수 추출 → CSV 저장|
|공공기관 입찰공고 수집기|표 기반 데이터 → pandas 변환|
|wisereport HTML 테이블 파서|API 응답 대신 HTML fragment 파싱|
|블로그 포스트 요약 수집기|<div class="post"> 반복 구조 파싱|

---

#### 고급 학습 (선택)

|주제|설명|
|---|---|
|정규표현식(Regex)|.find_all(text=re.compile("키워드")) 로 텍스트 필터|
|Nested parsing (중첩 구조)|여러 레벨의 find/select 결합|
|HTML 정제 라이브러리|bleach, html5lib 활용|
|속도 최적화|lxml + XPath 병행|
|크롤링 파이프라인 통합|httpx → bs4 → pandas → DB|


**실전 연습 사이트**  

- https://quotes.toscrape.com (연습용 무료 크롤링 사이트)
- https://books.toscrape.com (상품 리스트 파싱 연습)

