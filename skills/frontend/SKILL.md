---
name: frontend
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

**Check**: If missing, prompt user to complete preceding stages.

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

```
<frontend-system>/
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
│   ├── types/               # Global types
│   ├── App.tsx
│   └── main.tsx
├── public/
├── Dockerfile
├── docker-compose.yaml
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
└── README.md
```

## Process

### Step 1: Initialize Project

```bash
npm create vite@latest <system-name> -- --template react-ts
cd <system-name>
npm install
```

Setup Tailwind CSS and shadcn/ui:
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
npx shadcn-ui@latest init
```

### Step 2: Setup API Layer

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

### Step 3: Implement Features

For each feature in requirements:

#### 3.1 Create Feature Structure
```
src/features/<feature>/
├── components/     # UI components for this feature
├── hooks/          # Logic hooks
├── types/          # Feature types
└── index.ts        # Exports
```

#### 3.2 Logic in Hooks (NOT in components)
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

#### 3.3 UI in Components (props only)
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

#### 3.4 Compose in Page
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

### Step 4: Add shadcn/ui Components

```bash
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
# ... add as needed
```

Components go to `src/components/ui/`.

### Step 5: Create Startup Files

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

**docker-compose.yaml**:
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:80"
    environment:
      - VITE_API_URL=http://localhost:8080
```

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

### Step 6: Delivery Verification (MANDATORY)

**After coding, MUST execute these checks:**

#### 6.1 Install Dependencies
```bash
npm install
```
- MUST complete without errors

#### 6.2 Type Check & Build
```bash
npm run build
```
- MUST pass with no TypeScript errors
- If fails: fix and retry

#### 6.3 Start Check
```bash
docker-compose up -d
```
- Wait for container to be ready

#### 6.4 Access Check
```bash
curl http://localhost:3000
```
- MUST return 200
- If fails: check logs, fix, restart

#### 6.5 Result
- All passed → Development complete
- Any failed → Fix → Return to 6.1

**DO NOT claim completion without passing all checks.**

## Key Principles

- **UI/Logic Separation** - Components receive props, hooks handle logic
- **Feature-based Organization** - Group by business domain, not file type
- **shadcn/ui for UI** - Consistent, accessible components
- **Type Safety** - Full TypeScript, no `any`
- **API Contract Compliance** - Match backend API exactly
- **Verification Required** - Must build and start successfully
