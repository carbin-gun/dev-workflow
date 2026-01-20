---
name: storage
description: Use when docs/architecture.md exists but docs/storage/ is incomplete - designs data models and storage for each backend system
---

# Storage Design

## Overview

Design data models and storage strategy for each backend system, following a data-model-first approach.

## Prerequisites

**Required**:
- `docs/requirements.md`
- `docs/architecture.md` (must contain backend system list)

**Check**: If missing, prompt user to run preceding stages first.

## Output

`docs/storage/<backend-system>.md` (one per backend system)

## Process

### Step 1: Identify Backend Systems

Read `docs/architecture.md` and list all backend systems.

Check existing storage docs in `docs/storage/`:
- Which systems already have storage design?
- Which need to be created?

Ask user which system to design (or all remaining).

### Step 2: For Each Backend System

#### 2.1 Identify Entities

Based on system responsibility and requirements:
- What are the core entities (nouns)?
- What data does this system own?

Present proposed entities to user:
```
Backend System: backend-user

Proposed Entities:
- User: Core user data
- UserProfile: Extended profile info
- Session: Login sessions

Does this look complete?
```

#### 2.2 Define Fields

For each entity, define:
- Field name
- Data type
- Constraints (PK, FK, UNIQUE, NOT NULL, etc.)
- Description

```
Entity: User

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK | Primary key |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Login email |
| password_hash | VARCHAR(255) | NOT NULL | Hashed password |
| created_at | TIMESTAMP | NOT NULL | Creation time |
| updated_at | TIMESTAMP | NOT NULL | Last update |
```

#### 2.3 Define Relationships

Map relationships between entities:
- One-to-One
- One-to-Many
- Many-to-Many

```
User 1 ──── 1 UserProfile
User 1 ──── N Session
```

#### 2.4 Choose Storage Technology

Based on data characteristics:

| Factor | Consider |
|--------|----------|
| Relational data | PostgreSQL / MySQL |
| Document data | MongoDB |
| Cache / Session | Redis |
| Search | Elasticsearch |
| File storage | S3 / MinIO |

Present recommendation with reasoning.

#### 2.5 Design Indexes

Analyze query patterns from requirements:
- Which queries are frequent?
- Which fields are filtered/sorted?

```
| Table | Index | Fields | Type | Purpose |
|-------|-------|--------|------|---------|
| User | idx_user_email | email | UNIQUE | Login lookup |
```

### Step 3: Generate Storage Document

Create `docs/storage/<backend-system>.md`:

```markdown
# <Backend System> Storage Design

## 1. Storage Selection

| Storage Type | Technology | Purpose |
|--------------|------------|---------|
| Primary DB | PostgreSQL | Business data |
| Cache | Redis | Session, hot data |

## 2. Core Entities

### 2.1 User

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK | Primary key |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Email |
| created_at | TIMESTAMP | NOT NULL | Created time |

### 2.2 [Other entities...]

## 3. Entity Relationships

```
User 1 ──── N Order
Order N ──── M Product (via OrderItem)
```

## 4. Index Design

| Table | Index | Fields | Type | Purpose |
|-------|-------|--------|------|---------|
| User | idx_user_email | email | UNIQUE | Login |

## 5. Query Pattern Analysis

| Query Scenario | Frequency | Tables | Optimization |
|----------------|-----------|--------|--------------|
| User login | High | User | email index |
```

### Step 4: Confirm

Present storage design to user:
- Entities complete?
- Relationships correct?
- Storage technology appropriate?
- Indexes sufficient?

## Key Principles

- **Data model first** - Understand entities before choosing technology
- **One system at a time** - Focus on single backend system
- **Query-driven indexing** - Design indexes based on actual query patterns
- **Explicit relationships** - Document all entity relationships clearly
