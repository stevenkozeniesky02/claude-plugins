---
name: testgap-patterns
description: Test prioritization frameworks, coverage-vs-confidence models, testing pyramid strategies, and risk-based test selection patterns. Use when analyzing test coverage or planning test improvement initiatives.
---

# Test Gap Patterns

Frameworks for prioritizing test coverage improvements and selecting the right test types.

## When to Use This Skill

- Running the `/coverage` command
- Planning a test improvement initiative
- Deciding what tests to write for a new module
- Reviewing test quality across a codebase

## The Testing Pyramid

```
         /  E2E  \        Few, slow, expensive
        /----------\
       / Integration \    Some, moderate speed
      /----------------\
     /    Unit Tests     \  Many, fast, cheap
    /----------------------\
```

| Level | Speed | Cost | Confidence | When to Use |
|-------|-------|------|-----------|------------|
| Unit | Fast (ms) | Low | Function-level | Pure logic, calculations, transformations |
| Integration | Medium (s) | Medium | Cross-boundary | DB queries, API calls, service interactions |
| E2E | Slow (min) | High | User-flow-level | Critical user journeys only |

**Rule of thumb:** 70% unit, 20% integration, 10% e2e.

## Risk-Based Test Prioritization

Prioritize testing by risk, not by coverage percentage.

### Risk Categories (Highest to Lowest)

| Priority | Category | Examples | Why High Risk |
|----------|----------|---------|--------------|
| P0 | Security | Auth, permissions, input validation | Bugs = breach |
| P1 | Money | Payments, billing, subscriptions | Bugs = revenue loss |
| P2 | Data integrity | Write operations, migrations, imports | Bugs = corruption |
| P3 | Core user flows | Sign up, main feature, checkout | Bugs = churn |
| P4 | Error handling | Catch blocks, fallbacks, retries | Bugs = silent failures |
| P5 | Edge cases | Boundary values, empty states, concurrency | Bugs = intermittent issues |
| P6 | Utilities | Helpers, formatters, constants | Bugs = minor annoyance |

### Coverage vs Confidence

Coverage percentage is misleading. Focus on confidence instead:

| Metric | What It Tells You | Limitation |
|--------|-------------------|-----------|
| Line coverage | Which lines execute during tests | Doesn't mean assertions verify behavior |
| Branch coverage | Which conditions are tested | Doesn't verify the right outcomes |
| **Behavior coverage** | Which behaviors have assertions | Best measure but hardest to measure |
| Mutation score | How many bugs tests catch | Most accurate but slowest to compute |

**100% line coverage with no assertions = 0% real coverage.**

## Test Quality Signals

### Good Test Signals

| Signal | Example |
|--------|---------|
| Tests describe behavior | `it('should reject expired tokens')` |
| Tests have specific assertions | `expect(result.status).toBe(401)` |
| Tests cover edge cases | Empty input, null, boundary values |
| Tests verify error messages | `expect(error.message).toContain('expired')` |
| Tests are independent | Each test sets up its own state |

### Bad Test Signals (Test Smells)

| Smell | Example | Problem |
|-------|---------|---------|
| No assertions | `it('should work', () => { fn() })` | Tests nothing |
| Tautological | `expect(true).toBe(true)` | Always passes |
| Test name mismatch | Name says one thing, test does another | Misleading |
| Excessive mocking | Mock everything, test the mock | Tests nothing real |
| Shared mutable state | Tests depend on execution order | Flaky |
| Magic numbers | `expect(result).toBe(42)` without context | Unclear intent |

## Common Test Gaps by Framework

### Web Applications (Express/Fastify/Django/Flask)

| Often Tested | Often Missed |
|-------------|-------------|
| Happy path routes | Error middleware |
| GET endpoints | Auth middleware |
| Model validation | Rate limiting |
| Basic CRUD | Concurrent writes |
| Response format | Timeout handling |

### Frontend Applications (React/Vue/Angular)

| Often Tested | Often Missed |
|-------------|-------------|
| Component renders | Loading states |
| Click handlers | Error boundaries |
| Form submission | Accessibility |
| API success path | Network failure |
| Desktop layout | Mobile responsive |

### CLI Applications

| Often Tested | Often Missed |
|-------------|-------------|
| Happy path commands | Invalid arguments |
| Main features | File permission errors |
| Output format | Interrupt handling (Ctrl+C) |
| Config loading | Large input handling |

## Test Strategy Decision Tree

```
Is it pure logic with no dependencies?
  YES → Unit test
  NO → Does it cross a system boundary (DB, API, filesystem)?
    YES → Does the boundary behavior matter for correctness?
      YES → Integration test
      NO → Unit test with mock
    NO → Does it involve user interaction flow?
      YES → Is it a critical user journey?
        YES → E2E test
        NO → Integration test
      NO → Unit test
```

## Effort Estimation Guide

| Test Type | Setup | Per Test | Dependencies |
|-----------|-------|---------|-------------|
| Unit (new file) | 15 min | 5-10 min | None |
| Unit (existing file) | 0 min | 5-10 min | None |
| Integration (with DB) | 30-60 min | 15-30 min | Test DB, migrations |
| Integration (with API) | 15-30 min | 10-20 min | Mock server or fixture |
| E2E | 1-2 hours | 30-60 min | Browser driver, test env |
| Property-based | 30 min | 15-30 min | Property testing library |
