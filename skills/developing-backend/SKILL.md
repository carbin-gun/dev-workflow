---
name: developing-backend
description: Use when developing a backend system - requires architecture.md, storage design, and API contracts to exist for the target system
---

# Backend Development

## Overview

Develop a backend system using Go + Gin + gRPC + sqlx with DDD (Domain-Driven Design) architecture.

**Core Behavior**: When encountering ambiguity in contracts or requirements, STOP and ask. Never assume - clarify first.

## Prerequisites

**Required**:
- `docs/architecture.md`
- `docs/storage/backend.md`
- `docs/api-contracts/api.yaml`
- `docs/grpc-contracts/*.proto` (if gRPC communication needed)

**Check**: If any missing, prompt user to complete preceding stages.

**Note**: Frontend does NOT need to be complete. Backend and frontend can develop in parallel using shared API contracts.

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

**Location**: All backend code goes in `backend/` directory at project root.

```
backend/
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
├── api/                        # API definitions (copy from docs)
│   ├── openapi/
│   └── proto/
├── migrations/                 # Database migrations
├── Dockerfile                  # Backend standalone build
├── docker-compose.yaml         # Backend standalone run (with DB)
├── Makefile
├── go.mod
└── README.md
```

**Key Layout Rules:**
- Backend code ONLY in `backend/` (not `backend-user/`, `backend-order/`)
- Documentation stays in `docs/` at project root
- Backend has its own `docker-compose.yaml` for standalone operation
- Root `docker-compose.yaml` orchestrates full stack

## Process

### Step 1: Initialize Project

```bash
mkdir -p backend && cd backend
go mod init <module-path>
```

Create directory structure as above. All backend code lives in `backend/` at project root.

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

**backend/docker-compose.yaml** (standalone - run backend only):
```yaml
version: '3.8'
services:
  backend:
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

**Root docker-compose.yaml** (full stack - backend + frontend):
```yaml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    environment:
      - VITE_API_URL=http://backend:8080
    depends_on:
      - backend

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

**Startup Options:**
- `cd backend && docker-compose up` - Backend only (for backend development)
- `docker-compose up` (from root) - Full stack (for integration testing)

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

### Step 7: Write Unit Tests (MANDATORY)

**Every backend MUST have unit tests covering:**

#### 7.1 Test Structure
```
backend/
├── internal/
│   ├── domain/
│   │   ├── entity/
│   │   │   └── user_test.go          # Entity tests
│   │   └── service/
│   │       └── user_service_test.go  # Domain service tests
│   ├── application/
│   │   └── usecase/
│   │       └── create_user_test.go   # Use case tests
│   └── interfaces/
│       └── http/
│           └── handler/
│               └── user_handler_test.go  # Handler tests
```

#### 7.2 Test Coverage Requirements

| Layer | What to Test | Min Coverage |
|-------|--------------|--------------|
| **Domain Entity** | Validation, business rules | 80% |
| **Domain Service** | Domain logic | 80% |
| **Use Case** | Business flow, error handling | 70% |
| **Handler** | Request/response, status codes | 70% |
| **Repository** | CRUD operations (with test DB) | 60% |

#### 7.3 Test Example
```go
// internal/domain/entity/user_test.go
func TestUser_Validate(t *testing.T) {
    tests := []struct {
        name    string
        user    User
        wantErr bool
    }{
        {"valid user", User{Email: "test@example.com", Name: "Test"}, false},
        {"empty email", User{Email: "", Name: "Test"}, true},
        {"invalid email", User{Email: "invalid", Name: "Test"}, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := tt.user.Validate()
            if (err != nil) != tt.wantErr {
                t.Errorf("Validate() error = %v, wantErr %v", err, tt.wantErr)
            }
        })
    }
}
```

#### 7.4 Run Tests
```bash
go test ./... -v
go test ./... -cover  # Check coverage
```
- ALL tests MUST pass
- If fails: fix and retry

### Step 8: Requirements & Contract Compliance Check (MANDATORY)

**Before delivery, verify implementation matches requirements, contracts, AND business logic.**

#### 8.1 API Contract Compliance

Read `docs/api-contracts/api.yaml` and verify EACH endpoint:

| Check | How to Verify |
|-------|---------------|
| **Endpoint exists** | Handler implemented for each path |
| **Request schema** | Go struct matches OpenAPI schema exactly |
| **Response schema** | Response DTO matches OpenAPI schema exactly |
| **Status codes** | All documented codes handled (200, 400, 401, 404, 500) |
| **Error format** | Error response matches contract error schema |

**Create API compliance matrix (MUST present to user):**
```
API Contract Compliance:

| Endpoint | Handler | Request ✓ | Response ✓ | Errors ✓ |
|----------|---------|-----------|------------|----------|
| POST /api/v1/users | CreateUser | ✓ | ✓ | ✓ |
| GET /api/v1/users/:id | GetUser | ✓ | ✓ | ✓ |
| PUT /api/v1/users/:id | UpdateUser | ✓ | ✓ | ✓ |
| DELETE /api/v1/users/:id | DeleteUser | ✓ | ✓ | ✓ |

All endpoints implemented: YES/NO
```

#### 8.2 Business Logic Verification (MUST GET USER ACKNOWLEDGEMENT)

**For EACH User Story in requirements, verify business logic:**

```
Business Logic Verification:

US-001: User Registration
├── Business Rule: Email must be unique
│   └── Implementation: UserRepo.FindByEmail() check in CreateUserUseCase
│   └── Test: TestCreateUser_DuplicateEmail
├── Business Rule: Password minimum 8 characters
│   └── Implementation: User.Validate() in domain/entity/user.go
│   └── Test: TestUser_Validate_PasswordTooShort
├── Business Rule: Send welcome email after registration
│   └── Implementation: EmailService.SendWelcome() in CreateUserUseCase
│   └── Test: TestCreateUser_SendsWelcomeEmail
└── Status: IMPLEMENTED / PARTIAL / NOT IMPLEMENTED

US-002: User Login
├── Business Rule: Lock account after 5 failed attempts
│   └── Implementation: LoginAttemptService in application/service/
│   └── Test: TestLogin_AccountLocked
├── Business Rule: JWT token expires in 24 hours
│   └── Implementation: JWTConfig.Expiration = 24h
│   └── Test: TestLogin_TokenExpiration
└── Status: IMPLEMENTED / PARTIAL / NOT IMPLEMENTED
```

**STOP AND ASK USER:**
```
请确认以下业务逻辑实现是否正确：

1. US-001 用户注册：
   - [x] 邮箱唯一性检查 → CreateUserUseCase 第 45 行
   - [x] 密码最少 8 位 → User.Validate() 第 23 行
   - [x] 注册后发送欢迎邮件 → CreateUserUseCase 第 67 行

2. US-002 用户登录：
   - [x] 5 次失败后锁定账户 → LoginAttemptService 第 34 行
   - [x] JWT 24 小时过期 → config/jwt.go 第 12 行

以上业务逻辑实现是否符合预期？有需要调整的吗？
```

**DO NOT proceed until user acknowledges business logic is correct.**

#### 8.3 Acceptance Criteria Verification

**Map EACH acceptance criterion to code/test:**

```
Acceptance Criteria Traceability:

| AC ID | Description | Implementation | Test |
|-------|-------------|----------------|------|
| AC-001 | Password hashed with bcrypt | usecase/create_user.go:56 | TestCreateUser_PasswordHashed |
| AC-002 | Return 401 for invalid credentials | handler/auth.go:78 | TestLogin_InvalidCredentials |
| AC-003 | Pagination supports page/size | handler/user.go:112 | TestListUsers_Pagination |
```

#### 8.4 List All Assumptions

```bash
grep -r "ASSUMPTION" internal/
```

**Present ALL assumptions to user for verification:**
```
以下是开发过程中做出的假设，请确认：

1. ASSUMPTION: 邮箱大小写不敏感
   - 位置: internal/infrastructure/persistence/user_repo.go:45
   - 影响: 查询时使用 LOWER(email)
   - 确认: [需要确认]

2. ASSUMPTION: 删除用户为软删除
   - 位置: internal/domain/entity/user.go:89
   - 影响: 使用 deleted_at 字段
   - 确认: [需要确认]

请逐一确认或调整。
```

**DO NOT proceed until user confirms all assumptions.**

### Step 9: Delivery Verification (MANDATORY)

**After tests pass and compliance checked, execute these checks:**

#### 9.1 Compile Check
```bash
go build ./...
```
- MUST pass with no errors
- If fails: fix and retry

#### 9.2 Test Check
```bash
go test ./... -v
```
- ALL tests MUST pass
- If fails: fix and retry

#### 9.3 Start Check
```bash
docker-compose up -d
```
- Wait for containers to be ready (max 60s)
- Check logs: `docker-compose logs`

#### 9.4 Health Check
```bash
curl http://localhost:8080/health
```
- MUST return 200 OK
- If fails: check logs, fix, restart

#### 9.5 Result
- All passed → Development complete
- Any failed → Fix → Return to 9.1

**DO NOT claim completion without passing all checks.**

## Key Principles

- **DDD Layers** - Respect layer boundaries, dependencies flow inward
- **Repository Pattern** - Domain doesn't know about database
- **API Contract Compliance** - Match OpenAPI spec exactly
- **Requirements Traceability** - Every requirement maps to code
- **Test Coverage** - Unit tests for all layers
- **Verification Required** - Must compile, test, and start successfully
