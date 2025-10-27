Hypothesis는 단순한 랜덤 테스트 도구가 아니라, 
**함수의 “성질(property)”을 자동 검증하는 테스트 엔진**이기 때문에
개념적으로는 조금 다르게 접근해야 합니다.

아래는 “pytest → hypothesis 심화”로 이어지는 **완성형 커리큘럼**이에요.

---

## Hypothesis 학습 커리큘럼 (단계별)


### 1단계. Hypothesis의 기본 철학 이해

**목표: “Property-Based Testing” 개념을 직관적으로 이해**

|**주제**|**설명**|**실습 예제**|
|---|---|---|
|Property-Based Testing이란?|입력값을 수동으로 주는 대신, 프로그램이 자동으로 수백 개의 입력을 시도하여 **항상 참이어야 하는 성질(property)** 을 검증하는 테스트 방식|a + b == b + a, sorted(sorted(xs)) == sorted(xs)|
|Hypothesis의 동작 원리|입력 생성 → 테스트 실행 → 실패 탐지 → 최소 입력 축소(shrinking)|리스트 정렬 함수 테스트|
|Hypothesis vs pytest|pytest는 “케이스 기반”, hypothesis는 “성질 기반”|비교표 실습|

**핵심 이해 포인트**

> “예시를 검사하는 게 아니라, **함수의 성질을 증명**한다.”

---

### 2단계. 기본 문법과 전략(strategies) 익히기

**목표: Hypothesis의 입력 생성기(Strategy) 개념 이해**

|**주제**|**설명**|**실습 예제**|
|---|---|---|
|@given()|테스트 함수에 자동으로 입력 주입|@given(st.integers())|
|strategies (st 모듈)|데이터 생성 규칙 정의|st.text(), st.integers(), st.lists()|
|복합 전략|중첩 구조 만들기|st.tuples(st.integers(), st.text())|
|데이터 제약조건|min_size, max_value, allow_nan 등|st.lists(st.integers(), min_size=1)|
|예제 출력 확인|.example()|st.text().example()|

**핵심 이해 포인트**

> Hypothesis는 “랜덤값 생성기”가 아니라,
> “데이터 구조를 설명하는 DSL(Domain-Specific Language)” 이다.

---

### 3단계. Property(성질) 설계 훈련

**목표: “어떤 것이 항상 참이어야 하는가”를 추상화하는 능력 기르기**

|**주제**|**설명**|**실습 예제**|
|---|---|---|
|교환법칙 / 결합법칙|수학적 성질로 접근|assert add(a,b) == add(b,a)|
|역전의 역전은 원래로|문자열, 리스트|s[::-1][::-1] == s|
|정렬의 멱등성|정렬 두 번 == 한 번|sorted(sorted(xs)) == sorted(xs)|
|구조적 property|JSON 직렬화|json.loads(json.dumps(x)) == x|
|round-trip property|변환 후 복원은 원본과 같아야 함|encode/decode, dump/load|

**핵심 이해 포인트**

> 테스트 대상 함수의 **불변 조건(invariant)** 을 찾아내는 연습을 해야 함.

---

### 4단계. 실패 케이스 탐색 & shrinking 이해

**목표: Hypothesis가 버그를 “자동으로 좁혀가는” 원리 익히기**

|**주제**|**설명**|**실습 예제**|
|---|---|---|
|실패 예제 재현|실패 시 자동 저장된 example 재사용|pytest --hypothesis-show-statistics|
|Shrinking 과정|가장 단순한 예제 자동 탐색|리스트나 문자열의 최소화 확인|
|임계값 찾기|특정 범위에서 깨지는 케이스 탐색|나눗셈 함수에서 b=0 찾기|

**핵심 이해 포인트**

> Hypothesis는 “랜덤”이 아니라 “탐색적”이다.
> 실패를 발견하면 그걸 가장 단순한 입력으로 압축한다.

---

### 5단계. 전략 조합 (고급 전략 디자인)

**목표: JSON, Dict, Model 같은 복합 구조 자동 생성 익히기**

|**주제**|**설명**|**실습 예제**|
|---|---|---|
|st.fixed_dictionaries()|JSON 구조 자동 생성|{“id”: int, “name”: str}|
|st.builds()|dataclass / Pydantic 모델 자동 생성|st.builds(User, id=st.integers(), name=st.text())|
|st.from_type()|타입 힌트 기반 데이터 생성|@given(st.from_type(UserModel))|
|st.sampled_from()|샘플 기반 랜덤 선택|st.sampled_from(["A", "B", "C"])|
|st.recursive()|중첩 구조 (트리, JSON 등)|st.recursive(st.none(), lambda children: st.lists(children))|

**핵심 이해 포인트**

> Hypothesis는 타입 정보나 스키마로부터 **데이터 생성기를 자동 합성**할 수 있다.

---

### 6단계. pytest와 통합 사용법

**목표: pytest와 함께 효율적인 워크플로우 확립**

|**주제**|**설명**|**실습 예제**|
|---|---|---|
|pytest와 함께 실행|그냥 pytest 실행 시 자동 인식|pytest tests/|
|Hypothesis 설정|.hypothesis 디렉토리, 캐시, 시드(seed) 관리|--hypothesis-seed=1234|
|예외 검증|@given(...) + pytest.raises()|잘못된 입력 감지|
|Fixtures와 함께 사용|pytest fixture + hypothesis 전략 조합|@given(data=st.data())|

**핵심 이해 포인트**

> pytest는 실행 플랫폼, hypothesis는 입력 엔진이다.
> 두 개를 결합하면 “자동화된 테스트 생성기”가 된다.

---

### 7단계. 고급 주제 (실무 활용)

**목표: 실제 서비스 코드에서 property testing 적용**

|**주제**|**설명**|**실습 예제**|
|---|---|---|
|API 검증|JSON request/response 구조 property 테스트|FastAPI 엔드포인트 테스트|
|DB 모델 round-trip|ORM save/load 일관성 검증|SQLAlchemy / Beanie round-trip|
|파서 검증|parse → serialize → parse property|CSV/JSON 변환기|
|수학/통계 검증|수치 안정성, overflow 방지|NaN, inf 처리 검증|
|Hypothesis 플러그인|hypothesis-jsonschema, hypothesis-graphql|JSON 스키마 기반 자동 생성|

**핵심 이해 포인트**

> Hypothesis는 단순 테스트를 넘어 **시스템적 검증 도구**로 확장할 수 있다.

---

### 8단계. 학습 마무리 및 실전 프로젝트

**목표: 직접 Hypothesis 기반 테스트 스위트를 만들어보기**

|**주제**|**실습**|
|---|---|
|🎯 작은 프로젝트 하나 테스트|예: 수식 계산기, 문자열 변환기, 정렬 알고리즘|
|🎯 API 요청 검증 자동화|FastAPI 엔드포인트의 입력/출력 property 테스트|
|🎯 JSON 변환 property test|json.loads(json.dumps(data)) == data|
|🎯 랜덤 구조체 생성기 만들기|example 기반 랜덤 데이터 생성기 (.example() 사용)|

---

**학습 순서 요약 (핵심 흐름)**

|**단계**|**주제**|**키워드**|
|---|---|---|
|1️⃣|철학 이해|Property-Based Testing|
|2️⃣|기본 문법|@given, strategies|
|3️⃣|성질 설계|Invariant / Round-trip|
|4️⃣|Shrinking 이해|최소 실패 예제|
|5️⃣|복합 구조 생성|fixed_dictionaries, builds|
|6️⃣|pytest 통합|fixture, config|
|7️⃣|실무 적용|API, JSON, DB roundtrip|
|8️⃣|프로젝트 실습|자동 검증 스위트 완성|
