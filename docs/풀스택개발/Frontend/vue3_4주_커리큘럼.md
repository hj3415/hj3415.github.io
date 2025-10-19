### **전체 목표**

> - Vue 3 Composition API 완전 이해 
> - Django REST API와 연동 가능한 실전 구조 이해 
> - 컴포넌트/라우터/상태관리/빌드까지 한 번에 마스터 
> - “Vue로 관리자 페이지, 대시보드, 블로그 프론트” 직접 제작 가능

---

### 1주차: Vue 3 기초 & Composition API 감 잡기

**Day 1: Vue란 무엇인가 + 개발 환경 세팅**

- Vue 2 vs Vue 3 차이 
- CDN 방식으로 시작
- Vite(Vue CLI 대체)의 장점
- 프로젝트 생성 + VSCode 세팅 (Volar, ESLint)

**Day 2: 템플릿 문법 (Django Template 매핑)**

- {{ }} 데이터 바인딩 
- v-bind / v-on (@)
- v-if / v-else / v-show
- v-for (key 개념)
- v-model (양방향 바인딩)

**Day 3: Composition API 기초 ①**

- setup() 이해 
- ref / reactive
- methods 대신 함수
- return으로 템플릿 연결

**Day 4: Composition API 기초 ②**

- computed (읽기 전용)
- watch (값 감지)
- watchEffect

**Day 5: 실습 미니 프로젝트 ①**

- 간단한 TODO List 
- v-model + v-for + @click 사용
- 완료 / 삭제 / 필터 기능

---

### **2주차: 컴포넌트 & 데이터 흐름 완전 마스터**

**Day 6: 컴포넌트 개념**

- 컴포넌트 등록 방식 (전역/지역)
- 파일 구조 (Single File Component)

**Day 7: Props (부모 -> 자식)**

- 타입 지정
- 필수 / 기본값
- 데이터 전달 패턴

**Day 8: Emits (자식 -> 부모)**

- 이벤트 발생
- payload 전달
- v-model 커스텀 패턴

**Day 9: Slot (템플릿 확장)**

- 기본 슬롯
- 이름 있는 슬롯
- Scoped slots (강력!)

**Day 10: 실습 미니 프로젝트 ②**

- 컴포넌트 분리로 TODO 개선
- 입력창, 리스트, 필터, 헤더 분리

---

### **3주차: Vue Router + 상태관리(Pinia) + API 연동**

**Day 11: Vue Router ① 기본**

- router 설치 & 세팅
- createRouter / createWebHistory
- path → component 매핑

**Day 12: Vue Router ② 고급**

- 동적 라우팅 (/post/:id)
- 중첩 라우트
- 네비게이션 가드(로그인 제한) → Django middleware 느낌

**Day 13: Pinia (상태관리)**

- store 만들기
- state / getters / actions
- 여러 컴포넌트에서 공유

**Day 14: axios로 Django REST API 연동**

- GET/POST/PUT/DELETE
- async/await
- try/catch 에러 핸들링
- 인터셉터로 토큰 자동 설정

**Day 15: 실습 미니 프로젝트 ③ (가짜 API 연결)**

- json-server 또는 fastapi 예시
- 게시판 목록 + 상세 + 작성 + 수정 + 삭제

---

### **4주차: 실전 감각 완성 & Django 연결 & 배포**

**Day 16: Django REST API 직접 연결**

- Django Ninja or DRF로 API 생성
- CORS 설정 
- Vue에서 실제 요청 보내기 
- JSON 구조 설계

**Day 17: 인증(로그인) 구현**

- JWT or Session 기반 로그인
- Vue에서 로그인 → 토큰 저장(Pinia)
- 보호된 페이지 접근 제어 (라우터 가드)

**Day 18: UI 컴포넌트 라이브러리 사용**

- Tailwind CSS 혹은 Element Plus, Vuetify 중 택1
- 버튼, 모달, 테이블, 폼 활용법

**Day 19: 최종 미니 프로젝트 (Django + Vue Fullstack)**

예시:
- 블로그 프론트엔드
- 관리자 대시보드
- 상품 리스트 + 검색 + 필터
- 댓글, 좋아요 기능

➡ 실전에서 자주 쓰는 패턴 총집합

**Day 20: 빌드 & 배포**

- npm run build
- Django static으로 서빙
- Nginx로 Vue 별도 배포
- 폴더 구조 정리
- 성능 최적화 (lazy loading, code splitting)

---

**학습 후에는 이런 상태가 됩니다!**

- Vue 3 Composition API 완전 이해 
- Django 개발자 시각에서 Vue를 “자기 언어”처럼 다룸 
- Vue Router / Pinia까지 자연스럽게 사용 
- Django REST API 연동하여 실제 서비스 개발 가능 
- 앞으로 어떤 프론트 프로젝트가 와도 겁 안 남