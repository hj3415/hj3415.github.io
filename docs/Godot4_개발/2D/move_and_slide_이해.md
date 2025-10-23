이건 Godot의 **CharacterBody2D 이동의 핵심 함수**이자, 플랫폼 게임이나 탑다운 게임 모두에서 가장 많이 쓰이는 함수입니다.

이해만 제대로 하면 CharacterBody2D의 90%는 마스터한 거예요.

---

### 1. move_and_slide()란?

> **velocity 벡터를 기반으로 캐릭터를 이동시키고, 충돌이 발생하면 그 방향으로 “슬라이드” 되도록 처리해주는 함수.**

즉,

- 벽이나 바닥에 부딪혀도 “밀리지 않고 미끄러지듯 이동” 
- 중력, 경사, 점프, 충돌 모두 한 줄로 처리

```
move_and_slide()
```

이거 하나로 **물리 충돌 기반 이동 로직**이 끝납니다.

(예전 Godot 3의 move_and_slide(velocity, up_direction)과 유사하지만 더 자동화됨)

---

### 2. 기본 동작 순서

1. velocity에 따라 캐릭터를 이동시킴
2. 충돌 감지 시, 이동 벡터를 충돌면에 따라 슬라이드시킴
3. 그 결과 새로운 위치를 적용하고,
4. is_on_floor(), is_on_wall(), is_on_ceiling() 같은 충돌 상태를 자동 계산

---

### 3. 기본 사용법

```gdscript
extends CharacterBody2D

func _physics_process(delta):
    velocity.y += 1200 * delta  # 중력
    var dir = Input.get_axis("move_left", "move_right")
    velocity.x = dir * 300
    move_and_slide()
```

이 코드만으로 다음이 자동으로 됩니다:

- 이동 
- 중력 낙하 
- 충돌 감지 
- 벽 밀림 방지 
- 바닥 감지 (is_on_floor())

---

### 4. 내부 원리 간단히 이해하기

move_and_slide()는 “이동 + 충돌 후 벡터 반사(slide)”를 자동으로 계산합니다.

예시:

- 벽에 오른쪽으로 부딪힘 → 이동 벡터의 x 성분을 0으로 줄이고 y만 유지 
- 바닥에 닿음 → 중력 방향으로 더 이상 내려가지 않게 처리
- 경사면 → 충돌면(normal)에 따라 벡터를 슬라이드 시켜 부드럽게 따라감

> 즉, “캐릭터가 벽이나 경사에 부딪혀도 자연스럽게 미끄러지는 효과”를 만들어주는 함수예요.

---

### 5. 주요 속성과 함수 관계

CharacterBody2D는 내부적으로 이 값들을 갱신합니다.

|**속성/함수**|**설명**|
|---|---|
|velocity|이동 속도 벡터 (직접 수정)|
|move_and_slide()|이동 및 충돌 계산 수행|
|is_on_floor()|바닥과 닿아 있으면 true|
|is_on_wall()|벽과 닿아 있으면 true|
|is_on_ceiling()|천장과 닿아 있으면 true|
|get_floor_normal()|현재 서 있는 표면의 법선 벡터|
|get_slide_collision_count()|이번 프레임의 충돌 횟수|
|get_slide_collision(i)|i번째 충돌 정보 (위치, 노멀 등)|

---

### 6. 매개변수 설명 (Godot 4.x)

```
move_and_slide()
```

Godot 4에서는 대부분의 인자를 CharacterBody2D 속성에서 읽기 때문에, Godot 3처럼 인자를 직접 전달할 필요가 거의 없습니다.

하지만 내부적으로는 이런 형태로 작동합니다

|**인자**|**설명**|**기본값**|
|---|---|---|
|max_slides|한 프레임에 몇 번 충돌 계산할지|4|
|floor_max_angle|바닥으로 간주할 최대 경사각 (라디안)|0.785 (~45°)|
|stop_on_slope|경사면에서 정지할지 여부|false|
|floor_block_on_wall|벽에 닿을 때 바닥 판단을 막을지|true|

보통은 기본값 그대로 사용하면 충분합니다.

---

### 7. “슬라이드”의 의미

예를 들어,

- 캐릭터가 벽에 오른쪽으로 이동하려고 할 때
    
    → 벽의 법선(normal)은 (–1, 0)
    
    → 이동 벡터의 x 성분이 제거되고, y 성분만 유지됨
    
    → 즉, **벽을 따라 미끄러짐**

시각적으로 이렇게 작동합니다 👇

```
▶ 벽 충돌 전:   velocity = (200, -50)
▶ 충돌면 normal = (-1, 0)
▶ 충돌 후:      velocity = (0, -50)
```

---

### 8. 경사면 처리 (floor snap)

경사에서 떠오르지 않게 하려면 floor_snap_length를 설정하세요

```gdscript
func _ready():
    floor_snap_length = 10.0
    up_direction = Vector2.UP
```

→ 캐릭터가 경사면 위에서 점프하지 않는 한, **지면에 딱 붙어서 따라가게** 됩니다.

---

### 9. 충돌 정보 활용하기

move_and_slide()가 충돌을 감지하면, get_slide_collision_count()로 몇 번 충돌했는지 확인할 수 있습니다.

```gdscript
for i in range(get_slide_collision_count()):
    var collision = get_slide_collision(i)
    print("Collided with:", collision.get_collider(), " normal:", collision.get_normal())
```

→ 벽, 바닥, 적 등과의 충돌을 감지해서 별도 로직을 실행할 수 있습니다.

---

### 10. Floating 모드에서의 차이점

motion_mode = FLOATING일 때는:

- 중력 없음
    
- is_on_floor() 등 항상 false
    
- 슬라이드 계산은 여전히 가능 (벽 따라 미끄러짐)
    
- 탑다운/우주 게임용

```gdscript
var dir = Input.get_vector("move_left", "move_right", "move_up", "move_down")
velocity = dir * SPEED
move_and_slide()
```

---

### 11. 자주 하는 실수

|**실수**|**원인**|**해결**|
|---|---|---|
|캐릭터가 안 움직임|velocity를 갱신 안 함|velocity.x = dir * SPEED 추가|
|중력이 안 먹음|velocity.y += gravity * delta 안 넣음|직접 중력 추가|
|경사면에서 덜컥거림|floor_snap_length = 0|6~16 정도로 설정|
|공중에서 계속 is_on_floor() true|floor_snap_length 너무 큼|적당히 조정|
|점프가 튐|move_and_slide() 전에 velocity.y 수정 안 함|순서 확인|

---

### 12. 요약

|**항목**|**내용**|
|---|---|
|**함수명**|move_and_slide()|
|**역할**|velocity에 따라 이동 + 충돌 시 슬라이드 처리|
|**자동 계산**|바닥·벽·천장 충돌 상태|
|**Gravity/Jump 지원**|Grounded 모드에서 자연스럽게 동작|
|**Floating 모드**|자유 이동, 중력 없음|
|**자주 쓰는 속성**|velocity, floor_snap_length, up_direction|
|**실행 위치**|반드시 _physics_process(delta) 안에서 호출|

---

**한 줄 요약**

> move_and_slide()는 **캐릭터 이동 + 충돌 + 미끄러짐**을 자동으로 처리하는 핵심 물리 함수다.
>    
> velocity만 갱신해주면 나머지는 Godot이 알아서 해준다.
>
> Grounded에서는 중력·점프, Floating에서는 자유 이동이 된다.
    
