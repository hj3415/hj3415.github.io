Vue router, Pinia, Eslint, Prettier 전부 나중에 추가해도 깔끔하게 붙일 수 있습니다.
 (create-vue에서 “No”로 시작해도 문제 없고, 필요해질 때 설치·연결만 해주면 돼요.)

아래에 각 항목별 최소 절차와 주의점만 콕 집어드릴게요.

---

#### 1. Vue Router 추가

**설치**

```shell
npm i vue-router@4
```

**파일 추가** 

src/router/index.\[jt\]s

```typescript
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/views/Home.vue'
const routes = [{ path: '/', name: 'home', component: Home }]
export const router = createRouter({ history: createWebHistory(), routes })
```

**main에 연결** 

src/main.\[jt\]s

```typescript
import { router } from './router'
app.use(router)
```

**App.vue**에 출력 자리

```vue
<router-view />
```

**주의**

- 배포 시 **history 모드**면 서버(Nginx 등)에서 try_files ... /index.html; 설정 필요.
  
  (간단히는 createWebHashHistory()로 시작해도 됨.)
    
- @ 별칭을 안 썼다면 상대경로(../views/...)로 먼저 쓰고 나중에 alias 추가.
    

---

#### 2. Pinia 추가

**설치**

```shell
npm i pinia
```

**main에 연결**

```typescript
import { createPinia } from 'pinia'
const pinia = createPinia()
app.use(pinia)
```

**스토어 예시** 

src/stores/counter.\[jt\]s

```typescript
import { defineStore } from 'pinia'
export const useCounter = defineStore('counter', {
  state: () => ({ n: 0 }),
  actions: { inc() { this.n++ } }
})
```

**주의**

- 새로고침 후 상태 유지가 필요하면 pinia-plugin-persistedstate로 영속화 가능.
- 타입스크립트 사용 시 대부분 자동 추론으로 충분.
    

---

#### 3. ESLint 추가

**설치 (JS 프로젝트)**

```shell
npm i -D eslint eslint-plugin-vue @vue/eslint-config-prettier
```

**설치 (TS 프로젝트)**

```shell
npm i -D eslint eslint-plugin-vue @typescript-eslint/parser @typescript-eslint/eslint-plugin @vue/eslint-config-typescript @vue/eslint-config-prettier
```

**설정 파일** 

eslint.config.js(수동으로 만들어야 함)

```javascript
import js from "@eslint/js"
import pluginVue from "eslint-plugin-vue"
import skipFormatting from "@vue/eslint-config-prettier/skip-formatting"

export default [
  js.configs.recommended,
  ...pluginVue.configs["flat/essential"],
  skipFormatting,
]
```

실행

```shell
npx eslint .
```

**스크립트** 

package.json에 다음을 추가

```json
{ "scripts": { "lint": "eslint . --fix" } }
```

npm run lint 실행가능

**주의**

- 에디터에서 “저장 시 자동 고치기”를 켜두면 체감이 가장 좋습니다.
- .eslintignore에 dist, node_modules 추가.
    

---

#### 4. Prettier 추가

**설치**

```shell
npm i -D prettier
```

**설정 파일** 

.prettierrc.json(수동으로 생성해야함. 예시 기본 규칙)

```json
{
  "$schema": "https://json.schemastore.org/prettierrc",
  "semi": false,
  "singleQuote": true,
  "printWidth": 100
}

```

**스크립트**

package.json에 다음을 추가

```json
{ "scripts": { "format": "prettier --write src/" } }
```

npm run prettier 실행가능

**주의**

- **ESLint와 충돌 방지**를 위해 위에서 넣은 @vue/eslint-config-prettier가 필수.
- 에디터 기본 포매터를 Prettier로 지정하세요.

---

#### 결론

- **Router/Pinia/ESLint/Prettier** 모두 **후추가가 용이**합니다. 
- “Yes로 시작”은 편의의 문제일 뿐, “No로 시작”해도 **몇 개 파일/한두 줄 연결**로 충분히 붙습니다.
- 유의할 점은 **라우터(history 모드 서버 설정)**와 **ESLint–Prettier 충돌 방지** 두 가지뿐!