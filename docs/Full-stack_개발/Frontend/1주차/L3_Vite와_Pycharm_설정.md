**Vite 기반 Vue 3 프로젝트를 만들고, PyCharm Professional(프로)에서 “편하게 개발 + 린트/포맷 + 디버깅 + Django와 연동”** 까지 
한 번에 세팅하는 방법을 단계별로 정리해드릴게요.

(전제: PyCharm **Professional**에는 WebStorm 기능이 들어있어 Vue/JavaScript/ESLint/Prettier/Node 실행이 가능합니다.)

---

#### 0. 준비물 체크

- **Node.js 18+** (설치 후 node -v, npm -v 확인)
    
- **PyCharm Professional** (설정에 JavaScript/Vue/Node 설정이 보여야 함)
    
- (선택) Django 백엔드와 함께 돌릴 계획이면 Python venv 준비
    

---

#### 1. Vite + Vue 프로젝트 생성

터미널(=PyCharm 내 Terminal 탭도 OK):

```
# 1) 새 프로젝트 폴더 만들기(또는 기존 repo의 frontend 디렉토리로)
mkdir frontend && cd frontend

# 2) Vite 기반 Vue 프로젝트 생성
npm create vue@latest

# 3) 프롬프트 권장 선택
# - TypeScript: 초보는 No(권장)
# - Vue Router: Yes (SPA 필수)
# - Pinia: Yes (상태관리)
# - ESLint + Prettier: Yes (강력 추천)

# 4) 설치 & 기동
npm install
npm run dev
```

TypeScript로 이미 만든 프로젝트를 “JS로 되돌리려면” 새로 만드는 게 훨씬 낫습니다.

다른 것들(Vue Router, Pinia, ESLint, Prettier)은 추후에 추가하는 것도 가능합니다.
 
브라우저에서 http://localhost:5173 가 보이면 성공.

---

#### 2. PyCharm Professional 설정

**2-1. Node 인터프리터 연결**

설정 ⟶ 언어 및 프레임워크 ⟶ Node.js

- **Node interpreter**: (System Node) 선택 
- **Package manager**: npm (또는 pnpm/yarn 사용 시 변경)

**2-2. Vue/TS/ESLint/Prettier 인식**

- 설정 ⟶ Plugins 에서 **Vue.js**, **JavaScript and TypeScript** 활성화(보통 기본 포함)
- 설정 ⟶ 언어 및 프레임워크 ⟶ JavaScript : Language Level = ECMAScript 2022 이상
- 설정 ⟶ 언어 및 프레임워크 ⟶ TypeScript :
    - **TypeScript 언어 서비스 체크** (tsconfig 자동 감지)
- 설정 ⟶ 언어 및 프레임워크 ⟶ Javascript ⟶ 코드 품질 도구 ⟶ ESLint :
    - **자동 ESLint 구성(A)** 선택 
    - “다음 파일에 대해 실행”에 *.vue, *.ts, *.js 포함되어 있는지 확인
- 설정 ⟶ 도구 ⟶ 저장 시 액션:
    - **eslint –fix 실행** 체크 (가능하면)
    - **코드 서식 다시 지정** + **import 문 최적화** 켜두면 편함
- Prettier 사용 시:
    - 설정 ⟶ 언어 및 프레임워크 ⟶ Javascript ⟶ Prettier
        - **자동 Prettier 구성(A)** 선택 
    - 또는 저장 시 액션에서 **Prettier 실행** 체크

> ESLint + Prettier를 함께 선택해 생성했다면 충돌 규칙은 템플릿에서 이미 정리되어 있습니다(@vue/eslint-config-prettier).

---

#### 3. Run/Debug 설정 (Vite dev 서버)

**Run ⟶ Edit Configurations… ⟶ + ⟶ npm**

- Name: Vite: dev
- Package.json: frontend/package.json
- Command: run
- Scripts: dev
- Node interpreter: (위에서 선택한 것) 
- **Before launch**: (Optional) npm install

이제 상단의 **실행/디버그** 버튼으로 npm run dev를 PyCharm에서 바로 돌릴 수 있습니다.

브레이크포인트는 클라이언트 JS/TS 코드에서 **소스맵** 기반으로 잘 잡힙니다.

---

#### 4. Django와 연동(개발 단계)

**4-1. CORS/프록시(권장)**

Vite dev 서버(5173) → Django API(8000) 로 프록시하면 CORS 신경 덜 씁니다.

vite.config.js 예시:

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    port: 5173,
    open: true,
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:8000',
        changeOrigin: true
      }
    }
  }
})
```

프론트에서 /api/...로 호출하면 자동으로 Django 8000으로 전달됩니다.

**4-2. PyCharm “Compound” 구성(두 서버 동시 실행)**

Run ⟶ Edit Configurations…

- - ⟶ **Django Server** (백엔드) 추가
- - ⟶ **npm (Vite: dev)** 추가(위 3단계)
- - ⟶ **Compound** 추가 → 위 두 설정을 묶기

이제 한 번에 **Django + Vite dev 서버**를 동시에 실행 가능.

---

#### 5. Django와 연동(배포 단계)

원하는 경로로 Vite 빌드 산출물을 **Django static**으로 떨구면 편해요.

**5-1. Vite 빌드 출력 폴더 변경**

Django 프로젝트 구조 예:

```
project_root/
  backend/  (Django)
    static/
      frontend/   ← 여기에 빌드 산출물 투하
  frontend/ (Vite+Vue)
```

vite.config.js:

```javascript
export default defineConfig({
  plugins: [vue()],
  build: {
    outDir: '../backend/static/frontend',
    emptyOutDir: true
  }
})
```

**5-2. 빌드 & Django 템플릿 연결**

```shell
cd frontend
npm run build
```

이제 Django 템플릿에서 빌드된 자산을 로딩:

backend/templates/base.html 예시:

```html+django
{% load static %}
<!doctype html>
<html>
<head>
  <link rel="stylesheet" href="{% static 'frontend/assets/index-xxxx.css' %}">
</head>
<body>
  <div id="app"></div>
  <script src="{% static 'frontend/assets/index-xxxx.js' %}" type="module"></script>
</body>
</html>
```

> 파일명에 해시가 붙습니다. **django-webpack-loader** 류를 쓰거나, 또는 Vite의 manifest.json을 읽어 정적 경로를 동적으로 주입하는 방법도 있습니다.
>
> (규모가 커지면 manifest 방식 권장)

---

#### 6. 폴더 구조 추천

```
repo/
 ┣ backend/                # Django
 ┃ ┣ manage.py
 ┃ ┣ backend/...
 ┃ ┗ static/
 ┃    ┗ frontend/          # Vite 빌드 결과 (outDir)
 ┣ frontend/               # Vite + Vue
 ┃ ┣ src/
 ┃ ┣ index.html
 ┃ ┣ vite.config.ts
 ┃ ┣ package.json
 ┃ ┗ tsconfig.json
 ┗ README.md
```

---

#### 7. 품질/편의 설정 팁

- **Path alias** (@/…)

vite.config.js

```javascript
import path from 'path'
export default defineConfig({
  resolve: { alias: { '@': path.resolve(__dirname, './src') } }
})
```

- **ESLint 오류 자동 고치기**
    
    - 설정 ⟶ Tools ⟶ Actions on Save ⟶ ESLint: Run eslint --fix 체크
        
    
- **Prettier 규칙 공유**
    
prettier.config.cjs:

```javascript
module.exports = {
  semi: false,
  singleQuote: true,
  printWidth: 100,
  trailingComma: 'none'
}
```

- **API 주소 분리**
    
    .env / .env.production 사용:
    

```dotenv
VITE_API_BASE=/api
```

- 코드에서: import.meta.env.VITE_API_BASE
    

---

#### 8. 자주 겪는 이슈

- **ESLint가 안 돈다**
    
    - node_modules/eslint 존재 확인
    - PyCharm ESLint 설정 “Automatic”인지 확인
    - 프로젝트 루트에 .eslintrc.* 있는지 확인

- **Vue 구문 강조/자동완성 약함**
    
    - PyCharm Pro의 **Vue.js 플러그인** 활성화 확인
    - 프로젝트 SDK(=Node 인터프리터) 경로 올바른지 확인

- **CORS 에러**
    
    - 개발 중: Vite server.proxy 사용 (위 예시) 
    - 배포: Django CORS 설정 or 같은 도메인/경로로 정적 제공

---

#### 한 줄 정리

- **생성은** **npm create vue@latest** (ESLint+Prettier 포함 추천)
- **PyCharm Pro**에서 Node 인터프리터/ESLint/Prettier/Vue 인식만 켜주면 끝
- **개발**은 Vite dev 서버 + (선택) Django API 프록시
- **배포**는 Vite 빌드 결과를 Django static/으로 떨구고 템플릿에서 로드
    
