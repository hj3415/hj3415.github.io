좋습니다! 

monorepo 어디서나 공통으로 가져다 쓸 수 있는 
**Pydantic v2 기반 전역 설정 모듈**을 만들어드릴게요.

파일 이름: config_hj3415.py

---

#### 의존성

pydantic>=2와 함께 **pydantic-settings**가 필요합니다.

```
# pyproject.toml
[project.dependencies]
pydantic>=2.5
pydantic-settings>=2.2
```

.env와 시스템 환경변수 모두를 읽고, **타입 변환·검증까지 자동**으로 처리합니다.

(우선순위: **시스템 환경변수 > .env 값**)

---

#### config_hj3415.py

```python
# config_hj3415.py
from __future__ import annotations

from functools import lru_cache
from typing import Literal

from pydantic import Field, ValidationError, field_validator, computed_field
from pydantic_settings import BaseSettings, SettingsConfigDict


EnvName = Literal["dev", "test", "prod"]
LogLevel = Literal["DEBUG", "INFO", "WARNING", "ERROR"]


class Settings(BaseSettings):
    """
    Monorepo 전역 설정.
    - .env와 OS 환경변수를 모두 읽고 타입 캐스팅/검증을 수행합니다.
    - 우선순위: OS 환경변수 > .env
    - 신규 키를 추가할 때는 여기에 필드를 선언하세요.
    """

    # ─────────────────────────────────────────────
    # 기본 메타/환경
    # ─────────────────────────────────────────────
    APP_ENV: EnvName = "dev"
    LOG_LEVEL: LogLevel = "INFO"

    # ─────────────────────────────────────────────
    # Mongo / Beanie
    # ─────────────────────────────────────────────
    MONGO_URI: str = "mongodb://127.0.0.1:27017"
    MONGO_DB: str = "nfs_db"

    # ─────────────────────────────────────────────
    # Scraper / Playwright
    # ─────────────────────────────────────────────
    SCRAPER_HEADLESS: bool = True                      # .env 에 "true"/"false"여도 자동 변환됨
    SCRAPER_CONCURRENCY: int = 2                       # 내부 source의 동시성 기본값
    SCRAPER_SINK_CHUNK: int = 1000                     # 배치 저장 청크 크기
    PLAYWRIGHT_LAUNCH_TIMEOUT_MS: int = 180_000        # Playwright launch timeout
    REQUEST_TIMEOUT_S: float = 30.0                    # HTTP client timeout
    PROXY_URL: str | None = None                       # 필요 시 프록시 (예: http://user:pass@host:port)

    # ─────────────────────────────────────────────
    # 기타 (예: CORS/호스트 화이트리스트 등)
    # ─────────────────────────────────────────────
    ALLOWED_HOSTS: list[str] = Field(default_factory=list)  # "a.com,b.com" → ["a.com","b.com"]

    # Pydantic Settings 구성
    model_config = SettingsConfigDict(
        env_file=".env",                # .env 자동 로드
        env_file_encoding="utf-8",
        case_sensitive=False,           # 환경변수 대소문자 비구분
        extra="ignore",                 # 선언되지 않은 키는 무시
    )

    # ─────────────────────────────────────────────
    # 유효성 검사 / 변환기
    # ─────────────────────────────────────────────
    @field_validator("MONGO_URI")
    @classmethod
    def _validate_mongo_uri(cls, v: str) -> str:
        # AnyUrl을 쓰면 mongodb 스킴 커스텀 설정이 번거로워서 간단 체크로 대체
        if not (v.startswith("mongodb://") or v.startswith("mongodb+srv://")):
            raise ValueError("MONGO_URI must start with 'mongodb://' or 'mongodb+srv://'")
        return v

    @field_validator("SCRAPER_CONCURRENCY", "SCRAPER_SINK_CHUNK", "PLAYWRIGHT_LAUNCH_TIMEOUT_MS")
    @classmethod
    def _must_be_positive(cls, v: int) -> int:
        if v <= 0:
            raise ValueError("Value must be a positive integer.")
        return v

    @field_validator("REQUEST_TIMEOUT_S")
    @classmethod
    def _timeout_positive(cls, v: float) -> float:
        if v <= 0:
            raise ValueError("REQUEST_TIMEOUT_S must be > 0.")
        return v

    @field_validator("ALLOWED_HOSTS", mode="before")
    @classmethod
    def _coerce_allowed_hosts(cls, v):
        """
        ALLOWED_HOSTS가 문자열("a.com,b.com")로 들어와도 리스트로 캐스팅.
        이미 리스트면 그대로 통과.
        """
        if v is None or v == "":
            return []
        if isinstance(v, str):
            return [s.strip() for s in v.split(",") if s.strip()]
        return v

    # ─────────────────────────────────────────────
    # 파생 속성
    # ─────────────────────────────────────────────
    @computed_field
    @property
    def IS_PROD(self) -> bool:  # noqa: N802 (상수 스타일로 노출)
        return self.APP_ENV == "prod"

    @computed_field
    @property
    def IS_DEV(self) -> bool:
        return self.APP_ENV == "dev"

    @computed_field
    @property
    def IS_TEST(self) -> bool:
        return self.APP_ENV == "test"


@lru_cache(maxsize=1)
def get_settings() -> Settings:
    """
    Settings 싱글톤 팩토리.
    - 최초 한 번만 로드하여 캐시합니다.
    - 테스트 등에서 갱신이 필요하면 reset_settings_cache()를 호출하세요.
    """
    try:
        return Settings()
    except ValidationError as e:
        # 설정 값 유효성 실패 시, 에러를 명확히 노출
        # (필요 시 여기에서 로깅 시스템 접목)
        raise


def reset_settings_cache() -> None:
    """
    Settings lru_cache 초기화 (테스트 시 유용).
    """
    get_settings.cache_clear()


# 편의용 즉시 인스턴스
settings = get_settings()

__all__ = ["Settings", "get_settings", "reset_settings_cache", "settings"]
```

---

#### 사용 예시

```python
# 예) Beanie 초기화
from config_hj3415 import settings
from pymongo import AsyncMongoClient
from beanie import init_beanie
from db2_hj3415.adapters.mongo.c1034_models import (
    DimAccountDoc, DimPeriodDoc, FactFinanceDoc, DeltaFinanceDoc
)

async def init_beanie_from_settings():
    client = AsyncMongoClient(settings.MONGO_URI)
    await init_beanie(
        database=client[settings.MONGO_DB],
        document_models=[DimAccountDoc, DimPeriodDoc, FactFinanceDoc, DeltaFinanceDoc],
    )
    return client
```

```python
# 예) Playwright 실행
from config_hj3415 import settings
from scraper2_hj3415.adapters.clients.browser import PlaywrightSession

async def open_browser():
    session = await PlaywrightSession.create(headless=settings.SCRAPER_HEADLESS)
    return session
```

```python
# 예) Typer CLI 내부
import typer
from config_hj3415 import settings

app = typer.Typer()

@app.command()
def show_config():
    # 보안상 민감정보는 제외하고 출력할 것을 권장
    safe = settings.model_dump(exclude={"PROXY_URL"})
    typer.echo(safe)
```

---

#### .env 예시

```
# .env
APP_ENV=dev
LOG_LEVEL=DEBUG

MONGO_URI="mongodb://192.168.100.172:27017"
MONGO_DB="nfs_db"

SCRAPER_HEADLESS=true
SCRAPER_CONCURRENCY=2
SCRAPER_SINK_CHUNK=1000
PLAYWRIGHT_LAUNCH_TIMEOUT_MS=180000
REQUEST_TIMEOUT_S=30

# 콤마 구분 리스트
ALLOWED_HOSTS=localhost,127.0.0.1
```

---

#### 포인트 정리

- **타입 안정성**: .env는 문자열이라도, Pydantic이 bool/int/float/list로 **자동 변환**합니다.
    
- **검증**: 잘못된 값(음수, 형식 오류 등)은 시작 단계에서 바로 에러로 잡습니다.
    
- **일관성**: 전 서비스/CLI에서 **한 곳(****config_hj3415.py****)만** import해서 사용.
    
- **테스트 친화**: reset_settings_cache()로 캐시 리셋 → 환경변수 바꿔가며 테스트 쉬움.
