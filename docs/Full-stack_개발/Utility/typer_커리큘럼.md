좋아요 👏

지금 “Typer로 CLI를 만들겠다”는 건,

**단순한 스크립트 수준을 넘어서 개발자용 도구(collector, batch, 관리 툴)** 로 진입한다는 뜻이에요.

  

그래서 아래 커리큘럼은 “**기초 → 실전 → 아키텍처화 → 배포**” 순으로 구성했어요.

FastAPI를 이미 다루고 계시니, Typer는 금방 익히실 겁니다.

---

# **🧭 Typer CLI 완전 커리큘럼 (개발자용 실무 중심)**

---

## **1️⃣** 

## **CLI 기본 개념과 표준 도구 이해**

  

> 목표: “CLI란 무엇이며 argparse, click, typer의 차이를 직관적으로 이해하기”

  

**학습 주제**

- CLI (Command Line Interface)의 구조
    
    → 명령(command) / 옵션(option) / 인자(argument)
    
- 파이썬 표준 라이브러리 argparse로 간단한 CLI 만들기
    
- Click vs Typer 개념 비교
    
    → Typer = “타입힌트 기반 Click”
    
- Typer의 설계 철학 (FastAPI 철학과 동일함: 선언적 + 자동 문서화)
    

  

**실습**

```
python cli.py greet --name "홍길동"
python cli.py calc 10 5 --op add
```

---

## **2️⃣** 

## **Typer 기초 문법**

  

> 목표: typer의 명령, 옵션, 인자, 도움말 구조 익히기

  

**학습 주제**

- typer.Typer()의 역할
    
- @app.command() 데코레이터
    
- **인자 (Argument)** vs **옵션 (Option)** 구분
    
- typer.Argument(), typer.Option() 사용법
    
- help=, default=, prompt=, confirmation_prompt=
    
- 자동 도움말(--help) 생성
    
- typer.echo() vs print() 차이 (I/O 일관성)
    
- 명령 함수에서 타입힌트를 통한 자동 변환 (int, float, bool, datetime)
    

  

**실습**

```
@app.command()
def hello(name: str = typer.Argument(..., help="이름을 입력하세요")):
    typer.echo(f"안녕하세요 {name}님!")
```

---

## **3️⃣** 

## **서브 커맨드 구조**

  

> 목표: git처럼 main subcommand 형태 CLI를 구성하기

  

**학습 주제**

- @app.command() 다중 정의
    
- app.add_typer()로 하위 CLI 그룹 연결
    
- 프로젝트 구조화:
    

```
mycli/
  __main__.py
  main.py
  commands/
    collect.py
    clean.py
    report.py
```

-   
    
- CLI를 “플러그인 구조”로 관리하기
    

  

**실습**

```
python -m mycli collect start
python -m mycli clean old --days 10
```

---

## **4️⃣** 

## **고급 옵션 처리**

  

> 목표: CLI 옵션을 유연하게 다루기

  

**학습 주제**

- Option(..., help=""), Option(prompt=True)
    
- --flag/--no-flag (bool 옵션)
    
- typer.Choice() (한정된 값만 허용)
    
- callback으로 옵션 유효성 검증
    
- 환경변수(envvar=)로 기본값 설정
    
- Config 파일(.env, .yaml) 읽어오는 패턴
    
- 상호 의존 옵션 (예: –dry-run vs –confirm)
    

  

**실습**

```
python app.py run --mode dev --env staging
```

---

## **5️⃣** 

## **입출력 / 예외 처리 / 로깅 통합**

  

> 목표: Typer CLI에 loguru, rich 등 실무 로깅·출력 통합

  

**학습 주제**

- loguru와 CLI 통합 (logger.add(sys.stderr))
    
- typer.Abort, typer.Exit(code=1) 로 안전 종료
    
- try/except 대신 typer.BadParameter 사용법
    
- rich.print() or rich.console.Console로 컬러 출력
    
- 프로세스 종료 시 코드 반환값 (sys.exit)
    

  

**실습**

```
try:
    ...
except ValueError:
    typer.echo("잘못된 입력입니다.", err=True)
    raise typer.Exit(code=1)
```

---

## **6️⃣** 

## **외부 모듈 / 서비스 연동**

  

> 목표: CLI 명령이 실제 서비스 로직(DB, API 등)과 연동되도록 구성

  

**학습 주제**

- 서비스 로직(services/) 호출 구조
    
- 비동기 로직(async) 지원 (typer.run(async_main))
    
- 환경설정 로더 (pydantic-settings, dotenv)
    
- 외부 API 호출 (httpx, tenacity)
    
- CLI와 FastAPI 코드 재사용 (공용 services/ 디렉토리)
    

  

**실습**

```
python cli.py fetch users --limit 100
python cli.py send-report --email admin@example.com
```

---

## **7️⃣** 

## **테스트와 유지보수**

  

> 목표: pytest로 CLI를 자동 테스트하고 유지보수성 확보하기

  

**학습 주제**

- CliRunner (Typer의 테스트 헬퍼)
    
- CLI 테스트 패턴 (stdout 검사, exit code 확인)
    
- mocker(monkeypatch)로 외부 함수 모킹
    
- CLI 출력 캡처(capsys)
    
- pre-commit 훅으로 CLI lint/test 자동화
    

  

**실습**

```
from typer.testing import CliRunner
from app import app

runner = CliRunner()

def test_hello():
    result = runner.invoke(app, ["hello", "홍길동"])
    assert "홍길동" in result.stdout
    assert result.exit_code == 0
```

---

## **8️⃣** 

## **배포 및 실행 패키징**

  

> 목표: CLI를 설치 가능한 명령어로 배포하기

  

**학습 주제**

- pyproject.toml에 entry point 설정 ([tool.poetry.scripts])
    
- pip install . 후 mycli 명령어로 실행되게 하기
    
- PyPI 배포 시 typer CLI 자동 등록
    
- --install-completion / --show-completion 으로 셸 자동완성 지원
    

  

**실습**

```
[tool.poetry.scripts]
collector = "collector.main:app"
```

```
$ collector run naver
$ collector clean --days 7
```

---

## **9️⃣** 

## **고급 확장**

  

> 목표: 유지보수 가능한 CLI 프레임워크로 발전

  

**학습 주제**

- 멀티 커맨드 CLI 구성 (collect, export, status 등)
    
- Typer + Rich 통합 인터페이스 (표, 프로그레스바, 로그)
    
- Typer + Loguru + Watchdog + Schedule 조합으로 자동화 CLI
    
- 플러그인 아키텍처 (importlib.metadata.entry_points 활용)
    

---

## **🎯 10️⃣ 최종 프로젝트 실습**

  

> “Collector CLI” 완성 프로젝트

  

**프로젝트 구조 예시**

```
collector/
 ├── main.py                # Typer 진입점
 ├── commands/
 │    ├── collect.py        # 수집 실행
 │    ├── clean.py          # 오래된 로그 정리
 │    └── status.py         # 상태 출력
 ├── services/
 │    ├── http_client.py
 │    └── db_repo.py
 ├── config/
 │    └── settings.py
 ├── logging_setup.py
 ├── pyproject.toml
 └── tests/
      └── test_cli.py
```

**명령어 예시**

```
collector collect naver --limit 100
collector clean --days 10
collector status
```

---

# **✅ 최종 로드맵 정리**

|**단계**|**학습 주제**|**키워드**|
|---|---|---|
|1|CLI 기본 개념|argparse, click, typer 비교|
|2|Typer 기본 문법|Argument, Option, help, echo|
|3|구조화|Subcommand, app.add_typer|
|4|고급 옵션|Choice, envvar, prompt|
|5|예외·로깅 통합|typer.Exit, loguru|
|6|서비스 연동|httpx, dotenv, 비동기|
|7|테스트|pytest, CliRunner|
|8|배포|entry_points, pyproject.toml|
|9|확장|rich, schedule, watchdog 연동|
|10|실전 프로젝트|Collector CLI 완성|

---

💡 **한 줄 요약**

  

> Typer CLI는 “FastAPI의 CLI 버전”입니다.

1. > 기본 구조 익히고 → 2) 옵션/서브커맨드 → 3) 로깅·테스트 → 4) 배포로 가면
    
    > 실전급 자동화 CLI를 빠르고 깔끔하게 만들 수 있습니다.
    

---

원하신다면 🔧

위 커리큘럼의 **3단계 이후 실전 파트(Collector CLI)** 예제 구조를

직접 코드 템플릿으로 구성해드릴까요? (폴더/파일 전체 포함)