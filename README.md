# Auth Service

Authentication & User Management microservice for the Video Converter platform.

## Overview

The Auth Service provides JWT-based authentication and user account management for the Video Converter system. It handles user signup, login, and token verification used by other services.

**Service Details:**
- **Port**: 8001
- **Framework**: FastAPI 0.104.1
- **Database**: PostgreSQL (shared)
- **Authentication**: JWT (python-jose)

## Quick Start

### Prerequisites
- Docker & Docker Compose
- OR Python 3.11+ with pip

### Option 1: Docker (Recommended)

```bash
# Start with docker-compose (from monorepo root)
docker-compose -f docker/docker-compose.yml up auth-service
```

### Option 2: Local Development

```bash
# Install dependencies
pip install -r requirements.txt
pip install -r ../shared/requirements.txt

# Set environment variables
export DATABASE_URL=postgresql://user:password@localhost:5432/videoconverter
export SECRET_KEY=your-secret-key-here
export REDIS_URL=redis://localhost:6379
export RABBITMQ_URL=amqp://guest:guest@localhost:5672/

# Run the service
python -m uvicorn app.main:app --reload --port 8001
```

## API Endpoints

### Health Check
```bash
curl http://localhost:8001/health
```

Response:
```json
{"status": "ok", "service": "auth-service"}
```

### Create User Account

**Endpoint:** `POST /auth/signup`

```bash
curl -X POST http://localhost:8001/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "testpass123"
  }'
```

**Response:**
```json
{
  "id": 1,
  "username": "testuser",
  "created_at": "2026-01-21T10:00:00Z"
}
```

### User Login

**Endpoint:** `POST /auth/login`

```bash
curl -X POST http://localhost:8001/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "testpass123"
  }'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

### Verify Token

**Endpoint:** `POST /auth/verify`

```bash
curl -X POST http://localhost:8001/auth/verify \
  -H "Content-Type: application/json" \
  -d '{"token": "your-jwt-token-here"}'
```

**Response:**
```json
{
  "valid": true,
  "username": "testuser",
  "exp": 1673856000
}
```

## Project Structure

```
auth-service/
├── app/
│   ├── main.py          # FastAPI app initialization and health endpoint
│   └── routes.py        # Auth endpoints (signup, login, verify)
├── Dockerfile           # Production container configuration
├── requirements.txt     # Python dependencies
└── README.md           # This file
```

## Key Dependencies

- **fastapi** - Web framework
- **uvicorn** - ASGI server
- **sqlalchemy** - ORM
- **psycopg2-binary** - PostgreSQL adapter
- **python-jose** - JWT token handling
- **passlib** - Password hashing
- **pydantic** - Data validation

See [requirements.txt](requirements.txt) for full list.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | - | PostgreSQL connection string |
| `SECRET_KEY` | - | Secret key for JWT signing |
| `REDIS_URL` | redis://localhost:6379 | Redis connection URL |
| `RABBITMQ_URL` | amqp://guest:guest@localhost:5672/ | RabbitMQ broker URL |

## Development

### Run Tests

```bash
# From monorepo root
pytest tests/test_auth.py -v
```

Test coverage:
- ✅ Health check endpoint
- ✅ User signup validation
- ✅ Duplicate user detection
- ✅ Login authentication
- ✅ Invalid credentials handling
- ✅ Token verification
- ✅ Token expiration

### Lint Code

```bash
flake8 auth-service/ --max-line-length=127
```

### Security Scanning

```bash
safety check -r requirements.txt
bandit -r app/
```

## CI/CD

This service has automated workflows:

- **CI Pipeline** (`.github/workflows/auth-ci.yml`):
  - ✅ Lint (flake8)
  - ✅ Security scan (safety, bandit)
  - ✅ Tests (pytest)
  - Triggered on: Changes to `auth-service/` or `shared/`

- **Docker Build** (`.github/workflows/auth-docker.yml`):
  - ✅ Build Docker image
  - ✅ Push to ghcr.io (on main/develop)
  - Image: `ghcr.io/Ericatici/video-converter-auth`

**Badge:** ![Auth CI](https://github.com/Ericatici/auth-service/actions/workflows/auth-ci.yml/badge.svg)

## Architecture

The Auth Service is part of a larger microservices ecosystem:

```
┌─────────────────────────────────────────┐
│          Client Application             │
└──────────────────┬──────────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │   Auth Service       │
        │  (Port 8001)         │
        └──────────┬───────────┘
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
    PostgreSQL           Redis
    (Users)              (Cache)
```

## Integration with Other Services

The Auth Service provides token verification for:
- **Video Service** - Verifies user tokens on upload/download requests
- **Notification Service** - Validates webhook access

Token verification endpoint (`/auth/verify`) is used by other services to validate JWT tokens.

## Monitoring

When deployed with the full stack:
- **Prometheus Metrics** available at: `http://localhost:8001/metrics`
- **Grafana Dashboard**: "API Services" shows request metrics
- **Alerts**: Email notifications for errors > 1%

## Troubleshooting

**Issue**: "Could not connect to the database"
- Verify PostgreSQL is running: `docker-compose ps db`
- Check `DATABASE_URL` environment variable
- Ensure database exists and user has permissions

**Issue**: "Invalid token" errors
- Verify `SECRET_KEY` is consistent across services
- Check token expiration: tokens expire after 24 hours
- Ensure token format is correct: `Bearer <token>`

**Issue**: CI tests fail locally but pass in GitHub Actions
- Check Python version: Must be 3.11+
- Verify all dependencies installed: `pip install -r requirements.txt`
- Ensure PostgreSQL running locally: `docker-compose up -d db`

## Contributing

1. Create feature branch: `git checkout -b feature/my-feature`
2. Test locally: `pytest tests/test_auth.py -v`
3. Lint code: `flake8 auth-service/`
4. Commit changes: `git commit -m "feature: add new endpoint"`
5. Push and create pull request

## License

MIT

## Resources

- **Main Project**: [video-converter-prod](https://github.com/Ericatici/video-converter-prod)
- **Video Service**: [video-service](https://github.com/Ericatici/video-service)
- **Notification Service**: [notification-service](https://github.com/Ericatici/notification-service)
- **API Documentation**: See main project README for full architecture
