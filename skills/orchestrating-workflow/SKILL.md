---
name: orchestrating-workflow
description: Use when starting a new project from scratch or resuming an incomplete project workflow - orchestrates the full development flow from requirements to QA with parallel frontend/backend development
---

# Master Workflow

## Overview

Orchestrates the complete development workflow with **parallel frontend and backend development** enabled by API contracts as the shared interface.

**Key Principle**: After API contracts are defined, frontend and backend can develop simultaneously. Frontend uses mocks until backend is ready.

## Workflow Stages

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Phase 1: REQUIREMENTS                            │
│                    dev-workflow:requirements                            │
│            (captures BOTH frontend AND backend requirements)            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        Phase 2: ARCHITECTURE                            │
│                     dev-workflow:architecture                           │
│                                                                         │
│  Outputs:                                                               │
│  - docs/architecture.md                                                 │
│  - docs/api-contracts/*.yaml    ← SHARED CONTRACT (enables parallel)   │
│  - docs/frontend-architecture/*.md                                      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         Phase 3: STORAGE                                │
│                      dev-workflow:storage                               │
│                     (per backend system)                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
          ┌─────────────────────────┴─────────────────────────┐
          │            Phase 4: PARALLEL DEVELOPMENT          │
          │                                                   │
          ▼                                                   ▼
┌──────────────────────┐                      ┌──────────────────────┐
│  dev-workflow:       │                      │  dev-workflow:       │
│  backend             │                      │  frontend            │
│                      │                      │                      │
│  - Implements API    │                      │  - Uses Mock API     │
│  - Database layer    │                      │  - UI components     │
│  - Business logic    │                      │  - State management  │
└──────────────────────┘                      └──────────────────────┘
          │                                                   │
          └─────────────────────────┬─────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       Phase 5: INTEGRATION                              │
│                                                                         │
│  - Connect frontend to real backend                                     │
│  - Remove mocks, configure real API URL                                 │
│  - Verify API contract compliance                                       │
│  - Fix any mismatches                                                   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          Phase 6: QA                                    │
│                        dev-workflow:qa                                  │
│                                                                         │
│  - Unit tests (backend + frontend)                                      │
│  - Integration tests                                                    │
│  - E2E tests (full stack)                                               │
└─────────────────────────────────────────────────────────────────────────┘
```

## Process

### Step 1: Check Current Progress

Check which artifacts already exist:

| File | Stage | Enables |
|------|-------|---------|
| `docs/requirements.md` | Requirements | Architecture can start |
| `docs/architecture.md` | Architecture | - |
| `docs/api-contracts/api.yaml` | Architecture | **Frontend AND Backend can start** |
| `docs/frontend-architecture/frontend.md` | Architecture | Frontend can start |
| `docs/storage/backend.md` | Storage | Backend can start |
| `backend/` directory exists | Development | Backend in progress |
| `frontend/` directory exists | Development | Frontend in progress |

**Key Point**: Once `docs/api-contracts/api.yaml` exists, BOTH frontend and backend can start in parallel.

Report progress to user and ask where to resume.

### Step 2: Execute Pre-Development Stages

For incomplete stages (Requirements → Architecture → Storage):
1. Invoke the corresponding skill
2. Wait for user confirmation
3. Proceed to next stage

**Important**: Storage is only needed for backend. Frontend can start after Architecture.

### Step 3: Parallel Development Phase

After Architecture is complete (API contracts exist):

**Ask user:**
```
API contracts are ready. Both frontend and backend can now develop in parallel.

How would you like to proceed?

1. **Parallel** (Recommended): Start both frontend and backend simultaneously
   - Frontend will use mock APIs based on contracts
   - Backend implements real APIs
   - Integration happens when both are ready

2. **Backend first**: Complete backend, then start frontend
   - Traditional approach, no mocks needed
   - Frontend waits for real APIs

3. **Frontend first**: Complete frontend with mocks, then backend
   - Good for UX-driven projects
   - Backend implements what frontend expects

4. **Single system only**: Only develop frontend OR backend
   - For API-only or UI-only projects
```

#### If Parallel Development:

**For Backend:**
1. Ensure `docs/storage/<system>.md` exists
2. Invoke `dev-workflow:backend`

**For Frontend (can start immediately):**
1. Frontend uses `docs/api-contracts/*.yaml` to generate mocks
2. Invoke `dev-workflow:frontend` with mock mode enabled

**Communicate to user:**
```
Starting parallel development:

BACKEND TRACK:
- Working on: backend-<name>
- API contract: docs/api-contracts/<name>.yaml
- Storage design: docs/storage/<name>.md

FRONTEND TRACK:
- Working on: frontend-<name>
- API contract: docs/api-contracts/<name>.yaml (for mocks)
- Architecture: docs/frontend-architecture/<name>.md
- Mock mode: ENABLED

Both tracks can progress independently. We'll integrate when ready.
```

### Step 4: Integration Phase

After both frontend and backend are developed:

**Integration Checklist:**
1. [ ] Backend is running and healthy
2. [ ] Update frontend API base URL to real backend
3. [ ] Disable mock mode in frontend
4. [ ] Test all API endpoints
5. [ ] Verify response formats match contracts
6. [ ] Fix any contract mismatches

**If mismatches found:**
- Document the mismatch
- Decide: fix backend or update frontend?
- Update API contract if needed
- Re-test

### Step 5: QA Testing

After integration:
1. List completed systems
2. Invoke `dev-workflow:qa` for full test suite
3. Run E2E tests covering frontend → backend flows

## Standard Project Layout

**All projects MUST follow this monorepo structure:**

```
project-root/
├── docs/                        # ALL DOCUMENTATION
│   ├── requirements.md          # Phase 1
│   ├── architecture.md          # Phase 2
│   ├── api-contracts/           # Phase 2 - SHARED CONTRACT
│   │   └── api.yaml             # Enables parallel development
│   ├── frontend-architecture/   # Phase 2
│   │   └── frontend.md
│   └── storage/                 # Phase 3
│       └── backend.md
├── backend/                     # ALL BACKEND CODE
├── frontend/                    # ALL FRONTEND CODE
├── docker-compose.yaml          # Full stack orchestration
└── README.md
```

**Key Rules:**
- `docs/` - All documentation, NOT inside backend/ or frontend/
- `backend/` - Single directory for all backend code
- `frontend/` - Single directory for all frontend code
- Root `docker-compose.yaml` - Combines backend + frontend + dependencies

## Progress Tracking

### Artifact Checklist

```
docs/
├── requirements.md              [ ] Phase 1
├── architecture.md              [ ] Phase 2
├── api-contracts/
│   └── api.yaml                 [ ] Phase 2 (enables parallel dev)
├── frontend-architecture/
│   └── frontend.md              [ ] Phase 2
└── storage/
    └── backend.md               [ ] Phase 3

backend/                         [ ] Phase 4 (Backend Track)
frontend/                        [ ] Phase 4 (Frontend Track)

Integration verified             [ ] Phase 5
E2E tests passing                [ ] Phase 6
```

## Commands

- `/dev-workflow:master` - Start or resume full workflow
- `/dev-workflow:master status` - Check current progress only
- `/dev-workflow:master parallel` - Force parallel development mode

## Key Principles

- **API Contract First**: Define contracts before implementation
- **Parallel by Default**: Frontend and backend develop simultaneously
- **Mock-Enabled Frontend**: Frontend doesn't wait for backend
- **Integration Phase**: Explicit step to connect frontend and backend
- **Contract Compliance**: Both sides must match the agreed contract
