
이건 Vue의 반응형 시스템을 이해하는 “가장 핵심 포인트”예요.

v-model이 실제 내부에서 **무엇을 어떻게 하는지** 단계별로 아주 정확히 정리해드릴게요

---

### 1. v-model은 문법적 “축약형(syntax sugar)”

**v-model="data"**

내부적으로 Vue가 다음 두 줄로 풀어서 처리합니다:

```vue
<input
  :value="data"                       <!-- 데이터 → 화면 -->
  @input="data = $event.target.value" <!-- 화면 → 데이터 -->
/>
```

즉,

- :value → 데이터(data)를 input의 value에 넣는다.
    
- @input → 사용자가 입력할 때마다, 그 값을 다시 data로 돌려준다.

결국, **데이터 ↔ 화면이 서로 연결**된 거예요

---

### 2. 한 줄씩 해부해보기

**(1) :value="data"

- :는 **v-bind의 축약형** → “속성에 JS 표현식을 바인딩”
    
- 즉, input의 value 속성을 항상 data 값과 동일하게 유지합니다.

예를 들어

```vue
<input :value="data" />
```

이면,  data가 'Hello'라면 실제 DOM은 \<input value="Hello" \/\> 로 표시됩니다.

**데이터 → 화면 방향**

---

**(2) @input="data = $event.target.value"

- @input은 **input 이벤트 리스너**를 등록하는 문법입니다.
    
- 사용자가 타이핑하면 input 이벤트가 발생하고,
    
- $event는 그 이벤트 객체를 뜻합니다.
    
- $event.target은 실제 \<input\> 요소,
    
- $event.target.value는 입력한 실제 문자열입니다.
    

따라서 👇

```vue
@input="data = $event.target.value"
```

는

> “입력창의 값이 바뀌면, 그 값을 data 변수에 대입하라.”

**화면 → 데이터 방향**

---

### 3. 정리하면 내부 동작 흐름은 이렇게 됩니다

```
          (1) :value
     data → input.value
          (2) @input
     input.value → data
```

즉, Vue는 data와 \<input\>의 value 속성을 **양방향으로 동기화**하는 거예요.

---

### 4. 동작 순서 실제 예시

```vue
<script setup>
import { ref } from 'vue'
const name = ref('홍길동')
</script>

<template>
  <input v-model="name" />
  <p>{{ name }}</p>
</template>
```

**1. 초기 렌더링**

- name.value = '홍길동'
    
- Vue가 :value="name" 으로 input.value = '홍길동' 설정
    
    → 화면에 “홍길동” 표시

**2. 사용자 입력**

- 사용자가 “이영희” 입력
    
- input 이벤트 발생
    
- @input="name = $event.target.value" 실행
    
    → name.value = '이영희'
    
**3. 반응형 시스템 동작**

- name이 바뀜 → Vue의 reactivity 시스템이 감지
    
- 관련된 모든 바인딩({{ name }} 등)이 자동 재렌더링됨
    
    → <p> 내용이 즉시 “이영희”로 바뀜

---

### 5. Vue가 자동으로 관리하는 반응형 연결

Vue는 내부적으로 다음과 같은 “감시 체계”를 둡니다

```
┌────────────┐        :value         ┌────────────┐
│  data(ref) │ ───────────────────▶ │  input UI  │
└────────────┘                       └────────────┘
       ▲                                      │
       │              @input                  │
       └─────────────◀────────────────────────┘
```

- 데이터(ref)가 바뀌면 → 화면(UI)이 자동 갱신
    
- 화면에서 입력이 바뀌면 → 데이터(ref)가 자동 갱신

이 양쪽 흐름을 Vue가 자동으로 관리하는 게 바로 **v-model**이에요.

---

### 6. 타입별 내부 처리 차이 (Vue가 알아서 해줌)

| **요소**                    |**실제 내부적으로 하는 일**|
|---------------------------|---|
| \<input type="text"\>     |:value + @input|
| \<input type="checkbox"\> |:checked + @change|
| \<input type="radio"\>    |:checked + @change|
| \<select\>                |:value + @change|
| \<textarea\>              |:value + @input|

즉, v-model은 단순히 text input뿐 아니라,

checkbox, radio, select 등에도 각각 알맞은 방식으로 작동하도록 Vue가 내부에서 자동 분기 처리합니다.

---

### 7. 수식어 (Modifiers)의 실제 내부 동작도 단순한 추가 코드

|**수식어**|**내부적으로 바뀌는 코드**|
|---|---|
|.trim|$event.target.value.trim()|
|.number|Number($event.target.value)|
|.lazy|@change 이벤트로 변경 (입력 완료 후 반영)|

예시

```vue
<input v-model.trim="msg" />
```

내부적으로는

```vue
<input
  :value="msg"
  @input="msg = $event.target.value.trim()"
/>
```

---

### 8. 한눈에 요약

|**항목**|**내용**|
|---|---|
|**v-model의 정체**|:value + @input 의 축약형|
|**데이터 흐름**|데이터 → 화면, 화면 → 데이터|
|**이벤트 객체**|$event.target.value 로 입력값을 가져옴|
|**자동 감지 시스템**|Vue의 반응형(Reactivity)으로 자동 렌더링|
|**타입별 자동 처리**|input/checkbox/radio/select 모두 지원|
|**수식어**|.trim, .number, .lazy 등은 단순히 추가 전처리 코드|

