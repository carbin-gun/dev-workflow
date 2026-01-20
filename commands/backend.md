---
name: backend
description: Backend development with Go + Gin + gRPC + DDD architecture
---

Use the `dev-workflow:backend` skill to develop backend systems.

This skill will:
- Implement DDD layers (domain, application, infrastructure, interfaces)
- Create Go project with Gin HTTP handlers and gRPC servers
- Setup Docker, docker-compose, and Makefile
- Verify compilation and startup

Prerequisites: `docs/architecture.md`, `docs/storage/<system>.md`, and `docs/api-contracts/<system>.yaml` must exist.

Run `/dev-workflow:backend` to start.
