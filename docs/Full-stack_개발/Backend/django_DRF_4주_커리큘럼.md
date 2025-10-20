장고 MVT를 아시는 파이썬 개발자 분이 **기초부터 DRF로 백엔드 API**를 잡아가는 **4주 커리큘럼**을 드릴게요.

### 주차별 로드맵 개요

    **Week 0. 준비 & REST/HTTP 기초**

    **Week 1. DRF 핵심 ① — Serializer/Generic/ViewSet/Router**

    **Week 2. DRF 핵심 ② — 인증·권한·필터·페이징·문서**

    **Week 3. 실전 품질 — 검증·트랜잭션·파일·성능·테스트·배포 감각**

---

### Week 0 — 준비 & REST/HTTP 기초

**목표**

    - DRF가 해결하는 문제를 이해하고, HTTP/REST 기본을 확실히.
    
    - 개발 환경과 최소 의존성 세팅.

**알아둘 개념**

    - REST 리소스 설계(명사형 URI), 메서드 의미/멱등성(GET/PUT/DELETE vs POST/PATCH)
    
    - 상태코드(200/201/204/400/401/403/404/409), 헤더(Accept/Content-Type/Authorization)
    
    - Django와 DRF의 관계: Django(인증/ORM/템플릿/어드민) + DRF(직렬화/권한/라우팅/문서화)

**실습(30분)**

    - 새 프로젝트 생성 → djangorestframework, django-filter, drf-spectacular, django-cors-headers 설치
    
    - /api/schema/, /api/docs/(Swagger) 노출까지 구성
    
**체크리스트**

    - 같은 요청을 두 번 보내도 안전해야 할 때 왜 “멱등성”이 중요한지 설명할 수 있다.
    
    - 오류를 400/401/403/409로 구분해 말할 수 있다.

---

### Week 1 — DRF 핵심 ①: Serializer · Generic · ViewSet · Router

**목표**

    - **Serializer**로 입력/출력을 다루고, **Generic/ViewSet/Router**로 CRUD를 완성.
    
    - Parsers/Renderers를 이해하고 Browsable API 활용.
    
**알아둘 개념**

    - Serializer vs ModelSerializer, read_only_fields, write_only, SerializerMethodField
    
    - 검증 흐름: 필드 validator → validate_<field> → validate() → .create()/.update()
    
    - APIView vs GenericAPIView + mixins vs ViewSet/ModelViewSet 계층
    
    - DefaultRouter가 list/retrieve/create/update/partial_update/destroy로 라우팅하는 방식
    
    - request.data vs request.query_params, JSON 파서/렌더러
    
**작은 실습 A (블로그)**

    - Post 모델 기준 **ModelViewSet** + DefaultRouter로 /api/posts/ CRUD 구성
    
    - 목록에 search, ordering, filterset_fields 1–2개만 붙여보기
    
**체크리스트**

    - “왜 GET /api/posts/1/이 retrieve 액션으로 매핑되는지” 설명할 수 있다.
    
    - 입력에 포함되면 안 되는 필드를 read_only로 막을 수 있다.
    

---

### Week 2 — DRF 핵심 ②: 인증 · 권한 · 필터 · 페이징 · 문서

**목표**

    - **읽기 공개 / 쓰기 인증** 패턴을 익히고, **객체 단위 권한**을 구현.
    
    - 검색·필터·정렬·페이징을 표준화하고 OpenAPI 문서 자동화.
    
**알아둘 개념**

    - 인증: 세션 vs 토큰(JWT) — DRF의 authentication_classes
    
    - 권한: IsAuthenticated, IsAdminUser, 커스텀 IsOwnerOrReadOnly(객체 단위 권한)
    
    - 스로틀 소개(익명/인증/스코프별)
    
    - django-filter, SearchFilter, OrderingFilter, 페이지네이션(Page/Limit-Offset/Cursor)
    
    - drf-spectacular로 스키마 생성, 예시(example)·태그·응답 모델 지정
    
**작은 실습 B (권한 & 문서)**

    - **읽기 누구나 / 쓰기 로그인만 / 수정·삭제는 작성자만** 규칙 적용
    
    - JWT(SimpleJWT) 발급 엔드포인트 붙여서 토큰으로 쓰기 동작 확인
    
    - /api/docs/에서 스키마와 예시 응답 확인
    
**체크리스트**

    - 401(인증 필요)과 403(권한 없음)을 상황별로 구분해서 반환할 수 있다.
    
    - 목록 API에 검색·정렬·페이징을 일관되게 붙일 수 있다.

---

### Week 3 — 실전 품질: 검증 · 트랜잭션 · 파일 · 성능 · 테스트 · 배포 감각

**목표**

    - 비즈니스 규칙 검증을 Serializer에 담고, **트랜잭션/동시성/멱등성** 감각을 익힘.
    
    - 파일 업로드, N+1 제거, 캐시/조건부 GET, 기본 테스트, 배포 전 체크리스트까지.

**알아둘 개념**

    - 고급 검증: 교차 필드 검사(validate()), 비즈니스 규칙(예: “주목글이면 PUBLISHED여야 함”)
    
    - 트랜잭션: transaction.atomic(), DB 제약(UNIQUE/Check), 동시성(select_for_update) 기본
    
    - 멱등 키(Idempotency-Key) 개념(웹훅/결제 재시도 안전)
    
    - 파일 업로드: 멀티파트, ImageField, 미디어 서빙, S3 기본 개념(사전서명 URL)
    
    - ORM 성능: select_related/prefetch_related, 쿼리 수 점검
    
    - 캐싱/조건부 GET: ETag/If-None-Match, Last-Modified/If-Modified-Since
    
    - 테스트: pytest + APIClient로 권한/검증 최소 시나리오
    
    - 운영 감각: CORS 제한, HTTPS, 보안 헤더, 로깅/에러 수집(Sentry류)
    
**작은 실습 C (파일 & 성능 & 테스트)**

    - 썸네일 업로드 가능하도록 엔드포인트 점검(미디어 경로 포함)
    
    - 목록 조회 시 select_related로 N+1 제거 전/후 쿼리 수 비교
    
    - 테스트 2–3개: “비로그인 생성 금지”, “작성자만 수정”, “검증 실패 시 400”
    
**체크리스트**

    - 중복 요청/재전송이 와도 안전하게 처리할 수 있는 설계(트랜잭션/멱등/락)를 말할 수 있다.
    
    - 이미지 업로드/다운로드 시 권한과 보안 포인트를 짚을 수 있다.
    
    - 최소 테스트로 품질 회귀를 막는 법을 안다.

---

### 학습 진행 팁 (실전 위주)

    - **작게 만들어 바로 확인**: 엔드포인트 하나 추가할 때마다 /api/docs/와 Browsable API로 즉시 검증하세요.
    
    - **권한 매트릭스 표**: “익명/일반/관리자” 행 × “목록/상세/생성/수정/삭제” 열로 허용/금지 표를 먼저 적고 구현하세요.
    
    - **응답 규약 통일**: 성공/오류 바디 스켈레톤을 팀 규약으로 고정하면 클라이언트 생산성이 급상승합니다.
    
    - **N+1 습관적으로 점검**: select_related는 목록 API의 기본값처럼 생각하세요.

---

### 마무리 과제(캡스톤, 1~2일)

    - **블로그 API**: 공개 읽기, 작성자 전용 쓰기, 검색/정렬/페이징, 썸네일 업로드, 간단 테스트 3개.
    
    - **(선택) 예약 API**: 슬롯 생성·예약·취소, “같은 슬롯 동시 요청” 트랜잭션 처리 스케치, 문서화.
