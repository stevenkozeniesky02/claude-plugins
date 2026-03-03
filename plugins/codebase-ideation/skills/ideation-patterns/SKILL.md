---
name: ideation-patterns
description: Patterns and techniques for effective codebase analysis and improvement idea generation. Use when scanning projects for opportunities or when refining raw ideas into actionable cards.
---

# Ideation Patterns

Techniques for discovering high-value improvement opportunities in any codebase.

## When to Use This Skill

- Running the `/ideation:discover` command
- Manually scanning a project for improvement ideas
- Refining raw ideas into actionable cards
- Prioritizing a backlog of improvement opportunities

## Idea Quality Checklist

A good idea card must pass all of these:

- [ ] **Specific** — References actual files, modules, or code patterns (not "improve performance")
- [ ] **Actionable** — A developer could start implementing it without further research
- [ ] **Scoped** — Has a clear boundary (not "refactor the entire codebase")
- [ ] **Justified** — Explains WHY it matters, not just WHAT to do
- [ ] **Sized** — Complexity estimate is realistic for this codebase

## Priority Framework

| Priority | Criteria | Examples |
|----------|----------|---------|
| P1 | Blocks other work, fixes real user pain, security vulnerability | Missing auth, broken CI, data loss risk |
| P2 | Improves quality, reduces confusion, prevents future problems | Tech debt, missing docs, hardening |
| P3 | Nice-to-have, polish, optimization | Code style, minor perf gains, feature ideas |

## Complexity Estimation

| Size | Criteria |
|------|----------|
| S | Single file change, < 50 lines, no new dependencies |
| M | 2-5 files, < 200 lines, may add a dependency |
| L | 5+ files, architectural change, new module or system |

## Common Anti-Patterns in Ideation

- **Generic advice** — "Add more tests" instead of "Add integration tests for the payment webhook handler in src/webhooks/stripe.py"
- **Premature optimization** — Suggesting caching for an endpoint that gets 10 requests/day
- **Scope creep** — "Rewrite the auth system" when "Add rate limiting to /api/login" is the real need
- **Ignoring context** — Suggesting React patterns for a Vue project
- **Duplicate ideas** — Same underlying issue surfaced by multiple analysts without deduplication

## Category Discovery Heuristics

When looking for categories beyond the fixed set, check for:

- **DevOps** — If the project has CI/CD configs, Dockerfiles, or deployment scripts
- **Testing** — If test coverage is low or test patterns are inconsistent
- **Observability** — If the project has logging but no metrics, tracing, or dashboards
- **Developer Experience** — If the project has complex setup, missing scripts, or poor error messages
- **Accessibility** — If the project has a frontend with no a11y testing
- **Internationalization** — If the project has user-facing strings but no i18n framework
- **API Design** — If the project exposes an API with inconsistent conventions
