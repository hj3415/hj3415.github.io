강한 타입을 사용하는 언어에서는, **타입에 따라 서로 다른 요소들을 구분할 수 있습니다.** 
예를 들어 Java에서는 Animal 타입의 객체를 매개변수로 받는 메서드를 만들고, 그 객체가 Dog인지 Cat인지
확인한 뒤 서로 다른 동작을 수행할 수 있습니다.

GDScript에서도 이와 같은 방식이 가능합니다. 만약 Animal이라는 클래스와 이를 상속받은 Dog, Cat이라는 두 개의 하위 클래스가 있다면,
다음과 같이 작성할 수 있습니다:

```gdscript
extends Node

func determine_animal_type(animal: Animal):
	if animal is Dog:
		print("It's a dog!")
	elif animal is Cat:
		print("It's a cat!")
	else:
		print("It's an animal, but we don't know which")
```

하지만 이 방식은 요소들이 **공통된 부모 클래스를 가질 때에만** 동작합니다. 만약 두 클래스가 **공통된 부모를 공유하지 않는다면**,
이 방법은 사용할 수 없습니다. 여기서 **덕 타이핑(duck typing)** 이 등장합니다.

덕 타이핑이란, **변수가 가진 메서드나 속성을 바탕으로 그것이 무엇인지 추론하는 방식** 입니다. 즉, “오리처럼 걷고 오리처럼 꽥꽥거린다면,
그것은 오리다”라는 개념입니다.

이러한 방식은 Godot에서 **적에게 피해를 입히는 데 자주 사용됩니다.** 예를 들어, 어떤 게임에서 Player 클래스와 다양한 적 캐릭터들은 
서로 다른 클래스로부터 파생되었을 수 있습니다. 하지만 이들이 모두 take_damage() 메서드를 가지고 있다면, **객체의 정확한 타입을 몰라도**
해당 메서드를 호출할 수 있습니다. 예를 들어, 아래는 take_damage() 메서드를 가진 BirdEnemy 클래스입니다:

```gdscript
class_name BirdEnemy
extends Area3D

var health := 100

func take_damage(damage: int) -> void:
    health -= damage
    if health <= 0:
        queue_free()
```

그리고 이 총알은 어떤 객체가 take_damage() 메서드를 가지고 있는지를 확인한 후 아래와 같이 처리합니다:

```gdscript
func on_area_entered(target: Area3D) -> void:
  if target.has_method("take_damage"):
    target.take_damage(10)
```

이 코드는 **take_damage() 메서드를 가진 모든 객체가 동일한 [Function_Signature](Function_Signature.md)
를 사용한다는 계약을 존중한다는 전제 하에** 동작합니다.
위의 예시에서는 take_damage()가 **정수(int) 또는 실수(float)** 값을 매개변수로 받는 구조입니다.

객체의 타입을 확인하는 또 다른 방법은 **특정 속성을 추가하는 것**입니다. 예를 들어, Area3D를 기반으로 하는 새 적(BirdEnemy)과
CharacterBody3D를 기반으로 하는 황소 적(BullEnemy)이 있다고 가정해보겠습니다. 이 두 클래스에 is_enemy 속성을 추가할 수 있습니다:

```gdscript
class_name BirdEnemy
extends Area3D

var is_enemy := true
var health := 5

func take_damage(damage: int) -> void:
    health -= damage
```

```gdscript
class_name BullEnemy
extends CharacterBody3D

var is_enemy := true
var health := 15

func take_damage(damage: int) -> void:
    health -= damage
```

그런 다음, 객체에 is_enemy 속성이 있는지를 확인하여 그것이 적인지 판단할 수 있습니다:

```gdscript
func on_area_or_body_entered(target: Node) -> void:
  if target.is_enemy:
    target.take_damage(10)
```

이 방식은 **다른 형태의 계약** 으로, 좀 더 복잡합니다. 즉, **is_enemy 속성을 추가했다면, 반드시 take_damage() 메서드도 함께 추가해야 한다**
는 계약을 전제로 하고 있는 것입니다.

보시다시피, **덕 타이핑은 실용적이긴 하지만 오류가 발생하기 쉬운 방식** 이기도 합니다. 이러한 이유로 GDQuest에서는 가능하다면 
[Composition](Composition.md)을 사용하는 것을 선호합니다. 예를 들어, 피해를 처리하는 HitBox 클래스를 만들 수 있습니다.
이 HitBox는 피해를 받아야 하는 **어떤 객체에든 추가할 수 있으며** , 체력이 0에 도달하면 자신의 부모 노드를 삭제하도록 구성할 수 있습니다:

```gdscript
class_name HitBox
extends Area3D

@export var health := 10

func take_damage(damage: int) -> void:
  health -= damage
  if health <= 0:
    get_parent().queue_free()
```

그런 다음, 피해를 받아야 하는 **어떤 객체에든 HitBox를 추가** 할 수 있습니다. 이렇게 하면 코드가 더 간단해지고, **덕 타이핑을 피할 수 있습니다**:

```gdscript
extends Node

func on_area_entered(target: Area3D):
  if target is HitBox:
    target.take_damage(10)
```
