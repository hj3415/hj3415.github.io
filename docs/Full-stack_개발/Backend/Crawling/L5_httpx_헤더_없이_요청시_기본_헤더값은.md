
요약하자면 —

헤더를 직접 지정하지 않아도, httpx(혹은 브라우저, requests)는 
**필수 기본 헤더(Default Headers)**를 자동으로 붙여서 보냅니다.

하지만 Content-Type, Authorization 같은 중요한 헤더를
명시하지 않으면 → 서버가 요청을 이해하지 못하거나, 거부할 수 있습니다.

---

### 1. 기본적으로 포함되는 헤더

httpx는 아무 헤더도 지정하지 않아도, 다음과 같은 최소한의 헤더를 자동으로 보냅니다:


> 예시
>
> httpx.get("https://example.com")

```
GET / HTTP/1.1

Host: example.com

Accept: */*

Accept-Encoding: gzip, deflate, br

User-Agent: python-httpx/0.27.0

Connection: keep-alive
```

|   |   |   |
|---|---|---|
|헤더|설명|기본값|
|Host|요청 대상 서버 이름|URL에서 자동 추출|
|Accept|서버에 “어떤 형식의 응답을 원함”|*/* (모두 허용)|
|Accept-Encoding|압축 방식 지원|gzip, deflate, br|
|User-Agent|클라이언트 정보|python-httpx/<버전>|
|Connection|연결 유지 여부|keep-alive|

- 이건 HTTP 규약상 필수 최소 헤더입니다.

이 정도는 자동으로 추가되어야 서버가 정상적으로 응답할 수 있어요.

---

### 2. 하지만 다음 헤더들은 “자동 추가되지 않음”

|   |   |   |
|---|---|---|
|헤더|자동 설정 여부|의미|
|Authorization|❌|인증 토큰 (JWT, API 키 등)|
|Content-Type|❌ (단, json=일 때만 자동)|전송 데이터 형식|
|Accept|✅ (기본값: */*)|응답 형식 요청|
|Referer|❌|이전 페이지 정보|
|Cookie|❌|세션 유지용 쿠키|

**예를 들어 이런 상황:**

```python
import httpx

r = httpx.post("https://api.example.com/login", data={"id":"hj","pw":"1234"})
```

이때 자동으로 전송되는 헤더는

```
POST /login HTTP/1.1

Host: api.example.com

Content-Type: application/x-www-form-urlencoded

Accept: */*

User-Agent: python-httpx/0.27.0
```

→ data=를 쓰면 Content-Type: application/x-www-form-urlencoded 자동 추가

→ json=을 쓰면 Content-Type: application/json 자동 추가

→ 하지만 Authorization은 여전히 없음 ❌

---

### 3. 헤더를 생략하면 생기는 실제 문제들

|   |   |   |
|---|---|---|
|상황|생략된 헤더|결과|
|REST API 호출 시|Authorization|401 Unauthorized (인증 실패)|
|JSON 전송 시|Content-Type|서버가 Body를 못 읽어서 400 Bad Request|
|특정 API 요청 시|Accept|서버가 XML 등 예상치 못한 형식 반환|
|브라우저 흉내 낼 때|User-Agent|일부 서버가 “비정상 클라이언트”로 차단|

즉, **HTTP 요청은 헤더가 없으면 “문법상은 가능하지만 의미가 불분명”**해집니다.

서버는 헤더를 기준으로 요청을 해석하니까요.

---

### 4. 결론 요약

|   |   |
|---|---|
|항목|내용|
|기본 헤더|Host, Accept, Accept-Encoding, User-Agent, Connection|
|자동 추가되는 경우|json= 또는 data= 사용 시 Content-Type 자동|
|직접 지정해야 하는 경우|인증(Authorization), 언어(Accept-Language), 쿠키(Cookie) 등|
|생략 시 문제|인증 실패, 데이터 파싱 오류, 비정상 응답|
