
model_dump() 와 model_dump_json() 은 **옵션을 아무것도 주지 않았을 때의 “기본 동작”**이 다소 헷갈릴 수 있습니다.

### 1. model_dump() 의 기본 동작

model.model_dump()

**기본값 (옵션 미지정 시)**

|   |   |   |
|---|---|---|
|옵션명|기본값|의미|
|include|None|모든 필드 포함|
|exclude|None|제외 없음|
|exclude_unset|False|설정되지 않은(unset) 필드도 포함|
|exclude_defaults|False|기본값(default)과 같은 필드도 포함|
|exclude_none|False|값이 None인 필드도 포함|
|mode|"python"|Python 객체 그대로 직렬화 (datetime, UUID 그대로 유지)|
|round_trip|False|복원용 metadata는 포함하지 않음|

즉, 아무 옵션 없이 쓰면 모든 필드가 전부 들어간 일반적인 dict 이 나옵니다.

### 2. model_dump_json() 의 기본 동작

model.model_dump_json()

**기본값 (옵션 미지정 시)**

|   |   |   |
|---|---|---|
|옵션명|기본값|의미|
|include|None|모든 필드 포함|
|exclude|None|제외 없음|
|exclude_unset|False|모든 필드 포함|
|exclude_defaults|False|모든 필드 포함|
|exclude_none|False|None도 포함|
|by_alias|False|alias 대신 원래 필드명 사용|
|mode|"json"|JSON 변환-friendly 모드 (datetime → ISO 문자열 등)|
|indent|None|압축된 한 줄 JSON|
|round_trip|False|metadata 포함 안 함|

즉, model_dump_json() 은 모든 필드 포함, 단 datetime/UUID 등은 JSON 직렬화된 문자열로 변환됩니다.

