Vue 3를 막 배우기 시작하면 **.value 언제 써야 하는지** 헷갈리는 게 당연해요.

이건 Vue의 **반응형 시스템(ref, reactive)** 이 작동하는 원리 때문이에요.

---

### 1. 먼저 ref()가 뭐냐면

ref()는 **“하나의 값을 반응형으로 감싸는 함수”**예요.

→ 즉, **“상자 안에 실제 값이 들어 있는 구조”**

```vue
import { ref } from 'vue'

const count = ref(0)
```

이때 내부 구조는 이렇게 생겼어요

```vue
count = { value: 0 } // 실제로는 이런 객체
```

그래서 JS 코드에서 쓸 때는 반드시 .value를 통해 진짜 값을 꺼내야 해요.

```vue
console.log(count.value) // 0
count.value++            // 값 변경
```

---

### 2. 그런데 템플릿(template) 안에서는 .value를 안 써도 된다!

Vue가 **자동으로 .value를 풀어서 보여주기 때문이에요.**

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <p>{{ count }}</p>  <!-- ✅ .value 안 써도 자동으로 풀림 -->
  <button @click="count++">+1</button> <!-- ✅ 가능 -->
</template>
```

→ Vue 템플릿 내부에서는 ref를 **자동으로 언래핑(unwrap)** 해줍니다.

즉, {{ count }} → count.value,

count++ → count.value++ 로 내부적으로 처리돼요.

---

### 3. 반대로, JS 영역에서는 .value를 써야 함

템플릿이 아닌 **setup() 내부나 일반 함수, watch, computed 등 JS 코드**에서는

Vue가 자동으로 언래핑을 해주지 않아요.

즉

```vue
const count = ref(0)
console.log(count)       // { value: 0 } 객체
console.log(count.value) // 0  ✅
```

---

### 4. 예시로 구분해보기

|**위치**|**코드**|.value **필요?**|**이유**|
|---|---|---|---|
|**setup 내부**|count.value++|✅ 필요|JS 코드 안이기 때문|
|**watch**|watch(count, (val) => console.log(val))|❌ 자동 처리됨|watch가 내부에서 unwrap|
|**computed 내부**|computed(() => count.value * 2)|✅ 필요|계산식은 JS 문맥|
|**template**|{{ count }}|❌ 자동 언래핑|템플릿은 Vue가 해석|
|**template 이벤트**|@click="count++"|❌ 자동 언래핑|템플릿 문맥이기 때문|

---

### 5. reactive()는 조금 다름

reactive()는 객체 전체를 반응형으로 만들어줍니다.

```vue
const user = reactive({ name: 'Kim', age: 20 })
```

→ 여기선 .value가 필요 없습니다.

```vue
<template>
  <p>{{ user.name }}</p>  <!-- ✅ 바로 접근 -->
</template>
```

reactive는 내부 속성 자체가 반응형이기 때문이에요.

---

### 6. 정리 요약표

|**선언 방식**|**예시**|**접근 방식**|**이유**|
|---|---|---|---|
|ref()|const count = ref(0)|✅ JS: count.value✅ template: count|단일값을 감싼 객체|
|reactive()|const user = reactive({ name:'Kim' })|✅ JS: user.name✅ template: user.name|객체를 직접 반응형 처리|
|computed()|const doubled = computed(() => count.value * 2)|✅ JS: doubled.value✅ template: doubled|ref와 동일하게 동작|

---

**핵심 문장 (외우기용)**

> **“JS 안에서는 .value 써야 하고,**
>
> **템플릿 안에서는 Vue가 알아서 .value를 대신 붙여준다.”**

---

**기억 꿀팁**

> “setup() 안에서는 ref는 상자야.
>
> 화면(template)에서는 그 상자를 자동으로 열어 보여준다.”
