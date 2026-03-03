---
name: scaffold-patterns
description: Project structure conventions per tech stack, CI/CD templates, and configuration best practices. Use when bootstrapping new projects or evaluating scaffold output for completeness.
---

# Scaffold Patterns

Project structure conventions, CI/CD templates, and configuration best practices for bootstrapping projects quickly and correctly.

## When to Use This Skill

- Running the `/scaffold-kit:scaffold` command to bootstrap a new project
- Designing directory structure for a specific tech stack
- Setting up CI/CD pipelines for a solo developer project
- Choosing which configuration files to include in a new project

## Stack-Specific Directory Conventions

### Python (FastAPI / Django / Flask)

```
project/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── api/
│   │   ├── __init__.py
│   │   └── routes/
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   └── security.py
│   ├── models/
│   │   └── __init__.py
│   ├── schemas/
│   │   └── __init__.py
│   └── services/
│       └── __init__.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_api/
├── scripts/
├── docs/
├── pyproject.toml
├── Dockerfile
├── docker-compose.yml
├── .env.example
└── README.md
```

### TypeScript (Next.js)

```
project/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── api/
│   ├── components/
│   │   ├── ui/
│   │   └── layout/
│   ├── lib/
│   │   ├── db.ts
│   │   └── utils.ts
│   ├── hooks/
│   └── types/
├── public/
├── tests/
│   └── __tests__/
├── scripts/
├── docs/
├── package.json
├── tsconfig.json
├── next.config.ts
├── Dockerfile
├── docker-compose.yml
├── .env.example
└── README.md
```

### TypeScript (Express / Fastify)

```
project/
├── src/
│   ├── index.ts
│   ├── routes/
│   │   └── index.ts
│   ├── controllers/
│   ├── services/
│   ├── models/
│   ├── middleware/
│   ├── utils/
│   └── types/
├── tests/
│   ├── unit/
│   └── integration/
├── scripts/
├── docs/
├── package.json
├── tsconfig.json
├── Dockerfile
├── docker-compose.yml
├── .env.example
└── README.md
```

### Go

```
project/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handler/
│   ├── service/
│   ├── model/
│   ├── repository/
│   └── config/
├── pkg/
│   └── utils/
├── tests/
├── scripts/
├── docs/
├── go.mod
├── go.sum
├── Dockerfile
├── docker-compose.yml
├── .env.example
└── README.md
```

## CI/CD Minimums

Every project should have at minimum these pipeline stages:

| Stage | Required | Purpose |
|-------|----------|---------|
| **Lint** | Yes | Catch style issues and potential bugs early |
| **Type check** | Yes (if typed language) | Catch type errors before runtime |
| **Test** | Yes | Run unit and integration tests |
| **Build** | Yes | Verify the project compiles and bundles correctly |

### Advanced (add when needed)

| Stage | When to Add | Purpose |
|-------|-------------|---------|
| **Security scanning** | Before first deploy | Dependency vulnerability scanning (Dependabot, Snyk, trivy) |
| **Docker build** | When containerizing | Verify Docker image builds and passes health check |
| **Deploy** | When hosting is configured | Push to staging or production |

## Config File Checklist

| File | Include? | Purpose |
|------|----------|---------|
| `.gitignore` | Yes | Prevent committing build artifacts, secrets, and dependencies |
| `.editorconfig` | Yes | Consistent formatting across editors and contributors |
| `Dockerfile` | If deploying | Containerized builds for consistent environments |
| `docker-compose.yml` | If multi-service | Local development with databases, caches, queues |
| `.env.example` | If env vars needed | Document required environment variables without secrets |
| `.pre-commit-config.yaml` | Recommended | Automated linting and formatting on every commit |
| `Makefile` / `justfile` | Recommended | Common commands in one place (test, lint, build, deploy) |

## Anti-Patterns to Avoid

1. **Over-scaffolding** — Do not create directories or files you will not need in the first month. An empty `microservices/` folder is a lie. Start with what you need and add structure as the codebase grows.
2. **Premature abstraction** — Do not create `interfaces/`, `factories/`, or `adapters/` directories at project start. These patterns emerge from real needs, not from project templates.
3. **Monorepo for solo dev** — A single repository with `apps/`, `packages/`, and `libs/` adds tooling complexity (turborepo, nx, workspaces) that a solo developer does not need. Use a simple flat repo unless you have multiple deployable services.
4. **Custom build scripts** — Do not write shell scripts for tasks that your package manager or framework handles natively. Use `npm scripts`, `pyproject.toml` scripts, `go generate`, or `Makefile` targets instead of bespoke `scripts/build.sh` files.
