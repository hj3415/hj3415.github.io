Godot에서 **Input.get_axis()** 와 **Input.get_vector()** 는 둘 다 “입력 방향값”을 구하지만,

**차원이 다르고 (1D vs 2D)** 리턴값의 형태와 정규화 여부가 완전히 다릅니다.

**요약 표**

|**구분**|get_axis()|get_vector()|
|---|---|---|
|입력 개수|2 (neg, pos)|4 (left, right, up, down)|
|반환값|float|Vector2|
|정규화|없음|자동 정규화|
|주요 용도|1D 이동 (좌/우)|2D 이동 (상하좌우, 조이스틱)|
|대각선 처리|직접 normalize 필요|자동 normalize|
|예시|Input.get_axis("left", "right")|Input.get_vector("left","right","up","down")|

---

### “get_vector()가 내부적으로 get_axis() 두 개를 쓰는 거라면, 그냥 get_vector() 하나만 쓰면 되는 거 아닌가?”
 
결론부터 말씀드리면:

> **기술적으로는 대부분의 경우 get_vector() 하나로 통일해도 잘 작동합니다.**
>
> 하지만 **“1D 움직임”과 “2D 움직임”의 목적이 다르기 때문에**, 실제 개발에서는 여전히 둘 다 쓰입니다.

이유를 단계별로 볼게요

---

#### 1. 사실관계:get_vector() =get_axis() × 2 + normalize()

Godot 엔진 내부 구현을 단순화하면 이런 식이에요

```gdscript
func get_vector(neg_x, pos_x, neg_y, pos_y):
    var x = get_axis(neg_x, pos_x)
    var y = get_axis(neg_y, pos_y)
    return Vector2(x, y).normalized()
```

즉, get_vector()는 이미 get_axis() 두 번을 호출한 **상위 함수**가 맞습니다.

그래서 “결과만 필요하다면 get_vector로 통일 가능”해요.

---

#### 2. 하지만 get_vector()는 2D 이동 전제 라서 주의점이 있습니다

**예시: 플랫폼(좌우+점프) 게임**

```gdscript
var dir = Input.get_vector("move_left", "move_right", "jump", "nothing")
```

이렇게 쓸 수는 있지만…

- 점프(jump)는 위쪽 방향이 아니죠
    
- get_vector()는 자동 정규화해서, X/Y를 모두 고려한 벡터로 만들어버립니다.
    
- 즉, **“이동은 1D인데 점프까지 포함시키는 건 비논리적”**이에요.

> 1D 축(좌우)만 필요한 경우에는 get_axis()가 더 직관적이고 안전합니다.

---

#### 3. “왜 굳이 1D 축이 필요할까?”

플랫폼 게임에서는 “x축 이동”과 “y축 중력/점프”가 완전히 다른 논리로 작동합니다.

**예시 (2D 플랫폼)**

```gdscript
var dir_x = Input.get_axis("move_left", "move_right")
velocity.x = dir_x * SPEED

if Input.is_action_just_pressed("jump") and is_on_floor():
    velocity.y = -JUMP_FORCE
```

→ x축만 부드럽게 조정하고, y축은 물리·중력으로 따로 처리하죠.

get_vector()로 하면 이런 분리 제어가 어렵습니다.

---

#### 4. 반대로 “탑다운 2D”에서는 get_vector() 가 완전한 상위호환

**예시 (RPG, 쯔꾸르, 젤다 스타일)**

```gdscript
var dir = Input.get_vector("move_left", "move_right", "move_up", "move_down")
velocity = dir * SPEED
move_and_slide()
```

- 상하좌우를 모두 한 번에 계산
- 대각선 속도 자동 보정 
- 입력이 단일 코드로 끝남

→ 이런 구조에서는 get_vector()가 get_axis()보다 훨씬 깔끔합니다.

---

#### 5. 정리: “언제 get_vector만 써도 되는가?”

|**상황**|**get_vector만으로 충분**|**이유**|
|---|---|---|
|**탑다운 2D (상하좌우 이동)**|✅ 예|완벽히 2D 벡터 입력에 맞음|
|**조이스틱/패드 이동**|✅ 예|스틱 입력 자체가 2D 벡터임|
|**에임 방향(카메라, 마우스, 스틱)**|✅ 예|벡터 방향만 중요|
|**UI 네비게이션(상하좌우)**|✅ 예|단순한 2D 이동 신호|
|**플랫폼 게임(좌우 + 점프)**|⚠️ 아니오|1D 이동 + 별도 점프 논리 필요|
|**비물리 입력(회전, 슬라이더 등)**|⚠️ 아니오|1축만 필요한 상황|

---

#### 6. “get_vector 하나로 통합하고 싶을 때” 팁

만약 프로젝트 전체를 get_vector로 통일하고 싶다면, “Y축 입력이 없는 경우”엔 그냥 무시하면 됩니다.

```gdscript
var move = Input.get_vector("move_left", "move_right", "", "")
velocity.x = move.x * SPEED
```

→ neg_y, pos_y 빈 문자열을 주면 무시됨.

→ 사실상 get_axis("left", "right")와 동일하게 동작합니다.

즉,

> “Y축 없는 get_vector” → “get_axis와 거의 동일”.

---

#### 결론 요약

|**항목**|**get_axis()**|**get_vector()**|
|---|---|---|
|차원|1D|2D|
|반환값|float|Vector2|
|정규화|없음|있음|
|내부 구현|독립 함수|get_axis 2개 + normalize|
|주 용도|플랫폼, 회전, 슬라이더 등|탑다운 이동, 조이스틱 입력|
|상호 관계|하위 기능|상위 래퍼|
