---
name: master
description: Use when starting a new project from scratch or resuming an incomplete project workflow - orchestrates the full development flow from requirements to QA
---

# Master Workflow

## Overview

Orchestrates the complete development workflow: Requirements -> Architecture -> Storage -> Development -> QA Testing.

## Workflow Stages

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ dev-workflow:   │───▶│ dev-workflow:   │───▶│ dev-workflow:   │
│ requirements    │    │ architecture    │    │ storage         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                      │
       ┌──────────────────────────────────────────────┘
       ▼
┌─────────────────────────────────────────────────────────────┐
│ Development (parallel or sequential)                        │
│ - dev-workflow:backend (per backend system)                 │
│ - dev-workflow:frontend (per frontend system)               │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────┐
│ dev-workflow:qa │
└─────────────────┘
```

## Process

### Step 1: Check Current Progress

Check which artifacts already exist in `docs/`:

| File | Stage |
|------|-------|
| `docs/requirements.md` | Requirements |
| `docs/architecture.md` | Architecture |
| `docs/storage/*.md` | Storage Design |
| `docs/api-contracts/*.yaml` | API Contracts |

Report progress to user and ask where to resume.

### Step 2: Execute Stages Sequentially

For each incomplete stage:
1. Invoke the corresponding skill
2. Wait for user confirmation
3. Proceed to next stage

### Step 3: Development Phase

After Storage is complete:
1. List all systems from `architecture.md`
2. Ask user which system to develop (or all)
3. User decides: sequential or parallel
4. Invoke `dev-workflow:backend` or `dev-workflow:frontend` per system

### Step 4: QA Testing

After development:
1. List completed systems
2. Ask user which to test
3. Invoke `dev-workflow:qa` per system
4. Final integration test

## Commands

- `/dev-workflow:master` - Start or resume full workflow
- `/dev-workflow:master status` - Check current progress only
