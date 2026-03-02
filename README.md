# StockPulse - 주식 섹터 분석 대시보드

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Docker Compose                    │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Next.js  │  │  Spring  │  │  Python Data      │  │
│  │ Frontend │──│  Boot    │──│  Collector        │  │
│  │ :3000    │  │  API     │  │  (6h scheduler)   │  │
│  └──────────┘  │  :8080   │  └────────┬─────────┘  │
│                └────┬─────┘           │             │
│                     │                 │             │
│              ┌──────┴──────┐  ┌──────┴─────┐       │
│              │ PostgreSQL  │  │   Redis    │       │
│              │   :5432     │  │   :6379    │       │
│              └─────────────┘  └────────────┘       │
└─────────────────────────────────────────────────────┘
```

## Tech Stack

| Layer | Tech |
|-------|------|
| Frontend | Next.js 14 (App Router) + TypeScript + Tailwind CSS + shadcn/ui |
| Backend | Spring Boot 3.3 + Java 21 + JPA/Hibernate |
| Data Collector | Python 3.12 + FastAPI + FinanceDataReader + yfinance |
| Database | PostgreSQL 16 |
| Cache | Redis 7 |
| Infra | Docker Compose / Kubernetes |

## Quick Start

```bash
# Docker Compose로 전체 실행
docker compose up -d

# 개발 환경 (각각 별도 터미널)
# 1. DB + Redis
docker compose up postgres redis -d

# 2. Data Collector
cd data-collector && pip install -r requirements.txt && uvicorn main:app --reload --port 8000

# 3. Backend
cd backend && ./gradlew bootRun --args='--spring.profiles.active=local'

# 4. Frontend
cd frontend && npm run dev
```

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/sectors/{market}` | 시장별 섹터 목록 (KR/US) |
| GET | `/api/sectors/detail/{id}` | 섹터 상세 (개별 종목 포함) |
| POST | `/api/sync/{market}` | 수동 데이터 동기화 |
| GET | `/api/sync/status/{market}` | 동기화 상태 확인 |
| GET | `/api/sync/history` | 동기화 이력 |

## Data Sync

- 자동: 6시간마다 (0시, 6시, 12시, 18시)
- 수동: POST `/api/sync/{market}`

## Kubernetes

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/config.yaml
kubectl apply -f k8s/
```