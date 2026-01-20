# Dev Workflow Plugin for Claude Code

A comprehensive development workflow plugin that guides you through the entire software development lifecycle - from requirements gathering to QA testing.

## Features

- **Parallel Development**: Frontend and backend develop simultaneously using shared API contracts
- **Proactive Clarification**: AI asks clarifying questions to eliminate ambiguity (for BOTH frontend and backend)
- **Contract-First Design**: API contracts defined before implementation enable true parallel work
- **Mock-First Frontend**: Frontend uses MSW mocks while backend is being developed
- **DDD Architecture**: Domain-Driven Design for backend systems
- **Modern Frontend Stack**: React + TypeScript + shadcn/ui + MSW
- **Comprehensive Testing**: Test pyramid with unit, integration, and E2E tests

## Installation

### As a Plugin Marketplace

```bash
# Add the marketplace
/plugin marketplace add carbin-gun/dev-workflow

# Install the plugin
/plugin install dev-workflow@dev-workflow-marketplace
```

### Local Development

```bash
# Clone the repository
git clone <repo-url> ~/.claude/plugins/dev-workflow
```

## Available Commands

| Command | Description |
|---------|-------------|
| `/dev-workflow` | Start or resume full workflow (alias for master) |
| `/dev-workflow:requirements` | Requirements gathering with User Stories |
| `/dev-workflow:architecture` | System architecture design |
| `/dev-workflow:storage` | Data model and storage design |
| `/dev-workflow:backend` | Backend development (Go + DDD) |
| `/dev-workflow:frontend` | Frontend development (React + shadcn/ui) |
| `/dev-workflow:qa` | QA testing with test pyramid |
| `/dev-workflow:master` | Full workflow orchestrator |

## Workflow Stages

```
┌─────────────────────────────────────────────────────────────┐
│                    Phase 1: REQUIREMENTS                    │
│           (captures BOTH frontend AND backend needs)        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 2: ARCHITECTURE                    │
│                                                             │
│  Outputs:                                                   │
│  - docs/architecture.md                                     │
│  - docs/api-contracts/*.yaml  ← ENABLES PARALLEL DEV        │
│  - docs/frontend-architecture/*.md                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Phase 3: STORAGE                        │
│                   (per backend system)                      │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┴─────────────────────┐
        │         Phase 4: PARALLEL DEVELOPMENT     │
        │                                           │
        ▼                                           ▼
┌─────────────────┐                     ┌─────────────────┐
│     Backend     │                     │    Frontend     │
│   (Go + DDD)    │                     │  (React + TS)   │
│                 │                     │                 │
│ - Real APIs     │                     │ - Mock APIs     │
│ - Database      │                     │ - UI Components │
└─────────────────┘                     └─────────────────┘
        │                                           │
        └─────────────────────┬─────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 5: INTEGRATION                     │
│  - Connect frontend to real backend                         │
│  - Disable mocks, verify API contract compliance            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Phase 6: QA                            │
│  - Unit tests (backend + frontend)                          │
│  - Integration tests, E2E tests                             │
└─────────────────────────────────────────────────────────────┘
```

### Key Innovation: Parallel Development

**API Contract as Source of Truth**: Once `docs/api-contracts/*.yaml` is defined, both frontend and backend can develop simultaneously:

| Track | What It Does | Blocking? |
|-------|--------------|-----------|
| **Backend** | Implements real API endpoints | No - independent |
| **Frontend** | Uses MSW mocks based on same contract | No - independent |
| **Integration** | Connects frontend to real backend | Waits for both |

This eliminates the traditional bottleneck where frontend waits for backend APIs.

### Stage 1: Requirements (`/dev-workflow:requirements`)

**Output**: `docs/requirements.md`

- Collaborative requirements gathering through dialogue
- User Stories with acceptance criteria
- **Frontend Requirements**: UI/UX, device support, accessibility, i18n
- **Backend Requirements**: API design, data model, business logic
- Proactive clarification of edge cases, error handling, user roles
- Priority assignment (P0/P1/P2)

### Stage 2: Architecture (`/dev-workflow:architecture`)

**Output**:
- `docs/architecture.md`
- `docs/api-contracts/<system>.yaml` ← **Shared contract for parallel dev**
- `docs/frontend-architecture/<system>.md` ← **Frontend architecture**
- `docs/grpc-contracts/*.proto` (if needed)

- System split proposal with trade-offs
- **Frontend architecture**: State management, routing, component structure
- **Backend architecture**: API design, database design
- **Mock strategy**: How frontend will mock APIs during parallel development

### Stage 3: Storage (`/dev-workflow:storage`)

**Output**: `docs/storage/<system>.md`

- Entity identification and field definitions
- Relationship mapping (1:1, 1:N, M:N)
- Storage technology selection
- Index design based on query patterns

### Stage 4: Backend Development (`/dev-workflow:backend`)

**Tech Stack**: Go + Gin + gRPC + sqlx + DDD

**Output**: Complete backend system with:
- DDD layer structure (domain, application, infrastructure, interfaces)
- HTTP handlers and gRPC servers
- Database migrations
- Dockerfile and docker-compose.yaml
- Makefile for common operations

### Stage 5: Frontend Development (`/dev-workflow:frontend`)

**Tech Stack**: React 18+ + TypeScript + Vite + shadcn/ui + Tailwind CSS + MSW

**Can start immediately after Architecture** (doesn't wait for backend!)

**Output**: Complete frontend system with:
- Feature-based architecture
- Hooks for logic, components for UI
- **MSW mock handlers** based on API contracts
- API client layer (works with mocks or real backend)
- Dockerfile and docker-compose.yaml

### Stage 6: Integration

**After both frontend and backend are complete:**

- Switch frontend from mocks to real backend
- Verify all API endpoints work correctly
- Fix any contract mismatches
- Run integration tests

### Stage 7: QA Testing (`/dev-workflow:qa`)

**Test Pyramid**:
- Unit tests (most) - Domain logic, utilities, hooks
- Integration tests (medium) - API endpoints, component interactions
- E2E tests (few) - Critical user flows

## Project Structure (Standard Monorepo Layout)

```
project-root/
│
├── docs/                              # ALL DOCUMENTATION
│   ├── requirements.md                # Phase 1: Requirements
│   ├── architecture.md                # Phase 2: System architecture
│   ├── api-contracts/                 # Phase 2: SHARED API CONTRACTS
│   │   └── api.yaml                   # ← Enables parallel development
│   ├── frontend-architecture/         # Phase 2: Frontend architecture
│   │   └── frontend.md
│   ├── storage/                       # Phase 3: Storage design
│   │   └── backend.md
│   └── grpc-contracts/                # Phase 2: gRPC (if needed)
│       └── <service>.proto
│
├── backend/                           # ALL BACKEND CODE
│   ├── cmd/server/
│   │   └── main.go
│   ├── internal/
│   │   ├── domain/                    # DDD: Entities, Value Objects
│   │   ├── application/               # DDD: Use Cases, DTOs
│   │   ├── infrastructure/            # DDD: DB, External Services
│   │   └── interfaces/                # DDD: HTTP/gRPC Handlers
│   ├── migrations/
│   ├── api/                           # Copy of API contracts
│   ├── Dockerfile
│   ├── docker-compose.yaml
│   ├── Makefile
│   └── go.mod
│
├── frontend/                          # ALL FRONTEND CODE
│   ├── src/
│   │   ├── components/                # Shared UI components
│   │   ├── features/                  # Feature modules
│   │   ├── hooks/                     # Shared hooks
│   │   ├── api/                       # API client layer
│   │   ├── mocks/                     # MSW mock handlers
│   │   │   ├── handlers.ts            # Generated from api-contracts
│   │   │   └── browser.ts
│   │   ├── stores/                    # Global state
│   │   └── types/                     # TypeScript types
│   ├── public/
│   ├── Dockerfile
│   ├── docker-compose.yaml
│   ├── package.json
│   └── vite.config.ts
│
├── docker-compose.yaml                # Root: Full stack orchestration
└── README.md
```

**Key Layout Rules:**
- `docs/` - All documentation, contracts, and design documents
- `backend/` - All backend code (single directory, not `backend-<name>/`)
- `frontend/` - All frontend code (single directory, not `frontend-<name>/`)
- Root `docker-compose.yaml` - Orchestrates full stack for development

## Usage Example

### Start a New Project

```
User: /dev-workflow

Claude: Let me check your project progress...

No existing artifacts found in docs/. Starting from Requirements stage.

What problem are we solving with this project?
```

### Resume an Existing Project

```
User: /dev-workflow

Claude: Let me check your project progress...

Found:
✓ docs/requirements.md
✓ docs/architecture.md
✗ docs/storage/ (incomplete)

Would you like to continue with Storage Design?
```

### Jump to a Specific Stage

```
User: /dev-workflow:backend

Claude: Let me verify prerequisites for backend development...

Found for backend-user:
✓ docs/architecture.md
✓ docs/storage/backend-user.md
✓ docs/api-contracts/backend-user.yaml

Ready to start backend development. Which system should I develop?
```

## Key Principles

### Proactive Clarification

The workflow proactively asks about:
- **边界条件** (Boundary conditions)
- **异常情况** (Error scenarios)
- **用户角色** (User roles)
- **数据状态** (Data states)
- **并发场景** (Concurrency)
- **性能预期** (Performance expectations)

### Contract-First Development

API contracts are defined before implementation, enabling:
- Parallel frontend/backend development
- Clear interface agreements
- Automatic validation

### Verification Before Completion

Each development stage includes mandatory verification:
- Compile/build checks
- Container startup
- Health endpoint verification

## Plugin Structure

```
dev-workflow/
├── .claude-plugin/
│   ├── marketplace.json    # Marketplace definition
│   └── plugin.json         # Plugin manifest
├── commands/               # Command entry points
│   ├── requirements.md
│   ├── architecture.md
│   ├── storage.md
│   ├── backend.md
│   ├── frontend.md
│   ├── qa.md
│   ├── master.md
│   └── dw.md
└── skills/                 # Full skill definitions
    ├── requirements/SKILL.md
    ├── architecture/SKILL.md
    ├── storage/SKILL.md
    ├── backend/SKILL.md
    ├── frontend/SKILL.md
    ├── qa/SKILL.md
    ├── master/SKILL.md
    └── dw/SKILL.md
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

