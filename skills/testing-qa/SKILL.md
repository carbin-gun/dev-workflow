---
name: testing-qa
description: Use when system code is complete and needs testing - implements test pyramid with unit, integration, and E2E tests for both frontend and backend systems
---

# QA Testing

## Overview

Implement comprehensive testing following the test pyramid: unit tests (most), integration tests (medium), E2E tests (few).

## Prerequisites

**Required**:
- Target system code exists and can start successfully
- System has passed delivery verification (compile + start)

**Check**: If system cannot start, prompt user to fix first.

## Test Pyramid

```
        ╱╲
       ╱  ╲           E2E Tests (少量)
      ╱────╲          - Critical user flows
     ╱      ╲         - Cross-system integration
    ╱────────╲
   ╱          ╲       Integration Tests (中量)
  ╱────────────╲      - API endpoint tests
 ╱              ╲     - Component interaction tests
╱────────────────╲
        ▲             Unit Tests (大量)
        │             - Domain logic
                      - Utility functions
                      - Hooks / pure components
```

## Backend Testing (Go)

### Structure

```
<backend-system>/
├── internal/
│   ├── domain/
│   │   ├── entity/
│   │   │   └── user_test.go          # Unit: entity logic
│   │   └── service/
│   │       └── user_service_test.go  # Unit: domain service
│   ├── application/
│   │   └── usecase/
│   │       └── create_user_test.go   # Unit: use case
│   └── interfaces/
│       └── http/
│           └── handler/
│               └── user_handler_test.go  # Integration: HTTP
└── tests/
    └── e2e/
        └── user_flow_test.go         # E2E: full flow
```

### Unit Tests

Test domain logic in isolation:

```go
// internal/domain/entity/user_test.go
func TestUser_ValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {"valid email", "test@example.com", false},
        {"invalid email", "invalid", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            user := &User{Email: tt.email}
            err := user.Validate()
            if (err != nil) != tt.wantErr {
                t.Errorf("Validate() error = %v, wantErr %v", err, tt.wantErr)
            }
        })
    }
}
```

### Integration Tests

Test HTTP handlers with real HTTP calls:

```go
// internal/interfaces/http/handler/user_handler_test.go
func TestUserHandler_Create(t *testing.T) {
    // Setup test server
    router := setupTestRouter()

    // Create request
    body := `{"email": "test@example.com"}`
    req := httptest.NewRequest("POST", "/api/v1/users", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")

    // Execute
    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)

    // Assert
    assert.Equal(t, http.StatusCreated, w.Code)
}
```

### E2E Tests

Test complete flows with database:

```go
// tests/e2e/user_flow_test.go
func TestUserRegistrationFlow(t *testing.T) {
    // Setup: real database, real server
    cleanup := setupTestEnvironment()
    defer cleanup()

    // Test complete user registration flow
    // 1. Create user
    // 2. Login
    // 3. Get profile
    // 4. Update profile
}
```

### Run Backend Tests

```makefile
# Makefile additions
test-unit:
	go test ./internal/... -short

test-integration:
	go test ./internal/... -run Integration

test-e2e:
	go test ./tests/e2e/...

test-all:
	go test ./...

test-coverage:
	go test ./... -coverprofile=coverage.out
	go tool cover -html=coverage.out
```

## Frontend Testing (React)

### Structure

```
<frontend-system>/
├── src/
│   ├── hooks/
│   │   └── useAuth.test.ts           # Unit: hooks
│   ├── lib/
│   │   └── utils.test.ts             # Unit: utilities
│   ├── components/
│   │   └── ui/
│   │       └── Button.test.tsx       # Unit: UI components
│   └── features/
│       └── users/
│           ├── hooks/
│           │   └── useUsers.test.ts  # Unit: feature hooks
│           └── components/
│               └── UserList.test.tsx # Integration: feature
└── e2e/
    └── user-flow.spec.ts             # E2E: Playwright
```

### Setup Testing Libraries

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
npm install -D @playwright/test
```

**vite.config.ts**:
```typescript
export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
  },
});
```

### Unit Tests

Test hooks and utilities:

```typescript
// src/hooks/useAuth.test.ts
import { renderHook, act } from '@testing-library/react';
import { useAuth } from './useAuth';

describe('useAuth', () => {
  it('should login successfully', async () => {
    const { result } = renderHook(() => useAuth());

    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });

    expect(result.current.isAuthenticated).toBe(true);
  });
});
```

Test UI components:

```typescript
// src/components/ui/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('should call onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### Integration Tests

Test feature components with mocked API:

```typescript
// src/features/users/components/UserList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserList } from './UserList';
import { usersApi } from '@/api/users';

vi.mock('@/api/users');

describe('UserList', () => {
  it('should display users from API', async () => {
    vi.mocked(usersApi.list).mockResolvedValue([
      { id: '1', email: 'test@example.com' },
    ]);

    render(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('test@example.com')).toBeInTheDocument();
    });
  });
});
```

### E2E Tests (Playwright)

```typescript
// e2e/user-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Flow', () => {
  test('should complete registration', async ({ page }) => {
    await page.goto('/register');

    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=Welcome')).toBeVisible();
  });
});
```

### Run Frontend Tests

```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

## Integration Testing (Cross-System)

After individual systems pass:

1. Start all systems via docker-compose
2. Run E2E tests that span frontend + backend
3. Verify API contract compliance

```yaml
# docker-compose.test.yaml
version: '3.8'
services:
  frontend:
    build: ./frontend-web
    ports:
      - "3000:80"
  backend-user:
    build: ./backend-user
    ports:
      - "8080:8080"
  backend-order:
    build: ./backend-order
    ports:
      - "8081:8080"
  db:
    image: postgres:15
```

## Process

### Step 1: Identify Target System

Ask user which system to test:
- Backend system name?
- Frontend system name?
- Or integration test?

### Step 2: Verify System Starts

```bash
docker-compose up -d
# Wait and verify health
```

### Step 3: Implement Tests (Bottom-Up)

1. **Unit tests first** - Most coverage
2. **Integration tests** - API/component level
3. **E2E tests** - Critical paths only

### Step 4: Run All Tests

```bash
# Backend
make test-all

# Frontend
npm run test:unit
npm run test:e2e
```

### Step 5: Report Results

- Total tests
- Pass/fail count
- Coverage percentage
- Failed test details

## Key Principles

- **Test Pyramid** - More unit tests, fewer E2E tests
- **Test in Isolation** - Unit tests mock dependencies
- **Test Real Behavior** - Integration tests use real services
- **Critical Paths** - E2E covers most important user flows
- **Fast Feedback** - Unit tests run in seconds
