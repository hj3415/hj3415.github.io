이 다섯 가지 ConfigDict 옵션의 **기본값(default)** 은 Pydantic v2 기준으로 아래와 같습니다.

---

### 각 항목의 기본값 정리표

|**옵션명**|**기본값**|**의미 (기본 동작)**|
|---|---|---|
|from_attributes|False|ORM 객체나 dataclass의 **속성(attribute)** 은 읽지 않음 → **dict 형태 입력만 허용**|
|extra|"ignore"|모델에 정의되지 않은 필드는 **무시하지만 오류는 발생하지 않음**|
|populate_by_name|False|**alias(별칭)으로만 입력 가능**, 내부 필드명으로는 입력 불가|
|validate_assignment|False|모델 인스턴스 생성 이후 **값 변경 시 검증하지 않음**|
|use_enum_values|False|Enum 필드를 **그대로 Enum 객체로 유지**하고, 직렬화 시 .name이 아닌 Enum 자체로 표시|

---

### 예시별 기본 동작 요약

**1. from_attributes=False**

```python
class User(BaseModel):
    name: str

class ORM:
    name = "Alice"

User.model_validate(ORM())  
# ValidationError: "User" expected dict not "ORM"
```

---

**2. extra="ignore"**

```python
class User(BaseModel):
    name: str

User.model_validate({"name": "Alice", "age": 25})
# {'name': 'Alice'} — 'age' 필드는 무시됨
```

---

**3. populate_by_name=False**

```python
class User(BaseModel):
    first_name: str = Field(alias="firstName")

User(firstName="Alice")   # OK
User(first_name="Alice")  # ValidationError (내부 이름 허용 안 함)
```

---

**4. validate_assignment=False**

```python
class Product(BaseModel):
    price: float

p = Product(price=10)
p.price = "bad"    # 허용됨 (재검증 안 함)
print(p.price)     # 'bad' (타입 깨짐)
```

---

**5. use_enum_values=False**

```python
from enum import Enum
class Role(Enum):
    ADMIN = "admin"
    USER = "user"

class Member(BaseModel):
    role: Role

m = Member(role=Role.ADMIN)
print(m.model_dump())  # {'role': <Role.ADMIN: 'admin'>}
```

---

**한 줄 요약**

> 명시하지 않으면 기본 동작은 “**가급적 관대하고, ORM/alias/재검증은 비활성화된 상태**”

즉

```python
model_config = ConfigDict(
    from_attributes=False,
    extra="ignore",
    populate_by_name=False,
    validate_assignment=False,
    use_enum_values=False
)
```

이게 Pydantic BaseModel의 **기본 설정값(Default Config)** 이라고 보면 됩니다.