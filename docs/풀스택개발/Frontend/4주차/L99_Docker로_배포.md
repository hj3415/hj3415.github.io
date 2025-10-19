**Django(백엔드) + Vite(Vue 프론트) + Nginx**를 **Docker / Docker Compose**로 구성하는 표준 예시를 드릴게요.

개발용(핫리로드)과 운영용(정적 빌드 + Nginx 서빙) 둘 다 제공합니다.

---

**디렉터리 구조(추천)**

```
repo/
├─ backend/                 # Django
│  ├─ manage.py
│  ├─ requirements.txt
│  ├─ app/...
│  ├─ Dockerfile
│  └─ entrypoint.sh
├─ frontend/                # Vite + Vue
│  ├─ package.json
│  ├─ vite.config.ts
│  ├─ src/...
│  └─ Dockerfile
├─ nginx/
│  ├─ default.conf.template
│  └─ Dockerfile
├─ .env
└─ docker-compose.yml
```

> 운영(배포)에서는 frontend/dist를 Nginx가 정적으로 서빙하고, 개발에서는 Vite dev 서버(5173)와 Django dev 서버(8000)를 각각 컨테이너로 띄웁니다.

---

#### 1. 공통: .env 예시

```dotenv
# 공통
PROJECT_NAME=django-vue

# Django
DJANGO_SETTINGS_MODULE=config.settings
DJANGO_SECRET_KEY=please_change_me
DJANGO_DEBUG=1
DJANGO_ALLOWED_HOSTS=*

# DB (필요시)
POSTGRES_DB=app
POSTGRES_USER=app
POSTGRES_PASSWORD=pass
POSTGRES_HOST=db
POSTGRES_PORT=5432

# 프론트
VITE_PORT=5173
FRONTEND_URL=http://localhost:5173
BACKEND_URL=http://localhost:8000
```

---

#### 2. 개발용 Dockerfile & 설정

**2-1) backend/Dockerfile (dev)**

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# 시스템 의존성 (psycopg 등 필요 시)
RUN apt-get update && apt-get install -y build-essential curl && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 소스 복사
COPY . .

# 권한 및 실행 준비
RUN chmod +x /app/entrypoint.sh

EXPOSE 8000
CMD ["/app/entrypoint.sh"]
```

**backend/entrypoint.sh (dev)**

```shell
#!/usr/bin/env bash
set -e

# 마이그레이션
python manage.py migrate --noinput

# 개발 서버 실행
python manage.py runserver 0.0.0.0:8000
```

**2-2) frontend/Dockerfile (dev)**

```dockerfile
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 5173
# Vite dev server
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "5173"]
```

**2-3) nginx/Dockerfile (dev는 생략 가능)**

개발은 Nginx 없이도 충분합니다(프론트/백엔드 각자 dev 서버).

원하면 리버스 프록시로 붙일 수 있는데, 운영 섹션에서 다룹니다.

**2-4) docker-compose.yml (dev)**

```yaml
version: "3.9"

services:
  backend:
    build:
      context: ./backend
    container_name: backend
    env_file: .env
    volumes:
      - ./backend:/app:cached
    ports:
      - "8000:8000"
    depends_on:
      - db
    # DB 미사용이면 제거
    environment:
      DJANGO_DEBUG: "${DJANGO_DEBUG}"

  frontend:
    build:
      context: ./frontend
    container_name: frontend
    env_file: .env
    volumes:
      - ./frontend:/app:cached
    ports:
      - "${VITE_PORT:-5173}:5173"
    # 백엔드 API 프록시는 vite.config.ts에서 처리 권장

  db:
    image: postgres:16-alpine
    container_name: db
    env_file: .env
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  pgdata:
```

**(참고) frontend/vite.config.ts (dev 프록시)**

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    host: true,
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://backend:8000',
        changeOrigin: true
      }
    }
  }
})
```

> 개발 시에는 http://localhost:5173에서 프론트 열고, API 는 /api/...가 자동으로 backend:8000으로 프록시됩니다.

---

#### 3. 운영용(배포) Dockerfile & 설정

운영에서는:

- **프론트**: npm run build → frontend/dist 산출
- **백엔드**: gunicorn + (원하면 whitenoise 또는 Nginx로 정적 제공)
- **Nginx**: /는 Vue 정적, /api는 Django로 리버스 프록시

**3-1) frontend/Dockerfile (prod, 멀티스테이지)**

```dockerfile
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Nginx stage (정적 파일 서빙)
FROM nginx:1.27-alpine
COPY --from=build /app/dist /usr/share/nginx/html
# Vue history 모드 대비 try_files 설정은 conf에서
COPY ./nginx/default.conf.template /etc/nginx/templates/default.conf.template
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**3-2) backend/Dockerfile (prod)**

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN apt-get update && apt-get install -y build-essential && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt \
    && pip install --no-cache-dir gunicorn

COPY . .

ENV DJANGO_DEBUG=0
RUN python manage.py collectstatic --noinput

EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "config.wsgi:application", "--workers", "3"]
```

> 정적파일을 Django로 서빙하려면 whitenoise 사용 가능. 하지만 **프론트 정적은 Nginx가 서빙**하는 구성이 보편적입니다.

**3-3) nginx/default.conf.template (prod)**

```nginx configuration
server {
    listen 80;
    server_name _;

    # Vue history mode 지원
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # API → Django
    location /api/ {
        proxy_pass http://backend:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
    }

    # (선택) 정적/미디어 파일을 Django 대신 Nginx로
    # location /static/ { alias /app/static/; }
    # location /media/  { alias /app/media/;  }
}
```

**3-4) docker-compose.yml (prod)**

```yaml
version: "3.9"

services:
  backend:
    build:
      context: ./backend
    container_name: backend
    env_file: .env
    depends_on:
      - db

  frontend:
    build:
      context: ./frontend
    container_name: frontend
    depends_on:
      - backend

  nginx:
    image: nginx:1.27-alpine
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      # 프론트 정적 파일 (멀티스테이지에서 이미 내장되어 있으면 볼륨 불필요)
      - ./nginx/default.conf.template:/etc/nginx/templates/default.conf.template:ro
    depends_on:
      - backend
      - frontend
    environment:
      # nginx 템플릿 사용 (alpine의 /etc/nginx/templates)
      NGINX_ENVSUBST_OUTPUT_DIR: "/etc/nginx"
      # 필요 시 ENV 치환 변수 추가 가능

  db:
    image: postgres:16-alpine
    container_name: db
    env_file: .env
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

> 운영에서는 보통 **프론트 컨테이너가 Nginx 이미지를 그대로 사용**해도 됩니다(멀티스테이지 최종 이미지를 nginx로).
>
> 위 compose 예시는 **별도의 nginx 서비스**를 두는 구성입니다.
>
> 간소화하려면 프론트 이미지만 Nginx로 두고 거기서 백엔드 프록시 설정까지 같이 처리할 수도 있습니다.

---

#### 4. 개발 → 운영 전환 플로우

**1. 개발 모드**

```shell
docker compose up --build
# frontend:5173, backend:8000
```

**2. 프론트 빌드 산출 확인(선택)**

```shell
docker compose run --rm frontend npm run build
```

**3. 운영 모드(예: 별도 compose.prod.yml 사용)**

```shell
docker compose -f docker-compose.yml -f docker-compose.prod.yml up --build -d
```

> 운영 compose에서는 **frontend 컨테이너가 이미 dist를 포함**하고, **nginx가 / 와 /api 를 라우팅**합니다.

---

#### 5. 자주 묻는 팁

- **CORS**: 개발은 vite 프록시, 운영은 Nginx로 같은 도메인/경로(/api) 사용 → CORS 문제 최소화 
- **STATIC/MEDIA**: 업로드 파일(media)은 볼륨 마운트로 **영속화** 
- **환경 변수**: Django ALLOWED_HOSTS, DB 접속 정보 등은 .env로 분리
- **헬스체크**: Nginx에서 /api/health 같은 간단한 헬스 엔드포인트 두면 편함
- **로그/에러**: gunicorn access/error 로그, Nginx access/error 로그를 분리해 두면 문제 추적 용이
