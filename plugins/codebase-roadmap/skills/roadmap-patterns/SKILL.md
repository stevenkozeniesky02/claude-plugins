---
name: roadmap-patterns
description: Patterns and techniques for creating effective multi-phase project roadmaps with dependency tracking, milestone prioritization, and phase exit criteria.
---

# Roadmap Patterns

Techniques for building actionable project roadmaps from codebase analysis.

## When to Use This Skill

- Running the `/roadmap:generate` command
- Manually planning a project roadmap
- Prioritizing milestones across multiple domains
- Defining phase boundaries and exit criteria

## Maturity Model

| Level | Indicators | Typical Phase Goal |
|-------|-----------|-------------------|
| Greenfield | < 5 source files, no tests | Foundation |
| Prototype | Working code, no tests, no CI | Core + Testing |
| MVP | Core features, some tests, basic docs | Production Readiness |
| Production | Good tests, CI/CD, monitoring | Scale + Polish |
| Mature | Comprehensive coverage, CD, observability | Optimization + Innovation |

## Phase Design Principles

1. **Each phase has ONE goal** — "Production Readiness", not "Fix everything"
2. **Exit criteria are measurable** — "Test coverage > 80%" not "Better tests"
3. **Milestones within a phase are parallelizable** — If they depend on each other, they're in different phases
4. **Near-term phases are detailed, far phases are directional** — Don't plan Task-level detail for Phase 5
5. **3-6 phases is typical** — Fewer = too coarse, more = too granular

## Dependency Tracking

### Cross-Domain Dependencies (Common Patterns)

- Backend Auth API → Frontend Login UI
- Database Schema → Backend API → Frontend Data Display
- Containerization → CI/CD Pipeline → Production Deploy
- Secrets Management → Production Deploy
- Monitoring Setup → Production Operations

### Dependency Anti-Patterns

- **False dependencies** — "Frontend needs backend" when frontend could use mocks
- **Hidden dependencies** — Forgetting that security milestones block production
- **Circular dependencies** — Phase 2 needs Phase 3 which needs Phase 2

## Milestone Quality Checklist

- [ ] Has a clear, specific title (not "Improve backend")
- [ ] Priority is justified (P1 = blocks other work or launch)
- [ ] Dependencies are explicit and reference other milestones by name
- [ ] Effort estimate is realistic for the project's team size
- [ ] Tasks are specific enough to estimate individually
- [ ] Maturity impact is stated (what level does this enable?)

## Greenfield vs. Existing Project

### Greenfield Roadmap Shape
```
Phase 1: Foundation (project scaffold, core architecture, CI)
Phase 2: Core Features (primary user flows, data model)
Phase 3: Quality & Polish (tests, error handling, docs)
Phase 4: Launch Prep (deployment, monitoring, security hardening)
```

### Existing Project Roadmap Shape
```
Phase 1: Stabilize (fix critical issues, add missing tests)
Phase 2: Harden (security, performance, observability)
Phase 3: Extend (new features, integrations)
Phase 4: Scale (infrastructure, optimization)
```

## Estimation Heuristics

| Effort | Solo Developer | Small Team (2-4) |
|--------|---------------|------------------|
| S | 1-2 days | 0.5-1 day |
| M | 3-5 days | 1-3 days |
| L | 1-3 weeks | 1-2 weeks |
