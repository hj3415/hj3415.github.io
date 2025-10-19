
> **“CORS 방지 설정은 원칙적으로 백엔드에서 하는 게 정답”**이에요. 하지만 **“개발 중(local 환경)”에는 프론트엔드(Vite 등)에서 임시로 proxy 설정을 써도 괜찮습니다.**

즉,

- **운영 서버(배포 후)** → 백엔드에서 CORS 설정 필수   
- **로컬 개발 환경** → 프록시(proxy)로 임시 회피 가능

---

#### CORS가 뭐였는지 다시 정리

**CORS(Cross-Origin Resource Sharing)** = “서로 다른 출처(origin)” 간의 요청을 **브라우저가 차단**하는 보안 정책이에요.

예를 들어

|**프론트엔드**|**백엔드**|**결과**|
|---|---|---|
|http://localhost:5173|http://localhost:8000|❌ 차단됨 (포트 다름 = 다른 origin)|

→ 브라우저는 이렇게 말함:

> “둘의 origin이 다르니까, 백엔드가 허락한 요청만 보낼 수 있어!”

---

**그래서 정식 방법은 백엔드에서 허용하는 것**

백엔드에서 아래처럼 설정해줘야 해요.

**Django 예시**

패키지 설치

```shell
pip install django-cors-headers
```

```python
# settings.py

INSTALLED_APPS = [
    # ...
    "corsheaders",   # ★ 반드시 추가
    # ...
]

MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",  # ★ 가능한 위쪽(보통 SecurityMiddleware 바로 아래)
    # ...
]

# 프론트 개발 서버(5173) 허용
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",
    "http://127.0.0.1:5173",
]

# 쿠키/세션을 써야 한다면(선택)
# CORS_ALLOW_CREDENTIALS = True
```

→ 이렇게 하면 브라우저가 “아, 이 서버는 localhost:5173에서 오는 요청을 허락했구나” 하고 CORS 에러를 멈춥니다

즉, **실제 해결 주체는 백엔드입니다.**

---

**하지만 개발 중에는 프론트에서 “우회” 가능**

Vite(또는 Webpack dev server)는 개발할 때만 쓸 수 있는 **proxy(프록시) 기능**을 제공합니다.

```javascript
// vite.config.js
export default defineConfig({
  server: {
    // 프록시: 프론트가 /api 로 요청하면 Vite가 백엔드(8000)로 전달
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:8000',
        changeOrigin: true,
        secure: false,
        // 필요시 프리픽스 제거
        // rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
})
```

이걸 쓰면 구조가 이렇게 바뀝니다

```
브라우저 (http://localhost:5173)
  ↓
Vite dev server (같은 origin)
  ↓
백엔드 (http://localhost:8000)
```

즉,

브라우저 입장에서는 **같은 origin**(5173 → 5173) 으로 요청을 보낸 것처럼 보이고, 
Vite가 대신 8000번 서버로 요청을 전달해줘요.

→ 결과: **CORS 에러가 사라짐 (하지만 실제 해결은 아님)**

---

**정리: “진짜 해결” vs “임시 우회”**

|**환경**|**방법**|**설명**|
|---|---|---|
|**개발용 (로컬)**|Vite server.proxy|빠르고 편한 임시 우회|
|**운영 배포 환경**|백엔드 CORS 설정|근본적 해결 (필수)|
