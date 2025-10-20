M4에서는 우리 우주선 스프라이트가 위에서 내려다보는 시점이었기 때문에, 움직이는 방향에 맞춰 노드의 회전을 변경할 수 있었습니다.

![060_ship_in_space 3.mp4](videos/060_ship_in_space%203.mp4)

하지만 여기서 우리의 캐릭터는 사선(¾) 시점에서 보이기 때문에 스프라이트를 회전시킬 수는 없습니다.
아티스트가 이미 캐릭터가 여러 방향을 바라보는 이미지를 몇 장 제공해주었기 때문에, 코드에서 이동 방향에 맞춰 스프라이트의 텍스처 이미지를 변경해줄 예정입니다.

![020_075_runner_directions.webp](images/020_075_runner_directions.webp)

그래서 이번 강의에서는 플레이어가 누르는 키에 따라 캐릭터가 다른 방향을 바라보도록 만들겠습니다. 이를 구현하는 여러 가지 기법을 설명해드릴 예정이며, 먼저 간단한 if와 elif 문부터 시작해서 GDScript의 match 키워드를 사용하는 방법까지 차례대로 안내해드리겠습니다.

![030_character_010_runner_directions.mp4](videos/030_character_010_runner_directions.mp4)

---

## P1 The straightforward approach

캐릭터의 방향에 따라 스프라이트를 선택하는 데에는 매우 간단한 방법이 있습니다. 바로 일련의 if와 elif 문을 사용하는 것입니다.
캐릭터가 움직이는 방향을 확인한 후, 그에 따라 스프라이트를 변경해주면 됩니다.

좋습니다, 이제 시작해보겠습니다!

먼저 코드에서 표시 이미지를 변경하기 위해 Skin 노드에 대한 참조가 필요합니다. 씬 독(Scene Dock)에서 해당 노드를 우클릭한 후
**Access as Unique Name**을 선택하여 해당 노드에 씬 고유 이름을 지정해줍니다.

그 다음, Skin 노드를 선택한 상태에서 **Ctrl 키(Mac에서는 ⌘ 키)**를 누른 채로 스크립트 창으로 드래그하면, skin이라는 이름의
onready 변수로 자동 삽입됩니다.

### 의사(private) 변수

이제 Godot 커뮤니티에서 자주 사용하는 새로운 관례를 소개해드릴 때가 된 것 같습니다. 바로 **의사-프라이빗 변수(pseudo-private variables)**
입니다. skin 변수를 \_skin으로 이름을 바꿔주세요. 그러면 다음과 같은 코드가 됩니다:

```
@onready var _skin: Sprite2D = %Skin
```

우리는 속성 이름 앞에 밑줄(\_)을 붙여 이 속성이 **우리 스크립트 내부용** 이며, 다른 스크립트에서 접근해서는 안 된다는 것을 나타냅니다.
이는 Python과 같은 프로그래밍 언어에서 사용하는 표준적인 관례입니다.

이러한 변수를 **“의사-프라이빗(pseudo-private)” 변수**라고 부릅니다. 그런데 왜 “의사(pseudo)“일까요? C++ 같은 일부 다른 언어에서는
속성을 [private](../../Glossary/Private_Public_Properties.md)으로 명시하면, 다른 개발자가 그 속성에 접근하는 것을 
**엄격히 막을 수 있습니다**. 그런 언어에서는 코드의 다른 부분에서 private 속성에 접근하려 하면 오류가 발생합니다.

하지만 Python이나 GDScript 같은 언어에서는 앞에 밑줄이 붙은 변수도 코드의 다른 부분에서 여전히 **에러 없이 접근이 가능합니다**.
그렇기 때문에 엄밀한 의미의 private은 아니며, 단지 관례일 뿐입니다. 그래도 이런 변수들은 자동 완성 목록에 표시되지 않기 때문에, 
내부용임을 암묵적으로 구분하기에 좋은 방식입니다.

>[!info]- 변수를 왜 private(비공개)로 유지해야 할까요?
>이는 **코드에서 의도하지 않은 사용이나 변경을 방지하는 데 도움이 되기 때문**입니다.
>
>코드를 나중에 쉽게 변경할 수 있도록 하기 위한 일반적인 지침 중 하나는, **명확한 목적을 가진 코드 사용을 위해 [API(Application Programming Interface)](../../Glossary/API_Application_Programming_Interface.md)를 제공하고, 다른 부분에 노출할 의도가 없는 요소는 사용하지 못하도록 막는 것**입니다. 이러한 방식은 코드의 안정성과 유지보수성을 높여줍니다.
>
>다른 스크립트에서 여러분의 스크립트를 참조하는 팀원이 **의도하지 않은 요소를 사용해서 프로그램을 망가뜨릴 위험을 걱정하지 않아도 되도록** 해야 합니다. 그래서 개발자들은 객체의 내부 구조나 코드를 **숨기거나 보호** 하는 경우가 많습니다. 이것을 [Encapsulation.md](../../Glossary/Encapsulation.md) 라고 합니다.
>
>자동차를 운전할 때, 우리는 **엔진이 어떻게 작동하는지를 몰라도 핸들과 가속 페달을 사용할 수 있습니다.** 캡슐화는 이러한 **“블랙 박스” 원칙** 을 지향합니다. 즉, **다른 사람들이 사용하길 원하는 속성만 공개하고, 나머지는 숨기는 방식** 입니다.
>
>대기업에서는 **원치 않는 변경을 방지하기 위해** 이러한 규율이 존재합니다. 실제로는 어떤 코드를 최대한 잘 활용하려면 **그 코드가 어떻게 동작하는지 어느 정도는 알아야** 할 때가 많습니다. 하지만 그럼에도 불구하고, **의도되지 않은 접근을 줄이고, 어떤 부분의 코드를 사용해야 하는지를 명확히 전달하는 것은 좋은 습관**입니다.
>
>예를 들어, 몇 달 후에 어떤 변수가 더 이상 필요 없어져서 삭제하고 싶다고 가정해보겠습니다. 그 변수에 지금은 규모가 커진 프로젝트 내의 **다른 스크립트가 접근하고 있는지 어떻게 확인할 수 있을까요?** “의사-프라이빗(pseudo-private)” 관례를 따랐다면, 그 변수는 **오직 하나의 스크립트**—즉, 그 변수를 정의한 스크립트에서만 접근하고 있다는 것을 알 수 있습니다. 다른 곳에서 그 변수를 사용하지 않고 있다면, **안전하고 빠르게 삭제할 수 있습니다.**

다음으로, 캐릭터가 여러 방향을 바라보는 텍스처들을 모두 미리 불러올 수 있습니다. res://assets 디렉토리에서 runner_\*.png
파일들(runner_up.png, runner_down_left.png, runner_down_right.png, … runner_up_right.png까지)을 Shift 키를 누른 채로
모두 선택해 주세요. 그런 다음 Ctrl 키(맥에서는 ⌘ 키)를 누른 상태로 이 파일들을 runner 스크립트로 끌어다 놓으시면 됩니다. 이렇게 하면 상수들이
자동으로 생성되고, 모든 텍스처가 한 번에 미리 불러와집니다.

```gdscript
const RUNNER_DOWN = preload("res://assets/runner_down.png")
const RUNNER_DOWN_RIGHT = preload("res://assets/runner_down_right.png")
const RUNNER_RIGHT = preload("res://assets/runner_right.png")
const RUNNER_UP = preload("res://assets/runner_up.png")
const RUNNER_UP_RIGHT = preload("res://assets/runner_up_right.png")
const RUNNER_DOWN_LEFT = preload("res://assets/runner_down_left.png")
const RUNNER_LEFT = preload("res://assets/runner_left.png")
const RUNNER_UP_LEFT = preload("res://assets/runner_up_left.png")
```

변하지 않는 값을 사용할 때는, 이를 [Constant](../../Glossary/Constant.md)로 선언해 두는 것이 유용합니다.

이제 캐릭터의 방향에 따라 스프라이트를 변경하기 위해 if와 elif 조건문을 추가해 보겠습니다. 시작하실 수 있도록 처음 두 방향에 대한 예시를 보여드리겠습니다.
다음 코드를 \_physics_process() 함수 안에 추가해 주세요:

```gdscript
func _physics_process(_delta: float) -> void:
	# ...
	if direction.x > 0.0 and direction.y > 0.0:
		_skin.texture = RUNNER_DOWN_RIGHT
	elif direction.x < 0.0 and direction.y > 0.0:
		_skin.texture = RUNNER_DOWN_LEFT
```

첫 번째 조건문은 캐릭터가 오른쪽 아래 방향에 있는지를 확인합니다. 그렇다면 스프라이트를 RUNNER_DOWN_RIGHT 텍스처로 변경하게 됩니다.

x 방향이 양수이면 캐릭터는 오른쪽으로 이동하고 있는 것입니다. y 방향이 양수이면 캐릭터는 아래쪽으로 이동하고 있는 것입니다. 둘 다 양수라면
캐릭터는 오른쪽 아래 방향으로 이동 중이라는 뜻입니다. 따라서 그에 맞는 텍스처를 스프라이트에 할당할 수 있습니다.

두 번째 조건은 캐릭터가 왼쪽 아래 방향으로 이동하고 있는지를 확인합니다. 그렇다면 스프라이트를 RUNNER_DOWN_LEFT 텍스처로 변경합니다.

이제 캐릭터가 두 방향을 바라볼 수 있게 되었습니다. 직접 테스트해 보시기 바랍니다!

![030_character_015_runner_look_bottom_right_left.mp4](videos/030_character_015_runner_look_bottom_right_left.mp4)

> [!note] 직접 시도해 보세요
> 나머지 여섯 가지 조건—위, 아래, 왼쪽, 오른쪽, 왼쪽 위, 오른쪽 위—를 \_physics_process() 함수에 추가해 보시기 바랍니다.
> 
> 먼저 대각선 방향부터 처리하는 것이 좋습니다. 그렇지 않으면 수평이나 수직 방향의 조건이 대각선 조건보다 우선되어 적용될 수 있습니다.
> 
> 직접 시도해 보시고, 아래에 있는 정답도 참고해 보세요.

다음은 나머지 여섯 가지 방향에 대한 코드입니다. 오른쪽 위와 왼쪽 위 방향에 대한 첫 두 조건과 동일한 패턴을 따릅니다.
나머지 네 방향의 경우에는 방향 벡터의 한 구성 요소만 확인하면 됩니다.

```gdscript
func _physics_process(_delta: float) -> void:
	# ...
	elif direction.x > 0.0 and direction.y < 0.0:
		_skin.texture = RUNNER_UP_RIGHT
	elif direction.x < 0.0 and direction.y < 0.0:
		_skin.texture = RUNNER_UP_LEFT
	elif direction.x > 0.0:
		_skin.texture = RUNNER_RIGHT
	elif direction.x < 0.0:
		_skin.texture = RUNNER_LEFT
	elif direction.y > 0.0:
		_skin.texture = RUNNER_DOWN
	elif direction.y < 0.0:
		_skin.texture = RUNNER_UP
```

이제 캐릭터가 이동할 때마다 방향에 맞게 스프라이트가 바뀌는 모습을 확인하실 수 있을 겁니다. 간단하고 직관적이며, 현재 문제를 잘 해결해 줍니다.
다만 아쉬운 점이 있다면, 조건문의 순서를 주의해서 작성해야 한다는 것입니다. 그래도 잘 작동하고, 코드도 이해하기 쉬운 구조입니다.

![030_character_010_runner_directions 1.mp4](videos/030_character_010_runner_directions%201.mp4)

좋은 수업이었습니다. 뭐라구요, 너무 짧았다고요? 더 원하시나요? 알겠습니다, 그럼 좀 더 흥미롭게 만들어 보도록 하죠.