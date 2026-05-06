## Step-by-Step Installation

### Step 1: Clone the Repository

Open your terminal/command prompt and run:

```bash
git clone https://github.com/makeplane/plane.git
cd plane
```

---

### Step 2: Create Root Environment File

Create `.env` file in the root directory:

**Windows:**
```bash
copy .env.example .env
```

**Linux/Mac:**
```bash
cp .env.example .env
```

# Database Settings
POSTGRES_USER="plane"
POSTGRES_PASSWORD="plane"
POSTGRES_DB="plane"
PGDATA="/var/lib/postgresql/data"

# Redis Settings
REDIS_HOST="plane-redis"
REDIS_PORT="6379"

# RabbitMQ Settings
RABBITMQ_HOST="plane-mq"
RABBITMQ_PORT="5672"
RABBITMQ_USER="plane"
RABBITMQ_PASSWORD="plane"
RABBITMQ_VHOST="plane"

LISTEN_HTTP_PORT=80
LISTEN_HTTPS_PORT=443

# AWS Settings
AWS_REGION=""
AWS_ACCESS_KEY_ID="access-key"
AWS_SECRET_ACCESS_KEY="secret-key"
AWS_S3_ENDPOINT_URL="http://plane-minio:9000"
AWS_S3_BUCKET_NAME="uploads"
FILE_SIZE_LIMIT=5242880

# GPT settings
OPENAI_API_BASE="https://api.openai.com/v1"
OPENAI_API_KEY="sk-"
GPT_ENGINE="gpt-3.5-turbo"

# Docker Settings
DOCKERIZED=1
USE_MINIO=1

# SSL Settings
CERT_ACME_CA=https://acme-v02.api.letsencrypt.org/directory
TRUSTED_PROXIES=0.0.0.0/0
SITE_ADDRESS=:80
CERT_EMAIL=
CERT_ACME_DNS=
MINIO_ENDPOINT_SSL=0

# API Settings
API_KEY_RATE_LIMIT="60/minute"

# Application URLs
WEB_URL=http://localhost:3000
API_BASE_URL=http://localhost:8000
LIVE_SERVER_SECRET_KEY=supersecret123

# Django Secret Key (IMPORTANT: Change this in production)
SECRET_KEY=7u21i0eft22j6feyur62l4jgmvu5uzks2z0ou9f9htkxu1v42v

# Allowed Hosts
ALLOWED_HOSTS=localhost:3000,127.0.0.1,localhost,web,api
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://localhost:3001,http://localhost:3002




### Step 3: Create API Environment File

Create `apps/api/.env` file:

**Windows:**
```bash
copy apps\api\.env.example apps\api\.env
```

**Linux/Mac:**
```bash
cp apps/api/.env.example apps/api/.env
```

**Edit `apps/api/.env` file and replace ALL content with:**

# Backend
DEBUG=0
CORS_ALLOWED_ORIGINS="http://localhost:3000,http://localhost:3001,http://localhost:3002,http://localhost:3100"

# Database Settings
POSTGRES_USER="plane"
POSTGRES_PASSWORD="plane"
POSTGRES_HOST="plane-db"
POSTGRES_DB="plane"
POSTGRES_PORT=5432
DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}

# Redis Settings
REDIS_HOST="plane-redis"
REDIS_PORT="6379"
REDIS_URL="redis://${REDIS_HOST}:6379/"

# RabbitMQ Settings
RABBITMQ_HOST="plane-mq"
RABBITMQ_PORT="5672"
RABBITMQ_USER="plane"
RABBITMQ_PASSWORD="plane"
RABBITMQ_VHOST="plane"

# AWS Settings (IMPORTANT: Use plane-minio, not localhost)
AWS_REGION=""
AWS_ACCESS_KEY_ID="access-key"
AWS_SECRET_ACCESS_KEY="secret-key"
AWS_S3_ENDPOINT_URL="http://plane-minio:9000"
AWS_S3_BUCKET_NAME="uploads"
FILE_SIZE_LIMIT=5242880
SIGNED_URL_EXPIRATION=3600

# Docker Settings
DOCKERIZED=1
USE_MINIO=1

# Email redirections and minio domain settings
WEB_URL="http://localhost:3000"

# Gunicorn Workers
GUNICORN_WORKERS=2

# Base URLs
ADMIN_BASE_URL="http://localhost:3001"
ADMIN_BASE_PATH="/god-mode"

SPACE_BASE_URL="http://localhost:3002"
SPACE_BASE_PATH="/spaces"

APP_BASE_URL="http://localhost:3000"
APP_BASE_PATH=""

LIVE_BASE_URL="http://localhost:3100"
LIVE_BASE_PATH="/live"

LIVE_SERVER_SECRET_KEY="secret-key"

# Hard delete files after days
HARD_DELETE_AFTER_DAYS=60

# Force HTTPS for handling SSL Termination
MINIO_ENDPOINT_SSL=0

# API key rate limit
API_KEY_RATE_LIMIT="60/minute"

# Django Secret Key (IMPORTANT: Must match root .env)
SECRET_KEY="7u21i0eft22j6feyur62l4jgmvu5uzks2z0ou9f9htkxu1v42v"


### Step 4: Create Web App Environment File

Create `apps/web/.env` file:

**Windows:**
```bash
copy apps\web\.env.example apps\web\.env
```

**Linux/Mac:**
```bash
cp apps/web/.env.example apps/web/.env
```

**Edit `apps/web/.env` file and replace ALL content with:**

VITE_API_BASE_URL="http://localhost:8000"

VITE_WEB_BASE_URL="http://localhost:3000"

VITE_ADMIN_BASE_URL="http://localhost:3001"
VITE_ADMIN_BASE_PATH="/god-mode"

VITE_SPACE_BASE_URL="http://localhost:3002"
VITE_SPACE_BASE_PATH="/spaces"

VITE_LIVE_BASE_URL="http://localhost:3100"
VITE_LIVE_BASE_PATH="/live"


### Step 5: Update docker-compose.yml (No Proxy, Single App Port)

**IMPORTANT:** Open `docker-compose.yml` and find the `web` service section.

**Replace the entire `web` service with:**

services:
  web:
    container_name: web
    build:
      context: .
      dockerfile: ./apps/web/Dockerfile.web
      args:
        DOCKER_BUILDKIT: 1
        VITE_API_BASE_URL: "http://localhost:8000"
        VITE_WEB_BASE_URL: "http://localhost:3000"
        VITE_ADMIN_BASE_URL: "http://localhost:3001"
        VITE_ADMIN_BASE_PATH: "/god-mode"
        VITE_SPACE_BASE_URL: "http://localhost:3002"
        VITE_SPACE_BASE_PATH: "/spaces"
        VITE_LIVE_BASE_URL: "http://localhost:3100"
        VITE_LIVE_BASE_PATH: "/live"
    restart: always
    depends_on:
      - api
    ports:
    - "3000:3000"

  admin:
    container_name: admin
    build:
      context: .
      dockerfile: ./apps/admin/Dockerfile.admin
      args:
        DOCKER_BUILDKIT: 1
    restart: always
    depends_on:
      - api
      - web
    ports:
      - "3001:3001"

  space:
    container_name: space
    build:
      context: .
      dockerfile: ./apps/space/Dockerfile.space
      args:
        DOCKER_BUILDKIT: 1
    restart: always
    depends_on:
      - api
      - web
    ports:
      - "3002:3002"

  api:
    container_name: api
    build:
      context: ./apps/api
      dockerfile: Dockerfile.api
      args:
        DOCKER_BUILDKIT: 1
    restart: always
    command: ./bin/docker-entrypoint-api.sh
    env_file:
      - ./apps/api/.env
    depends_on:
      - plane-db
      - plane-redis
    ports:
      - "8000:8000"

  worker:
    container_name: bgworker
    build:
      context: ./apps/api
      dockerfile: Dockerfile.api
      args:
        DOCKER_BUILDKIT: 1
    restart: always
    command: ./bin/docker-entrypoint-worker.sh
    env_file:
      - ./apps/api/.env
    depends_on:
      - api
      - plane-db
      - plane-redis

  beat-worker:
    container_name: beatworker
    build:
      context: ./apps/api
      dockerfile: Dockerfile.api
      args:
        DOCKER_BUILDKIT: 1
    restart: always
    command: ./bin/docker-entrypoint-beat.sh
    env_file:
      - ./apps/api/.env
    depends_on:
      - api
      - plane-db
      - plane-redis

  migrator:
    container_name: plane-migrator
    build:
      context: ./apps/api
      dockerfile: Dockerfile.api
      args:
        DOCKER_BUILDKIT: 1
    restart: no
    command: ./bin/docker-entrypoint-migrator.sh
    env_file:
      - ./apps/api/.env
    depends_on:
      - plane-db
      - plane-redis

  live:
    container_name: plane-live
    build:
      context: .
      dockerfile: ./apps/live/Dockerfile.live
      args:
        DOCKER_BUILDKIT: 1
    restart: always
    ports:
      - "3100:3100"

  plane-db:
    container_name: plane-db
    image: postgres:15.7-alpine
    restart: always
    command: postgres -c 'max_connections=1000'
    volumes:
      - pgdata:/var/lib/postgresql/data
    env_file:
      - .env
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      PGDATA: /var/lib/postgresql/data

  plane-redis:
    container_name: plane-redis
    image: valkey/valkey:7.2.11-alpine
    restart: always
    volumes:
      - redisdata:/data

  plane-mq:
    container_name: plane-mq
    image: rabbitmq:3.13.6-management-alpine
    restart: always
    env_file:
      - .env
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
      RABBITMQ_DEFAULT_VHOST: ${RABBITMQ_VHOST}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

  plane-minio:
    container_name: plane-minio
    image: minio/minio
    restart: always
    command: server /export --console-address ":9090"
    volumes:
      - uploads:/export
    environment:
      MINIO_ROOT_USER: ${AWS_ACCESS_KEY_ID}
      MINIO_ROOT_PASSWORD: ${AWS_SECRET_ACCESS_KEY}
    ports:
      - "9000:9000"
      - "9090:9090"



volumes:
  pgdata:
  redisdata:
  uploads:
  rabbitmq_data:
**Keep all other services unchanged.**

**Note:** This keeps your browser entrypoint on **3000**. The API still runs on **8000** for direct browser calls (no proxy).

---

## Starting the Application

### Step 6: Build Docker Containers

This will take 10-20 minutes on first run:

```bash
docker-compose build
```

**Expected output:** You'll see build progress for each service.

---

### Step 7: Start All Services

```bash

```

**Wait 60 seconds** for all services to initialize.

---