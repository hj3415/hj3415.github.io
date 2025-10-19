이제 CORS랑 CSRF의 차이를 확실히 이해해두면 웹 보안의 큰 그림이 완성돼요.

두 개 이름이 비슷해서 헷갈리지만, **완전히 다른 개념**이에요.

---

**한 줄 요약부터**

|**구분**|**CORS**|**CSRF**|
|---|---|---|
|풀네임|Cross-Origin Resource Sharing|Cross-Site Request Forgery|
|초점|**“다른 도메인에서 접근해도 되나?”**|**“내 세션을 도용하려는 공격”**|
|주체|**브라우저 정책**|**공격자가 위조 요청을 보내는 문제**|
|해결 위치|백엔드(서버 응답 헤더)|백엔드(토큰 검증 등)|
|언제 문제됨|프론트가 다른 오리진(API 서버)에 요청할 때|로그인/세션 기반 사이트에서 POST 요청할 때|

---

#### CORS는 “요청할 수 있나?”의 문제

CORS는 “이 **사이트(origin)** 가 이 **서버(origin)** 에 요청을 보내도 되냐?”를 **브라우저가 판단해서 막거나 허락하는 기능**이에요.

즉,

> “보내는 게 아니라 **브라우저가 받을 수 있느냐**”의 문제.

예시 

http://localhost:5173 (Vue 앱)

→ http://localhost:8000 (Django API) 로 요청하면,

브라우저가 오리진이 다르니까 기본적으로 막음 ❌

서버가 이렇게 응답해야 허용됨 ✅

```
Access-Control-Allow-Origin: http://localhost:5173
```

CORS는 단순히 **“다른 오리진 간 통신 허용 여부”**를 정하는 보안장치입니다.

즉, **요청은 막지 않지만 응답을 차단**하는 거예요.

---

#### CSRF는 “내 세션을 훔치려는 공격”의 문제

CSRF는 **“Cross-Site Request Forgery (사이트 간 요청 위조)”**

즉, 사용자의 **인증 세션을 도용해 몰래 요청을 보내는 공격**이에요.

**예시로 보면**

1. 사용자가 bank.com에 로그인했어요. 

    → 브라우저에 sessionid=123abc 쿠키가 저장됨. 
2. 공격자가 evil.com이라는 사이트를 만들어서 그 안에 이런 코드를 숨겨놓아요

```html
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker_account">
  <input type="hidden" name="amount" value="1000000">
  <button type="submit">Click me!</button>
</form>
```

3. 사용자가 로그인 상태로 evil.com에 접속해서 버튼을 클릭하면

→ 브라우저는 자동으로 bank.com 쿠키(sessionid=123abc)를 같이 보냄

→ 즉, **사용자 대신 송금 요청이 들어감!**

---

**CSRF의 핵심: “세션 자동 전송이 문제”**

브라우저는 동일 도메인의 쿠키를 자동으로 첨부하니까 공격자가 위조 요청을 만들어도 “진짜 로그인된 사용자”처럼 보입니다.

---

**그래서 CSRF 방어는 이렇게 함**

서버(Django 등)가 요청마다 “CSRF 토큰”이라는 일회용 키를 요구해요.

1. Django는 템플릿 안에 자동으로 CSRF 토큰을 삽입합니다:

```html+django
<form method="POST">
  {% csrf_token %}
  ...
</form>
```

2. 프론트(Vue 등)에서 직접 POST 요청을 보낼 때도,

CSRF 토큰을 헤더에 같이 포함시켜야 해요.

```javascript
axios.post('/api/data/', data, {
  headers: { 'X-CSRFToken': getCookie('csrftoken') }
})
```

3. 서버는 이 토큰이 맞는지 검사하고, 맞지 않으면 요청을 거부합니다.

---

#### CORS vs CSRF 정리 비교표

|**항목**|**CORS**|**CSRF**|
|---|---|---|
|뜻|Cross-Origin Resource Sharing|Cross-Site Request Forgery|
|성격|브라우저 보안 정책|공격 기법|
|언제 발생|다른 오리진 요청 시|세션 쿠키 자동 첨부 시|
|누가 막음|브라우저|서버 (Django 등)|
|해결 방법|서버에서 Access-Control-Allow-Origin 헤더 설정|CSRF 토큰 검증 ({% csrf_token %} or X-CSRFToken)|
|프론트에서 필요?|개발 시 프록시로 우회 가능|JS 요청 시 토큰을 헤더로 보내야 함|

**한 줄 요약**

- **CORS**: “이 도메인끼리 통신해도 돼?” (접근 허용 문제) -> 브라우저가 막음
    
- **CSRF**: “로그인된 사용자의 세션을 도용하려고 하지?” (요청 위조 문제) -> 백엔드가 검증해서 막음
    
---

### **Vue(axios) + Django** 조합에서 **CSRF 토큰을 자동으로 포함**해 안전하게 POST/PUT/DELETE 요청하는 “실전 최소 세트”

**전체 흐름 (요약)**

1. **Django**가 브라우저에 csrftoken 쿠키를 내려줌 
2. **axios**가 그 쿠키를 자동으로 읽어 **X-CSRFToken** **헤더**로 붙여서 전송
3. **Django CSRF 미들웨어**가 헤더를 검증해 통과
    

#### 1. Django 설정

**(1) 기본 미들웨어 확인**

```python
# settings.py
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",  # ★ 반드시 포함
    ...
]
```

**(2) 개발 환경에서 프론트 오리진 허용 (다른 포트/도메인일 때)**

```python
ALLOWED_HOSTS = ["localhost", "127.0.0.1"]

# 프론트 개발 서버를 신뢰 대상으로
CSRF_TRUSTED_ORIGINS = [
    "http://localhost:5173",
    "http://127.0.0.1:5173",
]
```

> 배포 시에는 https://frontend.example.com 같은 **실제 프론트 도메인**을 넣어주세요.

**(3) CSRF 쿠키를 내려줄 엔드포인트(1회 호출용)**

SPA는 장고 템플릿을 안 쓰니, **최초에 쿠키를 심어줄 뷰**가 하나 있으면 편합니다.

```python
# views.py
from django.views.decorators.csrf import ensure_csrf_cookie
from django.http import HttpResponse

@ensure_csrf_cookie
def csrf(request):
    return HttpResponse(status=204)  # 내용 없는 응답이지만 csrftoken 쿠키를 심음
```

```python
# urls.py
from django.urls import path
from .views import csrf

urlpatterns = [
    path("api/csrf/", csrf),  # GET /api/csrf/ 호출 시 csrftoken 쿠키 세팅
]
```

> 이 엔드포인트를 프론트에서 **앱 시작 시 한 번** 호출하면 브라우저에 csrftoken 쿠키가 준비됩니다.

**(4) (선택) 보안 옵션들 (배포 전환 시)**

```python
# 기본은 JS 접근 가능 쿠키( HttpOnly=False )라 axios가 읽을 수 있습니다.
CSRF_COOKIE_SECURE = True          # HTTPS에서만 전송
SESSION_COOKIE_SECURE = True
# 서로 다른 도메인에서 쿠키 전송 필요하면:
# CSRF_COOKIE_SAMESITE = "None"    # Secure=True 반드시 함께!
```

---

#### 2. 프런트(Vue) – axios 설정

##### A안) axios의 내장 XSRF 기능 활용 (가장 간단, 추천)**

```javascript
// src/api/client.js
import axios from 'axios'

export const api = axios.create({
  baseURL: '/api',                  // Vite proxy를 쓰는 경우
  withCredentials: true,            // 다른 오리진이면 쿠키 전송을 허용
  xsrfCookieName: 'csrftoken',      // Django 기본 쿠키 이름
  xsrfHeaderName: 'X-CSRFToken',    // Django가 읽는 헤더 이름
})
```

> 이렇게만 해두면 axios가 자동으로 csrftoken 쿠키를 읽어 요청마다 X-CSRFToken 헤더를 **자동**으로 붙여줍니다.

**앱 시작 시 CSRF 쿠키 한번 받아오기**

```javascript
// e.g. src/main.js
import { api } from '@/api/client'

// 앱 부팅 시 한번 호출 (csrftoken 쿠키 세팅)
api.get('/csrf/').catch(() => {})   // 204라 에러 아님, 그냥 준비용
```

**사용 예시**

```javascript
// POST 예시
await api.post('/items/', { title: 'hello' })  // CSRF 자동 포함
```

##### B안) 직접 쿠키 읽어 수동으로 헤더 설정 (참고용)

```javascript
function getCookie(name) {
  return document.cookie
    .split('; ')
    .find(row => row.startsWith(name + '='))
    ?.split('=')[1]
}

import axios from 'axios'
export const api = axios.create({ baseURL: '/api', withCredentials: true })

api.interceptors.request.use((config) => {
  const token = getCookie('csrftoken')
  if (token) config.headers['X-CSRFToken'] = token
  return config
})
```

---

#### 3. Vite 개발 프록시(선택 – CORS 편의)

개발 중 프론트: :5173, 장고: :8000이면 CORS 이슈를 줄이려고 프록시를 쓰면 편합니다.

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:8000',
        changeOrigin: true,
        secure: false,
      },
    },
  },
})
```

---

#### 4. 체크리스트 (자주 막히는 포인트)

- **CsrfViewMiddleware** 가 MIDDLEWARE에 있는가 
- 프론트 도메인이 **CSRF_TRUSTED_ORIGINS** 에 있는가
- 첫 페이지에서 **/api/csrf/** 를 한 번 호출했는가 (쿠키 세팅)
- axios가 **withCredentials: true** 로 쿠키를 보내는가 (오리진 다를 때)
- 헤더 이름이 **X-CSRFToken** 인가 (Django 기본)
- 배포 환경이면 **HTTPS + Secure 쿠키 + SameSite 설정**을 맞췄는가

---

**한 줄 요약**

> Django가 csrftoken 쿠키를 심고, axios가 그것을 자동으로 X-CSRFToken 헤더로 전송하게 설정하면, Vue + Django 환경에서 **추가 작업 없이 안전한 CSRF 검증**이 동작합니다.