## monkeypatch를 꼭 써야하는 경우

말씀하신 대로 mocker.patch()가 대부분의 상황을 커버할 수 있지만,

**“monkeypatch를 꼭 써야만 하는 경우”** 도 명확히 존재합니다.

---

**요약 먼저**

> monkeypatch는 **환경 자체를 바꿔야 할 때** 필수입니다.
>
> 즉, 테스트 중에 “코드 외부의 세계(환경변수, sys.path, import 구조 등)”를 조작해야 할 때는 mocker.patch로는 불가능해요.

---

### 1. 환경변수를 바꿔야 할 때 (setenv, delenv)

이건 대표적인 경우예요

예시:

```python
# app/config.py
import os

def get_db_url():
    return os.environ["DB_URL"]
```

이 함수를 테스트하려면,

환경에 실제 DB_URL이 없을 수도 있죠.

그럴 때 monkeypatch.setenv()가 필요합니다

```python
def test_get_db_url(monkeypatch):
    monkeypatch.setenv("DB_URL", "sqlite://")
    from app.config import get_db_url
    assert get_db_url() == "sqlite://"
```

mocker.patch는 **os.environ 자체를 바꾸지 않습니다.**
(그냥 함수 호출만 모킹하니까요.)

> 환경변수는 OS 수준 전역 상태이므로 monkeypatch의 setenv() / delenv() 만이 안전하고 자동복원되는 방법입니다.

---

### 2. 모듈 import 경로를 바꿔야 할 때 (syspath_prepend)

가짜 모듈이나 임시 파일을 테스트하려면 Python의 모듈 탐색 경로(sys.path)를 바꿔야 할 때가 있습니다.

```python
def test_import_custom_module(monkeypatch, tmp_path):
    fake_dir = tmp_path / "fake_mod"
    fake_dir.mkdir()
    (fake_dir / "my_module.py").write_text("x = 123")

    # sys.path 앞에 가짜 모듈 디렉토리 추가
    monkeypatch.syspath_prepend(str(fake_dir))

    import my_module
    assert my_module.x == 123
```

이건 mocker로 절대 불가능합니다.

(mocker.patch는 코드 내부 속성만 바꿀 수 있지, import 경로는 못 바꿔요.)

---

### 3. 동적으로 import되는 모듈을 제어할 때

예를 들어 코드가 이런 경우

```python
# app/plugins.py
def load_plugin(name):
    mod = __import__(name)
    return mod
```

이럴 때 테스트에서 진짜 모듈을 import하지 않고 싶다면,

가짜 모듈을 만들어야 하죠:

```python
def test_load_plugin(monkeypatch):
    fake_module = type("Fake", (), {"run": lambda: "ok"})
    sys_modules = {"my_plugin": fake_module}
    monkeypatch.setattr(sys, "modules", sys_modules)
    from app.plugins import load_plugin
    assert load_plugin("my_plugin").run() == "ok"
```

mocker.patch는 단일 속성만 바꿀 수 있고

이런 **전체 모듈 import 동작 자체를 가로채는 건 불가능**합니다.

---

### 4. 단순 상수 / 글로벌 변수 값을 직접 바꿔야 할 때

```python
# app/constants.py
DEBUG = False

def is_debug():
    return DEBUG
```

이런 전역 상수는 mocker.patch 로도 가능하긴 하지만,

monkeypatch.setattr() 이 더 직관적이고 간단합니다

```python
def test_debug(monkeypatch):
    import app.constants as const
    monkeypatch.setattr(const, "DEBUG", True)
    assert const.is_debug() is True
```

여기선 mocker도 동작하지만,

이처럼 “단순한 전역 값 교체”에는 monkeypatch가 더 깔끔합니다.

---

### 5. pytest의 fixture 조합 테스트 시 (간단·일시적 변경)**

monkeypatch는 pytest 기본 fixture라서 다른 fixture들과 자연스럽게 조합됩니다.

예:

```python
@pytest.fixture
def with_test_mode(monkeypatch):
    monkeypatch.setenv("MODE", "test")
```

이건 전역 fixture에서 “테스트 모드 환경”을 주입할 때 매우 유용합니다.

(mocker는 fixture에 직접 주입되지 않아요.)

---

### 6. Python 내부 빌트인이나 시스템 전역 객체 교체할 때**

예를 들어 open() 같은 내장 함수 자체를 임시로 바꾸고 싶을 때:

```python
def fake_open(path, mode="r"):
    raise PermissionError("파일 접근 금지")

def test_no_file_access(monkeypatch):
    monkeypatch.setattr("builtins.open", fake_open)
    import app.file_utils
    with pytest.raises(PermissionError):
        app.file_utils.read_file("secret.txt")
```

이건 monkeypatch가 훨씬 간단하고, pytest가 끝나면 자동으로 복구됩니다.

---

**정리 표**

|**상황**|**monkeypatch 필요 여부**|**이유**|
|---|---|---|
|환경변수 바꾸기|✅ 필수|setenv, delenv 전용 기능|
|sys.path / import 구조 변경|✅ 필수|syspath_prepend 전용|
|동적 import 제어|✅ 필수|모듈 전역 수정 가능|
|전역 상수 값 변경|✅ 추천|단순하고 직관적|
|외부 API mock|❌ mocker.patch 권장|호출 추적 가능|
|함수 호출 횟수 검증|❌ mocker.patch 권장|assert_called_* 사용 가능|
|비동기 함수 가짜화|❌ mocker.patch 권장|AsyncMock 지원|

---

## mock.path()를 꼭 써야하는 경우

앞서 monkeypatch는 “**환경 자체를 바꾸는 용도**”라고 했죠.

그렇다면 mock.patch (또는 pytest-mock의 mocker.patch)는 반대로,

> **코드 내부의 함수나 메서드 호출을 “가짜로 바꿔서 동작을 흉내 내야 할 때”** 반드시 써야 합니다.

---

**요약 먼저**

|**구분**|**꼭 써야 하는 이유**|
|---|---|
|**함수 호출을 막고 싶을 때**|진짜 코드 실행 대신 mock 리턴|
|**외부 API 호출을 흉내낼 때**|HTTP 요청/응답을 가짜로 처리|
|**DB 쿼리, 파일 입출력, 이메일 전송 등 “부작용 있는 함수” 테스트 시**|실제로 실행되면 안 되니까|
|**호출 횟수 / 인자 검증을 해야 할 때**|assert_called_* 같은 추적이 필요|
|**비동기 함수 (async def)**|AsyncMock으로 가짜 동작 흉내 가능|
|**클래스 생성자나 메서드 교체할 때**|Class.method 패치로 동작 제어|

---

### 1. 외부 API 호출을 차단할 때 (가장 대표적)

```python
# app/weather.py
import requests

def get_weather(city):
    resp = requests.get(f"https://api.weather.com/{city}")
    return resp.json()
```

이걸 그대로 테스트하면, 테스트 실행할 때마다 **실제 네트워크 요청**을 날리게 되죠

이때 mock.patch 가 필수입니다

```python
from unittest.mock import patch

@patch("app.weather.requests.get")
def test_get_weather(mock_get):
    mock_get.return_value.json.return_value = {"temp": 22}

    from app.weather import get_weather
    result = get_weather("seoul")

    assert result == {"temp": 22}
    mock_get.assert_called_once_with("https://api.weather.com/seoul")
```

- 진짜 HTTP 호출 없이 테스트 통과

- 호출 여부, 인자, 결과 모두 검증 가능

**이런 종류의 “외부 호출이 포함된 코드”는 mock.patch 없이는 테스트 불가능합니다.**

---

### 2. DB / 파일 / 이메일 같은 “부작용 코드”를 막을 때

```python
# app/email_service.py
def send_email(to, msg):
    print(f"Sending mail to {to}: {msg}")
```

이걸 테스트할 때 실제로 콘솔에 프린트하거나 SMTP를 쓰면 곤란하죠.

따라서 mock.patch로 해당 함수를 가짜로 교체합니다

```python
from unittest.mock import patch

@patch("app.email_service.print")
def test_send_email(mock_print):
    from app.email_service import send_email
    send_email("user@test.com", "Hello")

    mock_print.assert_called_once_with("Sending mail to user@test.com: Hello")
```

- 진짜 print 안 찍힘

- 호출된지 추적 가능

---

### 3. 함수 내부의 특정 호출만 막고 싶을 때

예를 들어 이렇게 중첩된 구조가 있다고 합시다

```python
# app/users.py
def get_user_info(id):
    user = fetch_user_from_db(id)
    return {"id": id, "name": user.name}
```

fetch_user_from_db()를 테스트에서 진짜 DB로 부르고 싶지 않다면?

```python
from unittest.mock import patch

@patch("app.users.fetch_user_from_db")
def test_get_user_info(mock_fetch):
    mock_fetch.return_value = type("U", (), {"name": "Alice"})()
    from app.users import get_user_info
    result = get_user_info(1)

    assert result == {"id": 1, "name": "Alice"}
```

fetch_user_from_db()는 호출되지 않고,

가짜 값으로 대체되어 테스트 수행.

**이런 구조는 monkeypatch로도 가능은 하지만,**

**호출 추적까지 필요하면 mock.patch가 훨씬 낫습니다.**

---

### 4. 호출 횟수나 인자까지 검증해야 할 때

```python
mock_fetch.assert_called_once_with(1)
```

- **몇 번 호출됐는지**
    
- **어떤 인자로 호출됐는지**
    
- **순서대로 어떤 호출이 있었는지(call_args_list)**

이건 mock.patch에서만 가능한 기능이에요.

(monkeypatch는 “그냥 교체만” 하고 호출 추적 기능은 없음 ❌)

---

### 5. 비동기 함수(async) 테스트

```python
# app/payment.py
async def request_pay(order_id):
    # 실제 Payple API 호출한다고 가정
    ...
```

이런 경우는 AsyncMock 이 필수예요

```python
from unittest.mock import AsyncMock, patch
import pytest

@pytest.mark.asyncio
@patch("app.payment.request_pay", new_callable=AsyncMock)
async def test_payment(mock_pay):
    mock_pay.return_value = {"status": "OK"}

    from app.payment import request_pay
    result = await request_pay("ORD-123")

    assert result["status"] == "OK"
    mock_pay.assert_awaited_once_with("ORD-123")
```

async/await 기반 코드를 가짜로 돌릴 수 있는 건 mock 패치 계열(mock.patch, pytest-mock)만 가능해요.

---

### 6. 생성자나 클래스 메서드를 가짜로 만들 때

```python
# app/service.py
class UserRepo:
    def save(self, name):
        print(f"save {name}")

def run():
    repo = UserRepo()
    repo.save("Alice")
```

테스트에서 진짜 DB 저장이 일어나면 안 되니까

```python
from unittest.mock import patch

@patch("app.service.UserRepo")
def test_run(mock_repo_cls):
    mock_repo = mock_repo_cls.return_value
    mock_repo.save.return_value = None

    from app.service import run
    run()

    mock_repo.save.assert_called_once_with("Alice")
```

- 클래스 자체를 mock으로 교체

- 내부 메서드 호출 추적

이런 구조는 **mock.patch만 가능**, monkeypatch로는 불가능합니다.

---

### 7. monkeypatch로는 어렵거나 불가능한 케이스 정리

|**상황**|**monkeypatch로 대체 가능?**|**설명**|
|---|---|---|
|외부 API / 네트워크 호출|⚠️ 가능은 하지만 호출 검증은 불가|mock.patch 권장|
|DB 함수 호출 차단 + 인자 검증|❌ 불가|mock.patch 필요|
|async 함수 mock|❌ 불가|AsyncMock 필요|
|클래스나 메서드 호출 추적|❌ 불가|mock.patch 필요|
|함수 호출 횟수/인자 확인|❌ 불가|mock.patch 필요|
|단순한 전역 상수 교체|✅ 가능|monkeypatch.setattr 사용|
|환경변수 설정|✅ monkeypatch 전용|mock.patch 불가|

---

### 결론

> **mock.patch는 “코드 내부의 함수 호출을 제어하고, 호출 자체를 검증해야 할 때” 필수입니다.**

  

즉,

- “이 함수가 불렸는가?”
    
- “몇 번 불렸나?”
    
- “어떤 인자로 불렸나?”
    
- “리턴값을 테스트용으로 바꾸고 싶다”
    
- “외부 API, DB, 파일 호출을 막고 싶다”


이런 게 필요하면 mock.patch / mocker.patch 를 써야 합니다.

  

반면에,

- 환경 변수
    
- sys.path
    
- 단순 속성 변경

같은 건 monkeypatch가 담당합니다.
    
