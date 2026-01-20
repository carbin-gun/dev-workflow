---
name: master
description: Full development workflow orchestrator - from requirements to QA
---

Use the `dev-workflow:master` skill to orchestrate the complete development workflow.

This skill will:
- Check current progress in `docs/`
- Guide through each stage sequentially
- Invoke appropriate skills for each phase
- Coordinate development and testing

Workflow: Requirements → Architecture → Storage → Development → QA

Run `/dev-workflow:master` to start or resume the full workflow.
