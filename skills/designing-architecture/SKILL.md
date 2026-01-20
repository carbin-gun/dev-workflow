---
name: designing-architecture
description: Use when docs/requirements.md exists but docs/architecture.md is missing - designs system architecture with proactive questioning on boundaries and trade-offs
---

# Architecture Design

## Overview

Design system architecture by splitting the project into frontend and backend systems, defining responsibilities, and establishing communication contracts.

**Core Behavior**: Proactively challenge system boundaries, identify potential issues, and clarify trade-offs before finalizing.

## Prerequisites

**Required**:
- `docs/requirements.md` must exist

**Check**: If missing, prompt user to run `dev-workflow:requirements` first.

## Output

- `docs/architecture.md`
- `docs/api-contracts/api.yaml` (shared API contract - enables parallel development)
- `docs/frontend-architecture/frontend.md` (frontend architecture)
- `docs/grpc-contracts/*.proto` (if backend-to-backend communication needed)

**Standard Monorepo Layout:**
```
project-root/
├── docs/                        # ALL documentation
│   ├── architecture.md
│   ├── api-contracts/
│   │   └── api.yaml             # Shared contract
│   └── frontend-architecture/
│       └── frontend.md
├── backend/                     # ALL backend code
├── frontend/                    # ALL frontend code
└── docker-compose.yaml          # Root: runs full stack
```

## Proactive Clarification Rules

### System Split Challenges

When proposing system split, ALWAYS ask:

| Aspect | Challenge Questions |
|--------|---------------------|
| **边界清晰度** | "这两个服务的边界在哪？会不会有功能两边都想做？" |
| **数据归属** | "这个数据应该属于哪个服务？如果两个服务都需要呢？" |
| **拆分必要性** | "这两个服务真的需要分开吗？合在一起的代价是什么？" |
| **通信开销** | "这两个服务需要频繁通信吗？拆开后延迟能接受吗？" |
| **独立部署** | "这个服务需要独立扩缩容吗？独立发布吗？" |

### API Design Challenges

For each API endpoint:

| Aspect | Challenge Questions |
|--------|---------------------|
| **幂等性** | "这个接口需要幂等吗？重复调用会怎样？" |
| **版本兼容** | "接口升级时怎么保持兼容？" |
| **错误处理** | "失败时返回什么错误码？前端如何处理？" |
| **认证授权** | "这个接口需要登录吗？需要什么权限？" |
| **限流策略** | "需要限流吗？限流后返回什么？" |

### Cross-System Challenges

| Aspect | Challenge Questions |
|--------|---------------------|
| **事务一致性** | "跨服务操作需要事务吗？用 Saga 还是 TCC？" |
| **数据同步** | "服务间数据如何保持同步？实时还是最终一致？" |
| **服务发现** | "服务如何互相发现？用注册中心还是 DNS？" |
| **熔断降级** | "依赖的服务挂了怎么办？需要降级策略吗？" |

### Frontend Architecture Challenges

| Aspect | Challenge Questions |
|--------|---------------------|
| **状态管理** | "全局状态用什么方案？Context / Redux / Zustand / Jotai？" |
| **路由策略** | "路由用什么方案？React Router / TanStack Router？需要权限路由吗？" |
| **数据获取** | "数据获取用什么？fetch / axios / TanStack Query / SWR？" |
| **表单处理** | "复杂表单用什么方案？React Hook Form / Formik？" |
| **组件设计** | "组件粒度如何划分？原子设计还是按功能？" |
| **样式方案** | "样式用什么？Tailwind / CSS Modules / styled-components？" |
| **Mock 策略** | "后端未完成时前端如何开发？MSW / JSON Server / 手写 Mock？" |

### Uncertainty Markers

When decisions are unclear, mark them:

```markdown
<!-- ARCH-DECISION: 用户服务和订单服务是否需要拆分？当前假设拆分，待确认 -->
<!-- ARCH-TRADEOFF: gRPC vs REST for backend communication, chose gRPC for performance, user confirmed -->
```

## Process

### Step 1: Analyze Requirements

Read `docs/requirements.md` and identify:
- Major functional areas
- User types and interaction patterns
- Data domains

**追问：**
- "需求里提到的 XXX 功能，你预期用户量和数据量大概是多少？"
- "这些功能中，哪些是性能敏感的？"

### Step 2: Propose System Split

Based on analysis, propose system split:

**Frontend Systems** (React + TypeScript + shadcn/ui):
- Always at least one
- Split by user type or major functional boundary

**Backend Systems** (Go + Gin + gRPC + sqlx + DDD):
- Split by domain boundary
- Each owns its data

**Present proposal with trade-offs:**

```
方案 A（推荐）：拆分为 3 个后端服务
- backend-user: 用户管理
- backend-order: 订单处理
- backend-product: 商品管理

优点：
- 各服务独立部署，互不影响
- 团队可以并行开发

缺点：
- 服务间通信增加复杂度
- 需要处理分布式事务

方案 B：合并为 1 个后端服务
优点：简单，无跨服务问题
缺点：耦合度高，扩展性差

你倾向哪个方案？或者有其他想法？
```

**追问验证：**
- "你确定 XXX 服务需要独立出来吗？它和 YYY 服务的调用频率如何？"
- "如果以后需要拆分/合并，改动成本你能接受吗？"

### Step 3: Define Communication

**Fixed rules**:
- Frontend ↔ Backend: **REST API** (always)
- Backend ↔ Backend: **gRPC** (when needed)

**For each communication path, clarify:**

```
frontend-web → backend-user (REST API)

需要确认：
1. 认证方式？JWT / Session / OAuth？
2. Token 存储？localStorage / Cookie？
3. Token 刷新策略？
4. 跨域配置？
```

```
backend-order → backend-user (gRPC)

需要确认：
1. 同步调用还是异步消息？
2. 调用失败重试策略？
3. 超时时间设置？
4. 需要缓存用户信息吗？
```

### Step 4: Generate API Contracts

For each backend system, create `docs/api-contracts/<system>.yaml`.

**Before writing each endpoint, ask:**
- "这个接口的请求和响应格式是？"
- "有分页吗？分页参数怎么传？"
- "有哪些可能的错误？错误码是什么？"

**API Contract Template:**
```yaml
openapi: 3.0.3
info:
  title: <System Name> API
  version: 1.0.0

servers:
  - url: http://localhost:<port>

paths:
  /health:
    get:
      summary: Health check
      responses:
        '200':
          description: OK

  /api/v1/xxx:
    get:
      summary: xxx
      # ... with error codes documented
      responses:
        '200':
          description: Success
        '400':
          description: Bad Request
        '401':
          description: Unauthorized
        '500':
          description: Internal Error
```

### Step 5: Generate gRPC Contracts (if needed)

For backend-to-backend communication:

**追问：**
- "这个 RPC 调用的超时时间应该设多少？"
- "返回数据量大吗？需要分页或流式传输吗？"

### Step 6: Design Frontend Architecture

For each frontend system, create `docs/frontend-architecture/<system>.md`:

**追问：**
- "前端需要哪些页面？路由结构是什么？"
- "哪些页面需要登录才能访问？权限如何控制？"
- "全局状态有哪些？用户信息？主题？语言？"

**Frontend Architecture Template:**
```markdown
# Frontend Architecture: <system-name>

## 1. Tech Stack
- Framework: React 18+ / Vue 3
- UI Library: shadcn/ui / Ant Design / Element Plus
- State Management: Context / Zustand / Pinia
- Data Fetching: TanStack Query / SWR / axios
- Routing: React Router / Vue Router
- Styling: Tailwind CSS

## 2. Project Structure
```
src/
├── components/     # Shared UI components
├── features/       # Feature modules
├── hooks/          # Shared hooks
├── api/            # API client layer
├── stores/         # Global state
├── types/          # TypeScript types
└── utils/          # Utilities
```

## 3. Routing Structure
| Route | Page | Auth Required |
|-------|------|---------------|

## 4. State Management
### Global State
- User session
- Theme/Language preferences
- App-wide notifications

### Server State (API Cache)
- Managed by TanStack Query / SWR

## 5. API Integration
- Base URL configuration
- Error handling strategy
- Auth token management
- **Mock Strategy**: MSW / JSON Server for parallel development

## 6. Key Architecture Decisions
| Decision | Options | Choice | Reason |
|----------|---------|--------|--------|
```

**Critical: Mock API Strategy**

To enable parallel frontend/backend development:

```markdown
## Mock Strategy

Frontend can start development immediately using:
- **MSW (Mock Service Worker)**: Intercepts network requests
- **JSON Server**: Quick REST API mock
- **Hand-written mocks**: Simple fetch wrapper

Mock data location: `src/mocks/`
Enable in development: `VITE_USE_MOCK=true`
```

### Step 7: Resolve Uncertainties

**Before finalizing, list all `<!-- ARCH-DECISION -->` and `<!-- ARCH-TRADEOFF -->` markers:**

```
架构决策待确认：

1. [ ] 用户服务和订单服务是否拆分？
   - 当前方案：拆分
   - 影响：需要 gRPC 通信，处理分布式事务

2. [ ] 认证方式选择？
   - 选项：JWT / Session / OAuth
   - 推荐：JWT（无状态，易扩展）

请确认或调整。
```

### Step 7: Generate Architecture Document

Create `docs/architecture.md`:

```markdown
# System Architecture

## 1. System Overview

### 1.1 System List

| System | Type | Responsibility | Tech Stack |
|--------|------|----------------|------------|

### 1.2 System Diagram

[ASCII or text description]

### 1.3 Key Architecture Decisions

| Decision | Options | Choice | Reason |
|----------|---------|--------|--------|
| Service split | Monolith vs Microservices | Microservices | 独立部署需求 |
| Auth method | JWT vs Session | JWT | 无状态，易扩展 |

## 2. Communication

### 2.1 Frontend-Backend (REST API)
- Authentication: JWT
- Error handling: Standard error codes

### 2.2 Backend-Backend (gRPC)
- Timeout: 5s default
- Retry: 3 times with exponential backoff

## 3. Backend System Details

### 3.1 <backend-system-name>
- Responsibility boundary
- External API: docs/api-contracts/<system>.yaml
- Storage design: docs/storage/<system>.md

## 4. Frontend System Details

### 4.1 <frontend-system-name>
- Pages and routing
- State management approach
- API consumption
- Architecture document: docs/frontend-architecture/<system>.md

## 5. Parallel Development Strategy

### 5.1 API Contract as Source of Truth
- API contracts (`docs/api-contracts/*.yaml`) enable parallel work
- Both frontend and backend develop against the same contract

### 5.2 Frontend Mock Strategy
- Frontend uses MSW/JSON Server for mocks based on API contracts
- Can start immediately after API contracts are defined
- Switch to real backend when ready

### 5.3 Integration Phase
- Connect frontend to real backend
- Verify contract compliance
- E2E testing

## 6. Open Architecture Questions
Items deferred for later decision.
```

### Step 8: Confirm

Present architecture to user:

**Final verification questions:**
- "系统边界清晰吗？有没有职责不明确的地方？"
- "API 契约完整吗？有遗漏的接口吗？"
- "通信方式和错误处理策略确认了吗？"

## Key Principles

- **Challenge before confirm** - Question every split, every boundary
- **Explicit trade-offs** - Document why we chose A over B
- **API contract first** - Enable parallel development
- **Mark uncertainties** - Track decisions and their rationale
- **Defense in depth** - Plan for failures, timeouts, retries
