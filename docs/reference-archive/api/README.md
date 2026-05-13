# Clawboard Documentation

This is the **canonical documentation home for Clawboard** — the self-hosted operating system for AI agent work.

Clawboard is composed of two repos. Docs live alongside the code they describe, but this file is the single entry point for the full system.

> **User-facing documentation** is now at **[moltar-forge.github.io/clawboard](https://moltar-forge.github.io/clawboard/)**. This file indexes developer-focused docs that live alongside the code.

---

## Clawboard — start here

- **What is Clawboard**: `docs/clawboard.md`
- **First run (full stack)**: `docs/getting-started/first-run.md`
- **Configuration reference**: `docs/configuration.md`
- **System architecture**: `docs/architecture.md`
- **RBAC / permissions**: `docs/security/roles-and-permissions.md`

---

## Clawboard API docs

This repo. Covers the backend service, database, OpenClaw integration, and deployment.

### Getting started

- **Local dev**: `docs/guides/local-development.md`
- **Docker dev**: `docs/guides/docker.md`
- **OpenClaw local dev**: `docs/guides/openclaw-local-development.md`
- **Kubernetes deployment**: `docs/guides/kubernetes-deployment.md`

### Database

- **Migrations**: `docs/guides/database-migrations.md`
- **Constraints**: `docs/guides/database-constraints.md`

### OpenClaw integration

- **Overview**: `docs/openclaw/README.md`
- **Public API contract**: `docs/api/openclaw-public-api.md`
- **Workspace integration**: `docs/openclaw/workspace/`
- **Agent-to-agent access**: `docs/openclaw/agent-to-agent-access.md`

### Features

- **Standups**: `docs/features/standups.md`
- **Model fleet management**: `docs/features/model-fleet-management.md`

### Security

- **Secrets management**: `docs/security/secrets.md`
- **RBAC / permissions**: `docs/security/roles-and-permissions.md`

### Operations

- **Deployment**: `docs/deployment.md`
- **Release checklist**: `docs/release/checklist.md`
- **Troubleshooting**: `docs/troubleshooting/`

---

## Clawboard Dashboard docs

The dashboard is a separate repo. Its docs cover the UI, features, and frontend deployment.

See: [`docs/dashboard.md`](dashboard.md) for the full index, or go directly to the [clawboard-dashboard repository](https://github.com/moltar-forge/clawboard/tree/main/web).

---

## Where to put new docs

- **Guides / how-tos**: `docs/guides/`
- **Security / policies**: `docs/security/`
- **External API contracts**: `docs/api/`
- **OpenClaw integration**: `docs/openclaw/`
- **Design proposals / decisions**: `docs/decisions/`
- **Historical references** (not canonical): `docs/archive/`
- **Dashboard docs**: see the dashboard repo

## Canonical vs historical

- **Canonical docs** are the ones linked above and should match current behavior.
- **Historical docs** are kept under `docs/archive/` for reference only and may be outdated.
