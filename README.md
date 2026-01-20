# Dev Workflow Plugin for Claude Code

A comprehensive development workflow plugin that guides you through the entire software development lifecycle - from requirements gathering to QA testing.

## Features

- **Structured Workflow**: Sequential stages ensure nothing is missed
- **Proactive Clarification**: AI asks clarifying questions to eliminate ambiguity
- **Contract-First Design**: API contracts enable parallel frontend/backend development
- **DDD Architecture**: Domain-Driven Design for backend systems
- **Modern Frontend Stack**: React + TypeScript + shadcn/ui
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
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Requirements  │───▶│   Architecture  │───▶│     Storage     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                      │
       ┌──────────────────────────────────────────────┘
       ▼
┌─────────────────────────────────────────────────────────────┐
│                      Development                            │
│  ┌─────────────────┐              ┌─────────────────┐      │
│  │     Backend     │              │    Frontend     │      │
│  │  (Go + DDD)     │              │ (React + TS)    │      │
│  └─────────────────┘              └─────────────────┘      │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────┐
│    QA Testing   │
└─────────────────┘
```

### Stage 1: Requirements (`/dev-workflow:requirements`)

**Output**: `docs/requirements.md`

- Collaborative requirements gathering through dialogue
- User Stories with acceptance criteria
- Proactive clarification of edge cases, error handling, user roles
- Priority assignment (P0/P1/P2)
- Non-functional requirements

### Stage 2: Architecture (`/dev-workflow:architecture`)

**Output**:
- `docs/architecture.md`
- `docs/api-contracts/<system>.yaml`
- `docs/grpc-contracts/*.proto` (if needed)

- System split proposal with trade-offs
- Frontend/Backend communication contracts
- API design with error codes
- Authentication and authorization strategy

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

**Tech Stack**: React 18+ + TypeScript + Vite + shadcn/ui + Tailwind CSS

**Output**: Complete frontend system with:
- Feature-based architecture
- Hooks for logic, components for UI
- API client layer
- Dockerfile and docker-compose.yaml

### Stage 6: QA Testing (`/dev-workflow:qa`)

**Test Pyramid**:
- Unit tests (most) - Domain logic, utilities, hooks
- Integration tests (medium) - API endpoints, component interactions
- E2E tests (few) - Critical user flows

## Project Structure

```
your-project/
├── docs/
│   ├── requirements.md           # Stage 1 output
│   ├── architecture.md           # Stage 2 output
│   ├── api-contracts/            # Stage 2 output
│   │   └── <system>.yaml
│   ├── grpc-contracts/           # Stage 2 output (if needed)
│   │   └── <service>.proto
│   └── storage/                  # Stage 3 output
│       └── <system>.md
├── backend-<name>/               # Stage 4 output
│   ├── cmd/server/
│   ├── internal/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── interfaces/
│   ├── migrations/
│   ├── Dockerfile
│   └── docker-compose.yaml
└── frontend-<name>/              # Stage 5 output
    ├── src/
    │   ├── components/
    │   ├── features/
    │   ├── hooks/
    │   ├── api/
    │   └── types/
    ├── Dockerfile
    └── docker-compose.yaml
```

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

