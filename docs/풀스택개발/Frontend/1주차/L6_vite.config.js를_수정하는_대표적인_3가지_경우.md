
### vite.config.js 자주 수정하는 3가지 상황

  

#### 1. 백엔드(API)랑 연결할 때

- 개발 서버(:5173)와 백엔드 서버(:8000)가 다를 때
    
    → **CORS 오류**를 막기 위해 프록시(proxy) 설정이 필요합니다.
    

```javascript
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://127.0.0.1:8000', // Django나 Node 서버 주소
    },
  },
})
```

이렇게 하면 Vue에서 /api/... 로 요청해도 자동으로 http://127.0.0.1:8000/api/... 로 전달돼요.

즉, **CORS 문제 해결용** 설정입니다.

---

#### 2. 앱을 하위 폴더에 배포할 때

- 보통은 / 루트에 배포하지만 예를 들어 https://example.com/myapp/ 처럼 **서브폴더에 올릴 때**는 base 경로를 지정해야 해요.

```javascript
export default defineConfig({
  base: '/myapp/',
})
```

이렇게 안 하면 빌드된 JS, CSS 경로가 깨져서 화면이 안 나옵니다. 즉, **GitHub Pages**나 **서브폴더 배포용** 설정이에요.

---

#### 3. 경로 별칭(alias)을 추가할 때

- 기본으로 @ → src 는 이미 설정되어 있지만, 필요하면 추가 별칭을 만들 수 있습니다.

```javascript
import { fileURLToPath, URL } from 'node:url'

export default defineConfig({
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
      '~components': fileURLToPath(new URL('./src/components', import.meta.url)),
    },
  },
})
```

이렇게 하면 import Header from '~components/Header.vue' 처럼 짧게 경로를 쓸 수 있어요.

---

#### 정리 요약

|**상황**|**이유**|**추가되는 설정**|
|---|---|---|
|백엔드랑 연결할 때|CORS 방지|server.proxy|
|하위 폴더에 배포할 때|리소스 경로 깨짐 방지|base|
|import 경로를 단축하고 싶을 때|가독성 향상|resolve.alias|
