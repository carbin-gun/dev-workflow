---
name: developing-frontend
description: Use when developing a frontend system - requires architecture.md and API contracts to exist for the backend systems it consumes
---

# Frontend Development

## Overview

Develop a frontend system using React + TypeScript + shadcn/ui with strict separation of UI and logic.

**Core Behavior**: When encountering ambiguity in API contracts or UX requirements, STOP and ask. Never assume - clarify first.

## Prerequisites

**Required**:
- `docs/architecture.md`
- `docs/api-contracts/*.yaml` (for backend APIs this frontend consumes)
- `docs/frontend-architecture/<system>.md` (if exists, use as guide)

**Optional** (for parallel development):
- Backend does NOT need to be complete
- Frontend can use mock APIs based on API contracts

**Check**: If `api-contracts/*.yaml` missing, prompt user to complete Architecture stage first.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Framework | React 18+ |
| Language | TypeScript |
| UI Library | shadcn/ui |
| Styling | Tailwind CSS |
| Build Tool | Vite |

## Proactive Clarification During Development

### API Contract Ambiguity - MUST ASK

When reading API contracts, if ANY of these are unclear, STOP and ask:

| Ambiguity | Ask |
|-----------|-----|
| **响应格式不明** | "API 返回的 user 对象具体包含哪些字段？" |
| **错误处理未明确** | "401 错误时前端应该跳转登录还是显示提示？" |
| **分页参数不明** | "分页用 page/size 还是 offset/limit？" |
| **状态枚举不明** | "order.status 有哪些可能的值？各代表什么？" |

### UX/UI Decisions - MUST ASK

Before making UX choices not specified in requirements:

```
在实现 XXX 页面时，遇到以下 UX 问题需要确认：

1. 列表为空时显示什么？
   - 选项：空白 / 插画+提示文字 / 引导创建按钮
   - 需要你确认

2. 加载中显示什么？
   - 选项：全屏 loading / 骨架屏 / 局部 spinner
   - 需要你确认

3. 表单提交后？
   - 选项：停留当前页 / 跳转列表 / 跳转详情
   - 需要你确认
```

### Common UX Questions Checklist

For each page/feature, proactively ask:

| Scenario | Questions |
|----------|-----------|
| **列表页** | 每页多少条？支持跳页吗？空状态显示什么？ |
| **表单页** | 必填项有哪些？校验规则？提交后去哪？ |
| **详情页** | 哪些字段可编辑？编辑是弹窗还是新页面？ |
| **删除操作** | 需要二次确认吗？确认文案是什么？ |
| **错误处理** | 网络错误显示什么？如何重试？ |

### Assumptions Log

When you MUST make assumptions to proceed, log them:

```typescript
// ASSUMPTION: API 契约未明确分页格式，假设使用 { page, pageSize }
// TODO: 确认后更新
interface PaginationParams {
  page: number;
  pageSize: number;
}
```

**Before delivery, list all ASSUMPTION comments and ask user to verify.**

## Project Structure

**Location**: All frontend code goes in `frontend/` directory at project root.

```
frontend/
├── src/
│   ├── components/           # Shared UI components (NO business logic)
│   │   └── ui/              # shadcn/ui components
│   ├── features/            # Feature modules (by business domain)
│   │   └── <feature>/
│   │       ├── components/  # Feature-specific components
│   │       ├── hooks/       # Feature logic (data fetching, state)
│   │       ├── types/       # Feature types
│   │       └── index.ts     # Feature exports
│   ├── hooks/               # Global shared hooks
│   ├── lib/                 # Utilities, helpers
│   ├── api/                 # API client layer
│   │   ├── client.ts        # HTTP client setup
│   │   └── <domain>.ts      # API functions by domain
│   ├── mocks/               # MSW mock handlers (for parallel dev)
│   │   ├── handlers.ts
│   │   └── browser.ts
│   ├── types/               # Global types
│   ├── App.tsx
│   └── main.tsx
├── public/
├── Dockerfile                # Frontend standalone build
├── docker-compose.yaml       # Frontend standalone run
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
└── README.md
```

**Key Layout Rules:**
- Frontend code ONLY in `frontend/` (not `frontend-web/`, `frontend-admin/`)
- Documentation stays in `docs/` at project root
- Frontend has its own `docker-compose.yaml` for standalone operation
- Root `docker-compose.yaml` orchestrates full stack

## Process

### Step 1: Initialize Project

```bash
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install
```

Setup Tailwind CSS and shadcn/ui:
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
npx shadcn-ui@latest init
```

All frontend code lives in `frontend/` at project root.

### Step 2: Setup Mock API Layer (for Parallel Development)

**If backend is not ready**, setup MSW (Mock Service Worker) first:

```bash
npm install msw --save-dev
npx msw init public/ --save
```

**Create mock handlers based on API contracts** (`src/mocks/handlers.ts`):
```typescript
import { http, HttpResponse } from 'msw';

// Generate from docs/api-contracts/*.yaml
export const handlers = [
  http.get('/api/v1/users', () => {
    return HttpResponse.json([
      { id: '1', name: 'Mock User', email: 'mock@example.com' },
    ]);
  }),

  http.post('/api/v1/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: '2', ...body }, { status: 201 });
  }),

  // Add handlers for each endpoint in API contract
];
```

**Setup MSW browser worker** (`src/mocks/browser.ts`):
```typescript
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

**Enable mocks in development** (`src/main.tsx`):
```typescript
async function enableMocking() {
  if (import.meta.env.VITE_USE_MOCK !== 'true') {
    return;
  }

  const { worker } = await import('./mocks/browser');
  return worker.start({
    onUnhandledRequest: 'bypass',
  });
}

enableMocking().then(() => {
  ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
});
```

**Environment variable** (`.env.development`):
```
VITE_USE_MOCK=true
VITE_API_URL=http://localhost:8080
```

### Step 3: Setup API Client Layer

Based on `docs/api-contracts/*.yaml`:

1. **HTTP Client** (`src/api/client.ts`)
```typescript
const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:8080';

export async function apiClient<T>(
  endpoint: string,
  options?: RequestInit
): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
    ...options,
  });

  if (!response.ok) {
    throw new Error(`API Error: ${response.status}`);
  }

  return response.json();
}
```

2. **Domain API Functions** (`src/api/<domain>.ts`)
```typescript
// src/api/users.ts
import { apiClient } from './client';
import type { User, CreateUserRequest } from '@/types';

export const usersApi = {
  list: () => apiClient<User[]>('/api/v1/users'),
  create: (data: CreateUserRequest) =>
    apiClient<User>('/api/v1/users', {
      method: 'POST',
      body: JSON.stringify(data),
    }),
};
```

### Step 4: Implement Features

For each feature in requirements:

#### 4.1 Create Feature Structure
```
src/features/<feature>/
├── components/     # UI components for this feature
├── hooks/          # Logic hooks
├── types/          # Feature types
└── index.ts        # Exports
```

#### 4.2 Logic in Hooks (NOT in components)
```typescript
// src/features/users/hooks/useUsers.ts
export function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const fetchUsers = async () => {
    setLoading(true);
    try {
      const data = await usersApi.list();
      setUsers(data);
    } catch (e) {
      setError(e as Error);
    } finally {
      setLoading(false);
    }
  };

  return { users, loading, error, fetchUsers };
}
```

#### 4.3 UI in Components (props only)
```typescript
// src/features/users/components/UserList.tsx
interface UserListProps {
  users: User[];
  loading: boolean;
  onRefresh: () => void;
}

export function UserList({ users, loading, onRefresh }: UserListProps) {
  if (loading) return <Spinner />;

  return (
    <div>
      <Button onClick={onRefresh}>Refresh</Button>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

#### 4.4 Compose in Page
```typescript
// src/features/users/components/UsersPage.tsx
export function UsersPage() {
  const { users, loading, fetchUsers } = useUsers();

  useEffect(() => {
    fetchUsers();
  }, []);

  return <UserList users={users} loading={loading} onRefresh={fetchUsers} />;
}
```

### Step 5: Add shadcn/ui Components

```bash
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
# ... add as needed
```

Components go to `src/components/ui/`.

### Step 6: Create Startup Files

**Dockerfile**:
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**frontend/docker-compose.yaml** (standalone - run frontend only with mocks):
```yaml
version: '3.8'
services:
  frontend:
    build: .
    ports:
      - "3000:80"
    environment:
      - VITE_USE_MOCK=true
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
      - VITE_USE_MOCK=false
    depends_on:
      - backend

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

**Startup Options:**
- `cd frontend && npm run dev` - Development with hot reload (uses mocks)
- `cd frontend && docker-compose up` - Frontend container only (with mocks)
- `docker-compose up` (from root) - Full stack (real backend)

**package.json scripts**:
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint src --ext ts,tsx",
    "type-check": "tsc --noEmit"
  }
}
```

### Step 7: Write Unit Tests (MANDATORY)

**Every frontend MUST have unit tests covering:**

#### 7.1 Test Structure
```
frontend/
├── src/
│   ├── components/
│   │   └── ui/
│   │       └── Button.test.tsx       # Component tests
│   ├── features/
│   │   └── users/
│   │       ├── hooks/
│   │       │   └── useUsers.test.ts  # Hook tests
│   │       └── components/
│   │           └── UserList.test.tsx # Feature component tests
│   ├── hooks/
│   │   └── useAuth.test.ts           # Shared hook tests
│   └── lib/
│       └── utils.test.ts             # Utility tests
├── vitest.config.ts                  # or jest.config.js
└── package.json
```

#### 7.2 Setup Testing

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

**vitest.config.ts**:
```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
  },
});
```

**src/test/setup.ts**:
```typescript
import '@testing-library/jest-dom';
```

#### 7.3 Test Coverage Requirements

| Layer | What to Test | Min Coverage |
|-------|--------------|--------------|
| **Hooks** | State changes, API calls, error handling | 80% |
| **Components** | Rendering, user interactions, props | 70% |
| **Utils** | Pure functions, helpers | 90% |
| **API Client** | Request/response handling | 70% |

#### 7.4 Test Examples

**Hook Test:**
```typescript
// src/features/users/hooks/useUsers.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useUsers } from './useUsers';

describe('useUsers', () => {
  it('fetches users successfully', async () => {
    const { result } = renderHook(() => useUsers());

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.users).toHaveLength(2);
  });

  it('handles error', async () => {
    // Mock API error
    const { result } = renderHook(() => useUsers());

    await waitFor(() => {
      expect(result.current.error).toBeTruthy();
    });
  });
});
```

**Component Test:**
```typescript
// src/features/users/components/UserList.test.tsx
import { render, screen } from '@testing-library/react';
import { UserList } from './UserList';

describe('UserList', () => {
  it('renders users', () => {
    const users = [
      { id: '1', name: 'Alice', email: 'alice@example.com' },
      { id: '2', name: 'Bob', email: 'bob@example.com' },
    ];

    render(<UserList users={users} loading={false} onRefresh={() => {}} />);

    expect(screen.getByText('Alice')).toBeInTheDocument();
    expect(screen.getByText('Bob')).toBeInTheDocument();
  });

  it('shows loading state', () => {
    render(<UserList users={[]} loading={true} onRefresh={() => {}} />);

    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });

  it('shows empty state', () => {
    render(<UserList users={[]} loading={false} onRefresh={() => {}} />);

    expect(screen.getByText(/no users/i)).toBeInTheDocument();
  });
});
```

#### 7.5 Run Tests
```bash
npm run test           # Run tests
npm run test:coverage  # Check coverage
```

**Add to package.json:**
```json
{
  "scripts": {
    "test": "vitest",
    "test:coverage": "vitest --coverage"
  }
}
```

- ALL tests MUST pass
- If fails: fix and retry

### Step 8: Requirements & Contract Compliance Check (MANDATORY)

**Before delivery, verify implementation matches requirements, contracts, AND business logic.**

#### 8.1 API Contract Compliance

Read `docs/api-contracts/api.yaml` and verify EACH endpoint:

| Check | How to Verify |
|-------|---------------|
| **API client exists** | Function in src/api/ for each endpoint |
| **Request types** | TypeScript interface matches OpenAPI request schema |
| **Response types** | TypeScript interface matches OpenAPI response schema |
| **Error handling** | All status codes (400, 401, 404, 500) handled in UI |
| **MSW mocks** | Mock handlers return contract-compliant responses |

**Create API compliance matrix (MUST present to user):**
```
API Contract Compliance:

| Endpoint | API Client | Types ✓ | Mock ✓ | Error UI ✓ |
|----------|------------|---------|--------|------------|
| POST /api/v1/users | usersApi.create | ✓ | ✓ | ✓ |
| GET /api/v1/users/:id | usersApi.get | ✓ | ✓ | ✓ |
| PUT /api/v1/users/:id | usersApi.update | ✓ | ✓ | ✓ |
| DELETE /api/v1/users/:id | usersApi.delete | ✓ | ✓ | ✓ |

All endpoints implemented: YES/NO
```

#### 8.2 Business Logic Verification (MUST GET USER ACKNOWLEDGEMENT)

**For EACH User Story in requirements, verify UI business logic:**

```
Business Logic Verification:

US-001: User Registration Form
├── Business Rule: Email format validation
│   └── Implementation: useRegisterForm.ts validation
│   └── Test: RegisterForm.test.tsx - invalid email
├── Business Rule: Password strength indicator
│   └── Implementation: PasswordStrength.tsx component
│   └── Test: PasswordStrength.test.tsx
├── Business Rule: Show success message after registration
│   └── Implementation: RegisterPage.tsx success state
│   └── Test: RegisterPage.test.tsx - success flow
└── Status: IMPLEMENTED / PARTIAL / NOT IMPLEMENTED

US-002: User List Page
├── Business Rule: Pagination with 10 items per page
│   └── Implementation: useUsers.ts pagination params
│   └── Test: useUsers.test.ts - pagination
├── Business Rule: Search by name/email
│   └── Implementation: UserSearch.tsx + useUserSearch.ts
│   └── Test: UserSearch.test.tsx
├── Business Rule: Sort by name, email, created_at
│   └── Implementation: UserTable.tsx sortable columns
│   └── Test: UserTable.test.tsx - sorting
└── Status: IMPLEMENTED / PARTIAL / NOT IMPLEMENTED
```

**STOP AND ASK USER:**
```
请确认以下前端业务逻辑实现是否正确：

1. US-001 用户注册表单：
   - [x] 邮箱格式校验 → useRegisterForm.ts 第 23 行
   - [x] 密码强度提示 → PasswordStrength.tsx
   - [x] 注册成功提示 → RegisterPage.tsx 第 67 行

2. US-002 用户列表页：
   - [x] 每页 10 条分页 → useUsers.ts 第 12 行
   - [x] 按姓名/邮箱搜索 → UserSearch.tsx
   - [x] 按列排序 → UserTable.tsx sortable

3. UI/UX 实现：
   - [x] 空状态显示 → EmptyState.tsx
   - [x] 加载状态 → Skeleton 组件
   - [x] 错误提示 → Toast 组件

以上实现是否符合预期？有需要调整的吗？
```

**DO NOT proceed until user acknowledges business logic is correct.**

#### 8.3 Acceptance Criteria Verification

**Map EACH acceptance criterion to code/test:**

```
Acceptance Criteria Traceability:

| AC ID | Description | Implementation | Test |
|-------|-------------|----------------|------|
| AC-001 | Form validates email format | useRegisterForm.ts:23 | RegisterForm.test.tsx |
| AC-002 | Show loading during API calls | useUsers.ts loading state | useUsers.test.ts |
| AC-003 | Responsive at all breakpoints | Tailwind classes | Visual test |
| AC-004 | Empty state when no data | EmptyState.tsx | UserList.test.tsx |
```

#### 8.4 Frontend Requirements Checklist

| Requirement | Implementation | Verified |
|-------------|----------------|----------|
| **Responsive** | Tailwind breakpoints (sm, md, lg, xl) | [ ] 320px [ ] 768px [ ] 1024px [ ] 1440px |
| **Empty states** | EmptyState.tsx component | [ ] All lists checked |
| **Loading states** | Skeleton/Spinner components | [ ] All async ops |
| **Error states** | Toast/Alert for API errors | [ ] All API calls |
| **Accessibility** | ARIA labels, keyboard nav | [ ] Tab navigation works |

#### 8.5 List All Assumptions

```bash
grep -r "ASSUMPTION" src/
```

**Present ALL assumptions to user for verification:**
```
以下是开发过程中做出的假设，请确认：

1. ASSUMPTION: 分页默认每页 10 条
   - 位置: src/hooks/usePagination.ts:5
   - 影响: 所有列表默认显示 10 条
   - 确认: [需要确认]

2. ASSUMPTION: 表单提交后停留在当前页
   - 位置: src/features/users/components/CreateUserForm.tsx:45
   - 影响: 创建成功后不跳转
   - 确认: [需要确认]

3. ASSUMPTION: 删除操作需要二次确认
   - 位置: src/features/users/components/DeleteUserButton.tsx:12
   - 影响: 弹出确认对话框
   - 确认: [需要确认]

请逐一确认或调整。
```

**DO NOT proceed until user confirms all assumptions.**

### Step 9: Delivery Verification (MANDATORY)

**After tests pass and compliance checked, execute these checks:**

#### 9.1 Install Dependencies
```bash
npm install
```
- MUST complete without errors

#### 9.2 Type Check
```bash
npm run type-check
```
- MUST pass with no TypeScript errors

#### 9.3 Test Check
```bash
npm run test
```
- ALL tests MUST pass
- If fails: fix and retry

#### 9.4 Build Check
```bash
npm run build
```
- MUST complete without errors
- If fails: fix and retry

#### 9.5 Start Check (with mocks)
```bash
# For mock mode (parallel development)
VITE_USE_MOCK=true npm run dev

# Or with Docker
docker-compose up -d
```
- Wait for container/server to be ready

#### 9.6 Access Check
```bash
curl http://localhost:3000
```
- MUST return 200
- If fails: check logs, fix, restart

#### 9.7 Result
- All passed → Development complete (mock mode)
- Any failed → Fix → Return to 9.1

**DO NOT claim completion without passing all checks.**

### Step 10: Integration with Real Backend (when backend is ready)

**When backend is complete, switch from mocks to real API:**

1. **Update environment variable**:
```bash
# .env.production or when running locally
VITE_USE_MOCK=false
VITE_API_URL=http://localhost:8080
```

2. **Verify all endpoints work**:
- Test each API call
- Verify response formats match mock data
- Fix any contract mismatches

3. **Integration Checklist**:
- [ ] Backend is running and healthy
- [ ] All API endpoints return expected data
- [ ] Error handling works correctly
- [ ] Auth flow works (login, token refresh, logout)
- [ ] Edge cases handled (empty states, errors)

## Key Principles

- **UI/Logic Separation** - Components receive props, hooks handle logic
- **Feature-based Organization** - Group by business domain, not file type
- **shadcn/ui for UI** - Consistent, accessible components
- **Type Safety** - Full TypeScript, no `any`
- **API Contract Compliance** - Match backend API exactly
- **Requirements Traceability** - Every requirement maps to UI
- **Test Coverage** - Unit tests for hooks and components
- **Mock-First Development** - Use MSW mocks for parallel development
- **Verification Required** - Must type-check, test, build, and start successfully
- **Integration Ready** - Easy switch from mocks to real backend
