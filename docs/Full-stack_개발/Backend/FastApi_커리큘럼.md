좋아요! “FastAPI + MongoDB + Vue 3”를 전제로, 개념→실습→프로젝트 흐름으로 차근차근 올라가는 커리큘럼을 제안드립니다. (파이썬·장고 경험자 기준, 4–6주 로드맵)

⸻

0주차(준비) — 개발 환경/기본기

목표: 비동기 기초, 도구 세팅, 프로젝트 뼈대 만들기
	•	파이썬 비동기 기초: async/await, 이벤트 루프, I/O 바운드 vs CPU 바운드, asyncio.gather
	•	FastAPI/Starlette 개념 지도: ASGI, 라우팅, 의존성 주입(DI), 미들웨어, lifespan
	•	Pydantic v2 핵심: BaseModel, field_validator, model_validate, pydantic-settings로 환경변수 관리
	•	도구:
	•	포매터/린터: Ruff+Black, 타입체크: MyPy
	•	테스트: pytest, pytest-asyncio, 클라이언트: httpx.AsyncClient
	•	서버: uvicorn[standard]
	•	리포 구조(권장):

app/
  main.py
  core/        # config, security, logging
  api/         # routes, deps
    routes/
  db/          # mongo/motor or odm(beanie) init
  models/      # ODM 모델 (Beanie/ODMantic 사용 시)
  schemas/     # Pydantic 모델 (입출력)
  services/    # 비즈 로직
  repositories/# DB 접근 계층(Driver 직접 쓸 때)
tests/



미니 실습 A
	•	“헬스 체크” 엔드포인트 /health (DB 연결 상태 포함)
	•	CORS 설정(Vue3 개발 서버 도메인 허용)

⸻

1주차 — FastAPI 기본기 & 라우팅/검증

개념
	•	경로/쿼리/바디/헤더/쿠키 파라미터
	•	요청/응답 모델(검증, 직렬화), response_model, status_code
	•	의존성 주입(DI) 패턴: 공통 파라미터, DB 세션 주입, 보안 의존성
	•	예외 처리: HTTPException, 전역 핸들러, 커스텀 에러 모델
	•	미들웨어/라이프사이클: lifespan에서 DB 연결/종료

미니 실습 B
	•	“메모” CRUD (메모: title, content, tags, created_at) — 아직은 인메모리/목 DB로

⸻

2주차 — MongoDB 연동 (두 가지 트랙 중 택1)

트랙 2A: Driver 직결 (Motor)
	•	motor.motor_asyncio.AsyncIOMotorClient
	•	컬렉션 핸들, 인덱스 설계, ObjectId ↔ 문자열 변환, Pydantic 스키마 매핑
	•	리포지토리 계층으로 쿼리 캡슐화
	•	트랜잭션(복제셋 환경), 부분 업데이트, $set, $push, $lookup(집계)

트랙 2B: ODM (Beanie 권장)
	•	beanie.Document로 모델 정의, init_beanie, 인덱스/밸리데이션
	•	find, find_one, insert, update_many, Aggregation
	•	ODM 장점: 타입 안정성/간결성, 단점: 고급 쿼리 제약·추상화 학습 필요

미니 실습 C
	•	1주차 “메모”를 실제 MongoDB로 이관
	•	페이지네이션(커서 기반 or skip/limit), 정렬, 간단 텍스트 검색(인덱스 포함)
	•	컬렉션 인덱스 실제 생성/확인

⸻

3주차 — 인증/인가 & 보안

개념
	•	OAuth2 Password Flow + JWT(Access/Refresh)
	•	토큰 저장 전략
	•	권장: Access 토큰은 메모리/HTTP 헤더, Refresh 토큰은 HttpOnly+Secure 쿠키 (도메인 분리시 CORS+CSRF 고려)
	•	대안(학습용): 로컬스토리지(보안 리스크 주지)
	•	패스워드 해시(passlib[bcrypt]), 레이트 리밋(SlowAPI), CORS/CSRF 개념
	•	권한(roles/permissions)과 의존성으로 가드

미니 실습 D
	•	POST /auth/login, POST /auth/refresh, POST /auth/logout
	•	GET /me 보호 라우트, 역할 기반 접근 제어

⸻

4주차 — 실전 기능 묶음

파일 업로드/다운로드
	•	이미지 업로드, S3나 로컬 스토리지 어댑터 패턴
백그라운드 작업
	•	BackgroundTasks, 큐(ARQ or RQ/Celery) 비교
실시간/알림
	•	WebSocket(채팅/알림), 서버 sent events(간단)
캐싱/성능
	•	Redis 캐싱, ETag/Last-Modified, 응답 압축, Pydantic model_dump(exclude_none=...)

미니 실습 E
	•	썸네일 생성(백그라운드) + 업로드 상태 폴링(or WebSocket 알림)

⸻

5주차 — 테스트/품질/관측성

테스트
	•	단위/통합 테스트, pytest-asyncio + httpx.AsyncClient
	•	Mongo 테스트: 임시 DB 또는 Testcontainers
품질/로그
	•	구조화 로그(JSON), 요청/응답 로그 마스킹
	•	OpenAPI 스키마 커스터마이즈, 예제(example) 제공
문서화
	•	자동 문서 /docs//redoc 커스터마이즈, 버전 전략(v1/v2)

미니 실습 F
	•	인증 흐름/CRUD 통합 테스트 세트 구축(CI에서 실행)

⸻

6주차 — 배포/운영

배포
	•	Dockerfile (멀티스테이지) + docker-compose(Mongo/Redis 포함)
	•	Uvicorn 단독 vs Gunicorn+UvicornWorker
	•	역프록시(Nginx) + TLS, 헬스체크, Readiness/Liveness
설정/비밀관리
	•	pydantic-settings, .env, 프로파일별 설정(dev/stage/prod)
모니터링
	•	OpenTelemetry(추적), 프로메테우스 지표, 에러 수집(Sentry)

미니 실습 G
	•	docker-compose로 “백엔드+Mongo+Redis” 로컬 프로덕션 흉내
	•	Nginx로 Vue 정적 호스팅 + API 리버스 프록시

⸻

Vue 3 연동 핵심(동시에 진행)
	•	CORS: allow_origins=['http://localhost:5173'], allow_credentials=True
	•	인증 인터셉터(axios):
	•	401 시 refresh API 자동 호출→재시도
	•	쿠키 기반일 경우 withCredentials: true
	•	상태관리(Pinia): authStore(토큰/프로필), uiStore(스피너/알림)
	•	에러 처리: 에러 포맷 통일(백엔드에서 에러 모델 고정)

⸻

코드 스니펫(핵심 부분만)

1) app/main.py

from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.db.mongo import connect_to_mongo, close_mongo
from app.api.routes import router as api_router

@asynccontextmanager
async def lifespan(app: FastAPI):
    await connect_to_mongo()
    yield
    await close_mongo()

app = FastAPI(title="FastAPI+Mongo Starter", lifespan=lifespan)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(api_router, prefix="/api")

@app.get("/health")
async def health():
    from app.db.mongo import db
    ok = await db.command("ping")
    return {"status": "ok", "mongo": ok.get("ok", 0) == 1}

2) app/db/mongo.py (Motor 트랙)

from motor.motor_asyncio import AsyncIOMotorClient
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    MONGO_URL: str = "mongodb://localhost:27017"
    MONGO_DB: str = "myapp"
settings = Settings()

client: AsyncIOMotorClient | None = None
db = None

async def connect_to_mongo():
    global client, db
    client = AsyncIOMotorClient(settings.MONGO_URL)
    db = client[settings.MONGO_DB]
    # 예: 인덱스 보장
    await db["notes"].create_index([("created_at", 1)])

async def close_mongo():
    client and client.close()

3) Pydantic 스키마 & 리포지토리 예시

# app/schemas/note.py
from pydantic import BaseModel, Field
from datetime import datetime

class NoteIn(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    content: str
    tags: list[str] = []

class NoteOut(NoteIn):
    id: str
    created_at: datetime

# app/repositories/note_repo.py
from app.db.mongo import db
from bson import ObjectId
from datetime import datetime

def _id_str(d): 
    d["id"] = str(d.pop("_id"))
    return d

class NoteRepo:
    @staticmethod
    async def create(data: dict) -> dict:
        data["created_at"] = datetime.utcnow()
        res = await db["notes"].insert_one(data)
        doc = await db["notes"].find_one({"_id": res.inserted_id})
        return _id_str(doc)

    @staticmethod
    async def list(skip=0, limit=20) -> list[dict]:
        cursor = db["notes"].find().sort("created_at", -1).skip(skip).limit(limit)
        return [_id_str(d) async for d in cursor]

4) 라우트

# app/api/routes/notes.py
from fastapi import APIRouter, Query
from app.schemas.note import NoteIn, NoteOut
from app.repositories.note_repo import NoteRepo

router = APIRouter(prefix="/notes", tags=["notes"])

@router.post("", response_model=NoteOut, status_code=201)
async def create_note(payload: NoteIn):
    return await NoteRepo.create(payload.model_dump())

@router.get("", response_model=list[NoteOut])
async def list_notes(skip: int = Query(0, ge=0), limit: int = Query(20, le=100)):
    return await NoteRepo.list(skip, limit)

5) 인증(요약)

# app/core/security.py
from datetime import datetime, timedelta
import jwt

SECRET = "change-me"
ALG = "HS256"

def create_access_token(sub: str, ttl_minutes=15):
    now = datetime.utcnow()
    payload = {"sub": sub, "iat": now, "exp": now + timedelta(minutes=ttl_minutes)}
    return jwt.encode(payload, SECRET, algorithm=ALG)

실무에서는 Refresh 토큰은 HttpOnly 쿠키, Access 토큰은 헤더(Authorization: Bearer)로 운용을 권장합니다. 프론트는 axios withCredentials, 백엔드는 CORS allow_credentials=True, 도메인/스킴/포트 정합 체크, CSRF 방어(더블 서브밋 토큰 등)를 병행하세요.

6) 테스트(통합)

# tests/test_notes.py
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_notes_flow():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        r = await ac.post("/api/notes", json={"title":"t","content":"c","tags":[]})
        assert r.status_code == 201
        nid = r.json()["id"]
        r = await ac.get("/api/notes")
        assert any(n["id"] == nid for n in r.json())


⸻

캡스톤 제안(치과 도메인 맞춤)

“치과 예약/차트 경량 백엔드”
	•	엔티티: Patient, Appointment, Procedure, Invoice/Payment
	•	기능: 환자 검색(이름/전화), 일정 캘린더, 시술 코드/수가, 결제 내역, 알림(WebSocket)
	•	보안: 직원 역할(Role), 민감정보 마스킹 로그
	•	프론트(Vue3 + Pinia + Vuetify/Tailwind): 대시보드·달력·환자 카드·청구서 화면

⸻

주차별 체크리스트(요약)
	•	0주차: 비동기·도구·뼈대 OK
	•	1주차: 라우팅/검증/예외/DI OK
	•	2주차: Mongo 연결·인덱스·CRUD·페이지네이션 OK
	•	3주차: 로그인/갱신/권한/레이트리밋 OK
	•	4주차: 업로드·백그라운드·웹소켓·캐싱 OK
	•	5주차: 테스트·문서·로그·에러수집 OK
	•	6주차: Docker/Nginx/TLS·프로파일·모니터링 OK

⸻

다음 단계

원하시면 이 커리큘럼대로 **스타터 레포 구조와 더 자세한 예제(Beanie 트랙 포함 / JWT 쿠키 운용 / axios 인터셉터 샘플 / Dockerfile & compose)**를 바로 만들어드릴게요. 어떤 트랙(Driver vs ODM)을 선호하시는지만 알려주시면 그에 맞춰 템플릿을 드리겠습니다.