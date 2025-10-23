삼항 연산자는 불리언 값으로 평가되는 조건식입니다. 
“조건식(conditional expression)”, “삼항 연산자(ternary operator)”, 혹은 때로는 “inline if”라고도 불립니다. 
이는 if 문을 간단하게 작성할 수 있는 축약 방식입니다. 작성법은 다음과 같습니다:

```gdscript
var result = value_if_true if condition else value_if_false
```

예를 들어:

```gdscript
var player_expression := "normal" if health > 5 else "hurt"
```

이 예시에서, 변수 health가 5보다 크면 player_expression은 "normal"이 되고, 그렇지 않으면 "hurt"가 됩니다.

삼항 연산자는 결합해서 사용할 수도 있지만, 가독성이 떨어질 수 있습니다. 예를 들어:

```gdscript
var player_expression := "normal" if health > 5 else "hurt" if health > 2 else "dead"
```

보다 가독성을 높이려면 줄을 나누고 괄호를 사용하여 연산 순서를 명확히 하는 것이 좋습니다:

```gdscript
var player_expression := "normal" \
  if health > 5 \
  else \
  ( "hurt" \
    if health > 2 \
    else "dead"
  )
```

이와 같은 경우에는 아마 if 문이나 match 문을 사용하는 것이 더 적절할 수 있습니다. 
하지만 **분기 없는(branchless)** 방식으로 코드를 작성하거나 변수가 항상 값을 가지도록 보장하고자 할 때는 이 스타일이 유용할 수 있습니다.

예를 들어 다음 코드를 보겠습니다:

```gdscript
var user_message = null
if server_response == "ok":
  user_message = "Success! You're connected"
elif server_response == "error":
  user_message = "Error! Couldn't connect"
```

이 경우 서버 응답이 "ok" 또는 "error"가 아닌 다른 값일 때는 user_message가 null로 남게 됩니다. 
삼항 연산자를 사용하면 user_message가 항상 값을 가지도록 보장할 수 있습니다:

```gdscript
var user_message := "Success! You're connected" \
  if server_response == "ok" \
  else "Error! Couldn't connect"
```

> [!info]- 하지만 그 경우에도 아마 **방어적으로 코드를 작성하는 것**이 더 나을 것입니다.
> 위와 같은 문제를 피하는 좀 더 유용한 방법은 다음과 같습니다:
> 
> ```gdscript
> var user_message := "Error! Couldn't connect"
> if server_response == "ok":
> 	user_message = "Success! You're connected"
> ```
> 
> 이렇게 하면 서버 응답이 "ok"도 "error"도 아닐 경우에도 user_message는 여전히 값을 가지게 됩니다.