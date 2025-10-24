
## Pydantic 완전 정복 커리큘럼 (실전형, 단계별)

> 기준: Python 3.11+, **Pydantic v2**, FastAPI + Beanie 연계 전제

---

### 1단계: 기초 개념 이해 (Pydantic의 철학)

> 목표: “왜 Pydantic이 필요한가?”를 직관적으로 이해

**학습 항목**

- 데이터 검증(validation)과 직렬화(serialize)의 개념
    
- Python dataclass vs pydantic.BaseModel 차이
    
- BaseModel의 핵심 기능
    
- 동작 원리: 입력 → 검증 → 변환 → 출력
    
- JSON ↔ dict ↔ 객체 변환 과정

**실습**

- 잘못된 타입 입력 시 자동 변환/오류 확인
    
- .model_dump(), .model_validate(), .model_dump_json() 체험
    
- @dataclass 와 Pydantic BaseModel 비교 코드
    

---

### 2단계: BaseModel 실전 문법

> 목표: DTO/Schema를 자유롭게 설계할 수 있게 만들기

**학습 항목**

- 기본 필드 선언 및 타입 힌트 (str, int, datetime, UUID, EmailStr)
    
- 기본값, Field(default=...), Field(default_factory=...)
    
- 필수/옵션 필드 구분
    
- Nested 모델 (중첩 모델)
    
- alias, populate_by_name, json_schema_extra
    
- 모델 상속 / Mixin 패턴
    
**실습**

- User, Item 모델 작성
    
- Nested 모델 (User 안에 Address)
    
- Field 제약 (min_length, regex 등)
    
- 상속 구조로 BaseModel 공통화

---

### 3단계: 검증(Validation) 고급

> 목표: 비즈니스 로직 수준의 데이터 검증 구현

**학습 항목**

- @field_validator (단일 필드 검증)
    
- @model_validator (모델 전체 검증)
    
- 순서 (mode="before", "after")
    
- 커스텀 예외 메시지
    
- 동적 필드 생성 (computed_field)
    
- 타입 변환(before 단계 변환기)

**실습**

- 비밀번호 검증(len >= 8, contains number)
    
- 이메일 중복 금지 검증
    
- 날짜 검증 (end_date > start_date)
    
- computed_field로 full_name 필드 생성

---

### 4단계: 설정(Config / model_config)

> 목표: Pydantic의 전역 동작을 제어할 수 있게 만들기

**학습 항목**

- v1의 Config vs v2의 model_config
    
- ConfigDict 주요 옵션
    
    - from_attributes (ORM 호환)
        
    - extra (“ignore” / “allow” / “forbid”)
        
    - populate_by_name
        
    - validate_assignment
        
    - use_enum_values

- 설정 상속 패턴 (공통 ContractModel 만들기)

**실습**

- ContractModel 베이스 클래스 구현
    
- extra="forbid" 적용 시 미정의 필드 테스트
    
- ORM 객체에서 from_attributes=True로 변환 실습

---

### 5단계: 직렬화(Serialization) & 변환

> 목표: API 입출력, DB 매핑에 필요한 변환 로직 이해

**학습 항목**

- .model_dump(), .model_dump_json() 옵션
    
- include / exclude / exclude_none
    
- by_alias 옵션
    
- 모델 병합 (model_copy(update=...))
    
- Enum, datetime, UUID의 JSON 변환
    
- Beanie/SQLAlchemy 모델과의 변환 (from_attributes)

**실습**

- UserDoc → UserRead 변환
    
- dict → 모델 / 모델 → JSON
    
- FastAPI ResponseModel 직렬화 차이 확인

---

### 6단계: 모델 설계 패턴 (실전 DTO 구조)

> 목표: contracts 패키지 구조를 안정적으로 설계할 수 있게 하기

**학습 항목**

- Create / Update / Read / Event DTO 패턴
    
- 공통 Mixin (IDModel, TimestampMixin, ContractModel)
    
- Enum, Custom Type 관리 (common.py)
    
- 모델 버전 관리 (v1, v2 schema)
    
- API 입력/출력과 DB 저장 모델 분리 개념

**실습**

- contracts/users.py 완성 (Create/Update/Read)
    
- contracts/common.py 에 Enum / Mixin 정리
    
- 이벤트 DTO 설계 (UserCreatedEvent)

---

### 7단계: 유효성 검증 & 예외 처리 심화

> 목표: Pydantic의 오류 객체(ValidationError)를 다룰 줄 알기

**학습 항목**

- ValidationError 구조 (errors(), json())
    
- 커스텀 예외 메시지 포맷
    
- FastAPI와의 연동 (HTTPException 자동 변환)
    
- 필드 단위/모델 단위 검증 에러 메시지 핸들링
    
**실습**

- 잘못된 입력에 대한 오류 메시지 커스터마이징
    
- 에러를 API JSON 형태로 변환해보는 미니 실습

---

### 8단계: 고급 주제

> 목표: 대형 프로젝트/공용 contracts 수준까지 확장 가능하도록

**학습 항목**

- Generic 모델 (BaseModel\[T\])
    
- 타입 추론 & MyPy 지원
    
- 모델 버전 관리 (v1.UserCreate, v2.UserCreate)
    
- Schema Refactoring / Lint / AutoDoc (pydantic-settings, json_schema())
    
- TypedDict, dataclass interop
    
- 성능 최적화 (pydantic_core)

**실습**

- GenericResponse[T] 만들기 (data, meta 필드)
    
- 버전별 User DTO 관리
    
- model_json_schema() 로 API 문서 자동 생성

---

### 9단계: FastAPI / Beanie 연동 실습

> 목표: 지금 하려는 프로젝트 구조를 완성형으로 체험

**학습 항목**

- FastAPI에서 response_model / request_model 역할
    
- Pydantic 모델을 통한 자동 검증
    
- Beanie Document ↔ DTO 변환
    
- contracts 패키지를 별도 PyPI로 분리하기
    
**실습**

- contracts-hj3415 패키지 완성
    
- FastAPI 엔드포인트에서 DTO ↔ DB 변환
    
- 모델 버전 호환 실험 (contracts==1.1.* → 1.2.*)

---

### 10단계: 정리 + CI 자동검증

**학습 항목**

- 테스트 시 데이터 검증 (pytest + pydantic)
    
- 스키마 일관성 검사 (contracts → 각 서비스)
    
- 타입검사 (mypy, pyright)
    
- 린터 (ruff) + 포매터 (black)
    
- CI에서 Pydantic 모델 자동 테스트
    
**실습**

- pytest로 contracts 검증 테스트 작성
    
- contracts 패키지 → flit publish --repository testpypi
