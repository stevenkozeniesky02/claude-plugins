---
name: build-patterns
description: TDD workflow patterns, commit conventions, milestone decomposition strategies, and implementation best practices. Use when implementing features from a roadmap or following test-driven development workflows.
---

# Build Patterns

TDD workflow patterns, commit conventions, milestone decomposition strategies, and implementation best practices for building software feature-by-feature.

## When to Use This Skill

- Running the `/build` command to implement roadmap milestones
- Following TDD (test-driven development) workflow for any feature implementation
- Breaking down large milestones into manageable implementation tasks
- Deciding commit boundaries and message conventions during iterative development

## TDD Workflow (Red-Green-Refactor)

| Phase | Action | Goal | Done When |
|-------|--------|------|-----------|
| **Red** | Write a failing test | Define expected behavior | Test runs and FAILS because the feature does not exist |
| **Green** | Write minimal code | Make the test pass | Test runs and PASSES with the simplest possible implementation |
| **Refactor** | Clean up | Improve code quality | All tests still pass after cleanup |

### Critical TDD Rules

- **NEVER write implementation before tests** — The test defines the contract. Writing code first means you test what you built, not what you need.
- **NEVER write more implementation than tests require** — If the test does not check for error handling, do not add error handling. Add a test first, then add the handling.
- **NEVER skip "verify it fails"** — A test that passes before implementation proves nothing. It either tests the wrong thing or tests something that already exists.

## Milestone Decomposition Strategy

Break work into a clear hierarchy:

```
Milestone (hours to days)
└── Task (15-45 minutes)
    └── Test + Implementation (5-15 minutes per test case)
```

### Good Task Examples

| Task | Why It's Good |
|------|--------------|
| "Add User model with email and password fields" | Single responsibility, testable, foundational |
| "Add POST /auth/login endpoint returning JWT" | Clear input/output, independently testable |
| "Add rate limiter middleware (10 req/min)" | Isolated concern, measurable acceptance criteria |
| "Add email validation to registration" | Small scope, specific behavior to verify |

### Bad Task Examples

| Task | Why It's Bad | Fix |
|------|-------------|-----|
| "Build the auth system" | Too large, multiple features | Split into model, login, registration, middleware |
| "Refactor for performance" | No acceptance criteria, scope undefined | Identify specific bottleneck, write benchmark test |
| "Set up project" | Already done by scaffold-kit | Remove — do not include setup tasks in build plans |
| "Fix bugs" | Not a task — which bugs? | Identify specific bug, write failing test, fix |

## Task Ordering Rules

Within a milestone, always order tasks in this sequence:

1. **Data models first** — Define schemas, types, and database tables before anything references them
2. **Core logic second** — Business rules, validation, transformations that operate on the models
3. **API/routes third** — Endpoints, handlers, controllers that expose the core logic
4. **Integration last** — Connecting services, webhooks, third-party APIs, end-to-end flows

This ordering minimizes rework because each layer depends only on layers below it.

## Commit Conventions

Use conventional commit prefixes:

| Prefix | When to Use | Example |
|--------|-------------|---------|
| `feat:` | New functionality | `feat: add user registration endpoint` |
| `fix:` | Bug fix | `fix: handle null email in login validation` |
| `test:` | Test-only changes | `test: add edge case tests for rate limiter` |
| `refactor:` | Code improvement, no behavior change | `refactor: extract JWT logic into auth service` |
| `docs:` | Documentation only | `docs: add API endpoint documentation` |

### Commit Frequency

- **Commit after each completed task** within a milestone — not after each file, not after each test
- **Commit after milestone completion** with a summary commit message: `feat: complete milestone [N] - [name]`
- **Never leave uncommitted work** between milestones or between sessions

## When to Stop and Ask

Stop implementation and ask the user for guidance in these scenarios:

1. **Design flaw discovered** — The tests reveal that the planned approach will not work. The architecture needs to change before continuing.
2. **Out-of-scope files need changes** — The task requires modifying files not listed in the plan. Get approval before touching unplanned code.
3. **Missing dependency** — The task requires a library, service, or API that is not available. Do not add dependencies without approval.
4. **2x time overrun** — If a 30-minute task has consumed over an hour of iteration, something is wrong. Step back and reassess.
5. **Tests pass but something feels wrong** — The implementation satisfies the tests but the approach is fragile, hacky, or will cause problems later. Flag it before moving on.

## Anti-Patterns

1. **Gold plating** — Adding features, error handling, or abstractions that no test requires. If the test does not ask for it, do not build it. Add a test first if the behavior is needed.
2. **Test after** — Writing implementation first, then writing tests that confirm what you already built. This is confirmation bias, not verification. Tests written after implementation rarely catch bugs.
3. **Big bang commit** — Implementing an entire milestone before committing. If something breaks, you cannot isolate the cause. Commit after each task.
4. **Ignoring failing tests** — Marking tests as skipped or commenting them out to make the suite pass. Failing tests are information — fix them or delete them, never silence them.
5. **Premature refactoring** — Restructuring code during the green phase before all tests pass. Get to green first, then refactor. Refactoring broken code wastes effort because you do not know what "working" looks like yet.
