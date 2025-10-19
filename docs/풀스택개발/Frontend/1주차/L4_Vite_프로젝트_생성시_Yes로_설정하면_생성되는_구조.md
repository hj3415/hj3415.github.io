npm create vue@latest로 프로젝트를 만들 때 **Router / Pinia / ESLint / Prettier**를

모두 “Yes”로 선택하면,

**수동으로 해줄 설치 + 설정 작업이 자동으로 다 구성됩니다.**

> “Yes”를 누르면 Vue CLI가 미리 그 모든 설정 파일과 패키지를 **자동 설치 + 연결 + 초기 세팅**까지 끝내주는 것과 같습니다.

---

#### 항목별로 실제 자동으로 되는 일

|**항목**| **“Yes” 선택 시 자동으로 되는 일**                                                                                                                   |
|---|--------------------------------------------------------------------------------------------------------------------------------------------|
|**Vue Router**| ✅ vue-router@4 자동 설치<br/>✅ src/router/index.js 생성<br/>✅ App.vue에 <router-view />와 <router-link> 기본 포함<br/>✅ main.js에 app.use(router) 자동 추가 |
|**Pinia**| ✅ pinia 자동 설치<br/>✅ src/stores/counter.js 생성<br/>✅ main.js에 app.use(createPinia()) 자동 추가                                                   |
|**ESLint**| ✅ eslint, eslint-plugin-vue, @vue/eslint-config-* 자동 설치<br/>✅ eslint.config.js 자동 생성<br/>✅ package.json에 "lint" 스크립트 추가 |
|**Prettier**| ✅ prettier, @vue/eslint-config-prettier 자동 설치<br/>✅ .prettierrc.json<br/>✅ ESLint와 충돌 안 나게 규칙 자동 조정                                        |

---

#### 실제로 자동 생성되는 주요 파일 구조

예를 들어 **Vue 3 + JS + Router + Pinia + ESLint + Prettier**로 만들면

```
src/
 ├ assets/
 ├ components/
 │  └ HelloWorld.vue
 ├ router/
 │  └ index.js
 ├ stores/
 │  └ counter.js
 ├ views/
 │  ├ HomeView.vue
 │  └ AboutView.vue
 ├ App.vue
 └ main.js
eslint.config.js
.prettierrc.json
tsconfig.json
vite.config.js
package.json
```

---

#### 자동 설치되는 패키지 예시

```json
"dependencies": {
    "pinia": "^3.0.3",
    "vue": "^3.5.22",
    "vue-router": "^4.5.1"
  },
  "devDependencies": {
    "@eslint/js": "^9.33.0",
    "@vitejs/plugin-vue": "^6.0.1",
    "@vue/eslint-config-prettier": "^10.2.0",
    "eslint": "^9.33.0",
    "eslint-plugin-vue": "~10.4.0",
    "globals": "^16.3.0",
    "prettier": "3.6.2",
    "vite": "^7.1.7",
    "vite-plugin-vue-devtools": "^8.0.2"
  }
```

---

#### 자동 생성되는 주요 설정 예시

**eslint.config.js**

```javascript
export default defineConfig([
  {
    name: 'app/files-to-lint',
    files: ['**/*.{js,mjs,jsx,vue}'],
  },

  globalIgnores(['**/dist/**', '**/dist-ssr/**', '**/coverage/**']),

  {
    languageOptions: {
      globals: {
        ...globals.browser,
      },
    },
  },

  js.configs.recommended,
  ...pluginVue.configs['flat/essential'],
  skipFormatting,
])

```

**.prettierrc.json**

```json
{
  "$schema": "https://json.schemastore.org/prettierrc",
  "semi": false,
  "singleQuote": true,
  "printWidth": 100
}
```

**main.js**

```javascript
import { createApp } from 'vue'
import { createPinia } from 'pinia'

import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(createPinia())
app.use(router)

app.mount('#app')
```

---

**차이를 요약하면**

|**구분**| **“No”로 시작 (수동 추가)**                           |**“Yes”로 시작 (자동 구성)**|
|---|------------------------------------------------|---|
|패키지 설치| 직접 npm i vue-router pinia eslint prettier 등 설치 |자동 설치|
|설정 파일| 직접 eslint.config.js, .prettierrc.json 작성       |자동 생성|
|폴더 구조| src/router, src/stores 직접 생성                   |자동 생성|
|main.ts 수정| 직접 app.use() 추가                                |자동 추가|
|App.vue 수정| 직접 <router-view /> 추가                          |자동 포함|
|ESLint/Prettier 충돌 처리| 직접 구성                                          |자동 조정|
