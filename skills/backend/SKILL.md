---
name: backend
description: Use when developing a backend system - requires architecture.md, storage design, and API contracts to exist for the target system
---

# Backend Development

## Overview

Develop a backend system using Go + Gin + gRPC + sqlx with DDD (Domain-Driven Design) architecture.

**Core Behavior**: When encountering ambiguity in contracts or requirements, STOP and ask. Never assume - clarify first.

## Prerequisites

**Required** (for target system):
- `docs/architecture.md`
- `docs/storage/<system>.md`
- `docs/api-contracts/<system>.yaml`
- `docs/grpc-contracts/*.proto` (if system has gRPC communication)

**Check**: If any missing, prompt user to complete preceding stages.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Language | Go |
| Web Framework | Gin |
| Inter-service | gRPC |
| Database Driver | sqlx |
| Architecture | DDD |

## Proactive Clarification During Development

### Contract Ambiguity - MUST ASK

When reading API contracts or storage design, if ANY of these are unclear, STOP and ask:

| Ambiguity | Ask |
|-----------|-----|
| **字段含义不明** | "storage 里的 status 字段，具体有哪些值？每个值代表什么？" |
| **边界值未定义** | "API 契约里 name 字段最大长度是多少？" |
| **错误处理未明确** | "契约里没写用户不存在返回什么，应该返回 404 还是空数据？" |
| **业务规则模糊** | "订单取消后，关联的支付记录怎么处理？" |
| **性能要求不明** | "这个列表接口需要缓存吗？缓存多久？" |

### Implementation Decisions - MUST ASK

Before making implementation choices not covered in contracts:

```
在实现 XXX 功能时，遇到以下问题需要确认：

1. 密码加密算法选择？
   - 选项：bcrypt / argon2 / scrypt
   - 我的建议：bcrypt（成熟稳定）
   - 需要你确认

2. 数据库连接池大小？
   - 契约未指定
   - 我的建议：最大 20 连接
   - 需要你确认
```

### Assumptions Log

When you MUST make assumptions to proceed, log them:

```go
// ASSUMPTION: 契约未明确，假设 email 大小写不敏感
// TODO: 确认后更新
func (r *UserRepo) FindByEmail(email string) (*User, error) {
    return r.db.Query("... WHERE LOWER(email) = LOWER(?)", email)
}
```

**Before delivery, list all ASSUMPTION comments and ask user to verify.**

## Project Structure (DDD)

```
<backend-system>/
├── cmd/
│   └── server/
│       └── main.go              # Entry point
├── internal/
│   ├── domain/                  # Domain Layer
│   │   ├── entity/             # Entities, Value Objects
│   │   ├── repository/         # Repository interfaces
│   │   └── service/            # Domain services
│   ├── application/            # Application Layer
│   │   ├── usecase/            # Use cases
│   │   └── dto/                # Data Transfer Objects
│   ├── infrastructure/         # Infrastructure Layer
│   │   ├── persistence/        # Repository implementations
│   │   ├── external/           # External service clients
│   │   └── config/             # Configuration
│   └── interfaces/             # Interface Layer
│       ├── http/               # HTTP handlers (Gin)
│       │   ├── handler/
│       │   ├── middleware/
│       │   └── router/
│       └── grpc/               # gRPC servers
├── pkg/                        # Shared packages
├── api/                        # API definitions
│   ├── openapi/               # OpenAPI specs (copy from docs)
│   └── proto/                 # Proto files (copy from docs)
├── migrations/                 # Database migrations
├── Dockerfile
├── docker-compose.yaml
├── Makefile
├── go.mod
└── README.md
```

## Process

### Step 1: Initialize Project

```bash
mkdir <system-name> && cd <system-name>
go mod init <module-path>
```

Create directory structure as above.

### Step 2: Implement Domain Layer

Based on `docs/storage/<system>.md`:

1. **Entities** (`internal/domain/entity/`)
   - Map storage entities to Go structs
   - Add domain logic methods

2. **Repository Interfaces** (`internal/domain/repository/`)
   - Define interfaces for data access
   - Keep domain layer independent of infrastructure

```go
// internal/domain/repository/user_repository.go
type UserRepository interface {
    FindByID(ctx context.Context, id uuid.UUID) (*entity.User, error)
    FindByEmail(ctx context.Context, email string) (*entity.User, error)
    Save(ctx context.Context, user *entity.User) error
}
```

### Step 3: Implement Infrastructure Layer

1. **Repository Implementations** (`internal/infrastructure/persistence/`)
   - Implement repository interfaces using sqlx

2. **Database Setup**
   - Create migrations in `migrations/`
   - Setup connection pooling

### Step 4: Implement Application Layer

Based on requirements and use cases:

1. **Use Cases** (`internal/application/usecase/`)
   - One use case per business operation
   - Orchestrate domain objects

```go
// internal/application/usecase/create_user.go
type CreateUserUseCase struct {
    userRepo repository.UserRepository
}

func (uc *CreateUserUseCase) Execute(ctx context.Context, input CreateUserInput) (*CreateUserOutput, error) {
    // Business logic
}
```

### Step 5: Implement Interface Layer

Based on `docs/api-contracts/<system>.yaml`:

1. **HTTP Handlers** (`internal/interfaces/http/handler/`)
   - Map OpenAPI endpoints to handlers
   - Use Gin framework

2. **Router** (`internal/interfaces/http/router/`)
   - Setup routes matching API contract

3. **gRPC Servers** (if needed)
   - Generate from proto files
   - Implement service interfaces

### Step 6: Create Startup Files

**Dockerfile**:
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o server ./cmd/server

FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/server .
EXPOSE 8080
CMD ["./server"]
```

**docker-compose.yaml**:
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: <dbname>
      POSTGRES_USER: <user>
      POSTGRES_PASSWORD: <password>
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

**Makefile**:
```makefile
.PHONY: build run test docker-up docker-down

build:
	go build -o bin/server ./cmd/server

run:
	go run ./cmd/server

test:
	go test ./...

docker-up:
	docker-compose up -d

docker-down:
	docker-compose down
```

### Step 7: Delivery Verification (MANDATORY)

**After coding, MUST execute these checks:**

#### 7.1 Compile Check
```bash
go build ./...
```
- MUST pass with no errors
- If fails: fix and retry

#### 7.2 Start Check
```bash
docker-compose up -d
```
- Wait for containers to be ready (max 60s)
- Check logs: `docker-compose logs`

#### 7.3 Health Check
```bash
curl http://localhost:8080/health
```
- MUST return 200 OK
- If fails: check logs, fix, restart

#### 7.4 Result
- All passed → Development complete
- Any failed → Fix → Return to 7.1

**DO NOT claim completion without passing all checks.**

## Key Principles

- **DDD Layers** - Respect layer boundaries, dependencies flow inward
- **Repository Pattern** - Domain doesn't know about database
- **API Contract Compliance** - Match OpenAPI spec exactly
- **Verification Required** - Must compile and start successfully
