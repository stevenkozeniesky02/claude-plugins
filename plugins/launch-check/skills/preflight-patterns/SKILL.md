---
name: preflight-patterns
description: Production readiness checklists, severity classification frameworks, and industry benchmarks for launch audits. Use when assessing whether a codebase is ready for production deployment.
---

# Preflight Patterns

Comprehensive frameworks for assessing production readiness.

## When to Use This Skill

- Running the `/preflight` command
- Manually auditing a codebase for production readiness
- Creating a launch checklist for a new product
- Reviewing operational readiness before a major release

## Master Preflight Checklist (50+ Items)

### Security (14 checks)

| # | Check | Severity | How to Verify |
|---|-------|----------|---------------|
| S1 | No hardcoded secrets in source | Critical | Grep for API keys, passwords, tokens |
| S2 | .env in .gitignore | Critical | Check .gitignore for .env entries |
| S3 | Authentication on protected routes | Critical | Review route middleware |
| S4 | Password hashing (bcrypt/argon2) | Critical | Check auth implementation |
| S5 | Input validation on all endpoints | High | Review request handlers |
| S6 | SQL injection prevention | High | Check for parameterized queries |
| S7 | XSS prevention | High | Check output encoding |
| S8 | CORS configured restrictively | High | Review CORS middleware config |
| S9 | Security headers set (CSP, HSTS, etc.) | Medium | Check middleware/server config |
| S10 | Dependency vulnerability scan clean | High | Run npm audit / pip-audit |
| S11 | Rate limiting on auth endpoints | Medium | Check rate limiter middleware |
| S12 | HTTPS enforced | High | Check redirect and cookie flags |
| S13 | JWT expiration configured | Medium | Check token TTL settings |
| S14 | File upload validation | Medium | Check upload handlers |

### Ops (12 checks)

| # | Check | Severity | How to Verify |
|---|-------|----------|---------------|
| O1 | CI/CD pipeline exists and passes | High | Check .github/workflows/ etc. |
| O2 | Tests run in CI | High | Check CI config for test step |
| O3 | Linting runs in CI | Medium | Check CI config for lint step |
| O4 | Dockerfile exists and builds | Medium | Try docker build |
| O5 | .dockerignore exists | Low | Check for file |
| O6 | Health check endpoint exists | High | Look for /health route |
| O7 | .env.example documents all vars | High | Compare .env.example to code |
| O8 | Env var validation at startup | Medium | Check config/startup code |
| O9 | Deployment config exists | Medium | Check for platform config |
| O10 | Rollback strategy documented | Medium | Check deployment docs |
| O11 | Database migrations versioned | Medium | Check migration files |
| O12 | Seed data or fixtures available | Low | Check for seed scripts |

### Reliability (13 checks)

| # | Check | Severity | How to Verify |
|---|-------|----------|---------------|
| R1 | Global error handler exists | High | Check app-level error middleware |
| R2 | Async errors caught | High | Check for unhandled rejections |
| R3 | No empty catch blocks | Medium | Grep for empty catch/except |
| R4 | HTTP client timeouts set | High | Check fetch/axios/requests config |
| R5 | Database query timeouts | Medium | Check DB client config |
| R6 | Retry logic on external calls | Medium | Check HTTP client wrappers |
| R7 | Graceful shutdown handler | High | Check SIGTERM/SIGINT handling |
| R8 | Connection pool cleanup | Medium | Check shutdown hooks |
| R9 | Structured logging configured | Medium | Check logger setup |
| R10 | No sensitive data in logs | High | Grep logs for password/token |
| R11 | Error responses are consistent | Medium | Check error response format |
| R12 | Request ID / correlation ID | Low | Check middleware for request IDs |
| R13 | Circuit breaker on critical paths | Low | Check for circuit breaker patterns |

### Documentation (13 checks)

| # | Check | Severity | How to Verify |
|---|-------|----------|---------------|
| D1 | README exists | High | Check for README.md |
| D2 | README has project description | Medium | Read README content |
| D3 | README has quick start | High | Check for setup instructions |
| D4 | README lists prerequisites | Medium | Check for requirements section |
| D5 | README lists available commands | Medium | Check for commands/scripts section |
| D6 | API docs exist (if API project) | Medium | Check for OpenAPI/Swagger |
| D7 | Setup guide is complete | Medium | Follow guide mentally |
| D8 | .env.example has descriptions | Medium | Check for commented env vars |
| D9 | Architecture overview exists | Low | Check for architecture doc |
| D10 | CHANGELOG exists | Low | Check for CHANGELOG.md |
| D11 | LICENSE file exists | Medium | Check for LICENSE |
| D12 | Contributing guide exists | Low | Check for CONTRIBUTING.md |
| D13 | Public functions documented | Low | Sample check for docstrings |

## Severity Classification

| Severity | Definition | Action |
|----------|-----------|--------|
| Critical | Could cause data breach, data loss, or complete system failure | Must fix before launch. No exceptions. |
| High | Significant operational or security risk | Must fix before launch unless documented exception exists. |
| Medium | Quality or reliability concern | Should fix before launch; acceptable to defer with documented plan. |
| Low | Best practice improvement | Can launch without; add to backlog. |

## Verdict Decision Framework

| Verdict | Criteria |
|---------|----------|
| **GO** | Zero FAIL findings. All checks PASS or WARN only. No critical or high issues. |
| **CONDITIONAL GO** | FAIL findings exist but none are Critical severity. All Critical checks pass. A documented plan exists to fix remaining issues within 2 weeks. |
| **NO-GO** | Any Critical severity FAIL. Multiple High severity FAILs in the same category. Security category overall FAIL. |

## Common Launch Blockers (Solo Dev Edition)

These are the issues most commonly found in solo dev projects:

1. **Hardcoded secrets in git history** -- Even if removed from current code, they're in git history
2. **No error handling on external API calls** -- First timeout = unhandled crash
3. **No health check endpoint** -- Platform can't determine if app is alive
4. **Missing .env.example** -- No one (including future you) knows what env vars are needed
5. **No CI pipeline** -- Manual testing means manual mistakes
6. **No graceful shutdown** -- Deployments kill in-flight requests
7. **Console.log as logging** -- No structured logs, no log levels, no way to debug production issues
8. **No rate limiting** -- One bot = denial of service
