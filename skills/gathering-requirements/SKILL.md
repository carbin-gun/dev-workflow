---
name: gathering-requirements
description: Use when starting a new project or when docs/requirements.md is missing - guides collaborative requirements gathering through User Stories with proactive clarification
---

# Requirements Gathering

## Overview

Collaboratively gather and document project requirements through dialogue, producing a structured requirements document with User Stories as the primary format.

**Core Behavior**: Proactively identify ambiguity and ask clarifying questions. Never assume - always confirm.

## Prerequisites

None - this is the first stage.

## Output

`docs/requirements.md`

## Proactive Clarification Rules

### MUST ASK for Every Feature

When user describes a feature, ALWAYS probe these dimensions:

| Dimension | Example Questions |
|-----------|-------------------|
| **边界条件** | "如果用户输入为空会怎样？" "最大支持多少条数据？" |
| **异常情况** | "如果网络断开会怎样？" "操作失败后用户看到什么？" |
| **用户角色** | "这个功能所有用户都能用吗？" "管理员有额外权限吗？" |
| **数据状态** | "数据删除是软删除还是硬删除？" "历史记录保留多久？" |
| **并发场景** | "两个人同时编辑怎么处理？" "需要乐观锁还是悲观锁？" |
| **性能预期** | "预期多少用户同时使用？" "响应时间要求是什么？" |

### Uncertainty Markers

When user's answer is vague, mark it and revisit:

```markdown
<!-- UNCERTAINTY: 用户未明确并发处理策略，默认假设为乐观锁，待确认 -->
```

Before finalizing document, list ALL uncertainty markers and ask user to resolve.

### Frontend/UI Requirements - MUST ASK

For EVERY project with a frontend, proactively ask:

| Dimension | Example Questions |
|-----------|-------------------|
| **设备支持** | "需要支持移动端吗？平板？响应式还是单独设计？" |
| **设计风格** | "有设计稿吗？偏好什么风格？（简约/企业级/活泼）" |
| **组件库** | "有偏好的 UI 组件库吗？Ant Design/Element/shadcn?" |
| **交互体验** | "列表空状态显示什么？加载用骨架屏还是 spinner？" |
| **无障碍** | "需要支持无障碍访问吗？键盘导航？屏幕阅读器？" |
| **浏览器** | "需要支持哪些浏览器？IE11？移动端浏览器？" |
| **国际化** | "需要多语言支持吗？RTL 布局？" |
| **主题** | "需要深色模式吗？用户可切换主题？" |

### Challenge Assumptions

When user says something like:
- "就像 XXX 那样" → 追问: "XXX 的哪些具体行为是你想要的？"
- "正常的 CRUD" → 追问: "删除需要确认吗？批量操作需要吗？"
- "简单的列表" → 追问: "需要分页吗？排序？筛选？搜索？"
- "简单的界面" → 追问: "需要响应式吗？移动端适配？深色模式？"

## Process

### Step 1: Understand Background

Ask the user (one question at a time):
- What problem are we solving?
- Who are the target users?
- What does success look like?

**Then probe deeper:**
- "除了你提到的用户，还有其他角色吗？比如管理员、审核人员？"
- "你说的成功指标，有具体的数字吗？比如响应时间、并发用户数？"

### Step 2: Gather User Stories

For each feature area, guide user to define User Stories:

```
As a <role>
I want to <action>
So that <benefit>

Acceptance Criteria:
- [ ] AC1
- [ ] AC2
```

**For EACH story, run through the clarification checklist:**

```
Story: 用户登录

✓ 边界条件
  - 密码错误几次锁定？→ [待确认]
  - 锁定多久自动解锁？→ [待确认]

✓ 异常情况
  - 账号不存在提示什么？→ "账号或密码错误"（不暴露账号是否存在）

✓ 用户角色
  - 所有角色同一个登录入口？→ 是

✓ 安全要求
  - 需要验证码吗？→ 连续错误3次后需要
  - 需要双因素认证吗？→ [待确认]
```

### Step 3: Prioritize

Review all stories and assign priority:
- **P0**: Must have (MVP)
- **P1**: Should have
- **P2**: Nice to have

**追问：** "P0 功能中，有没有哪个是必须先上线的？上线顺序是什么？"

### Step 4: Define Boundaries

Ask user:
- What is explicitly OUT of scope?
- Any technical constraints?
- Non-functional requirements (performance, security)?

**追问清单：**
- "移动端需要支持吗？哪些平台？"
- "需要支持多语言吗？"
- "需要支持离线使用吗？"
- "数据需要导入导出吗？什么格式？"
- "需要对接哪些第三方系统？"

### Step 5: Resolve Uncertainties

**Before generating document:**

1. List all `<!-- UNCERTAINTY -->` markers
2. Present to user as a checklist
3. Get explicit answers for each
4. Only proceed when all resolved or explicitly marked as "TBD for later"

```
在生成文档前，以下问题需要确认：

1. [ ] 密码错误锁定策略？
2. [ ] 是否需要双因素认证？
3. [ ] 数据保留期限？

请逐一回答，或标记为 "后续再定"。
```

### Step 6: Generate Document

Create `docs/requirements.md` with structure:

```markdown
# Project Requirements

## 1. Background & Goals
- Project background
- Core objectives
- Success metrics

## 2. User Stories

### US-001: <title>
- **Role**: As a <role>
- **Goal**: I want to <action>
- **Value**: So that <benefit>
- **Acceptance Criteria**:
  - [ ] AC1
  - [ ] AC2
- **Priority**: P0/P1/P2
- **Open Questions**: (if any TBD items)

## 3. Feature List

| ID | Feature | Description | Priority | Dependencies |
|----|---------|-------------|----------|--------------|

## 4. Frontend Requirements

### 4.1 UI/UX Requirements
- Design style and branding
- Component library choice
- Empty states, loading states, error states

### 4.2 Device & Browser Support
- Responsive design requirements
- Browser compatibility matrix
- Mobile/tablet support

### 4.3 Accessibility & i18n
- Accessibility requirements (WCAG level)
- Internationalization needs
- Theme support (dark mode, etc.)

## 5. Non-Functional Requirements
- Performance
- Security
- Availability

## 6. Boundaries & Constraints
- Out of scope
- Technical constraints

## 7. Open Questions (TBD)
Items deferred for later decision.
```

### Step 7: Confirm

Present document to user for review and confirmation.

**Final check:** "我再过一遍文档，看看有没有遗漏的细节..."

## Key Principles

- **One question at a time** - Don't overwhelm
- **Proactively clarify** - Never assume, always ask
- **Challenge vague answers** - "简单的"、"正常的" 需要追问
- **Mark uncertainties** - Track and resolve before finalizing
- **YAGNI** - Remove unnecessary features, but don't skip clarification
