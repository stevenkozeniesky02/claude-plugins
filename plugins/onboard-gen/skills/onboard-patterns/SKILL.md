---
name: onboard-patterns
description: Documentation structure templates, architecture diagram conventions, onboarding checklist frameworks, and technical writing best practices for developer documentation. Use when generating or reviewing onboarding documentation.
---

# Onboard Patterns

Frameworks and best practices for creating effective developer onboarding documentation.

## When to Use This Skill

- Running the `/onboard` command
- Manually creating onboarding documentation
- Reviewing existing documentation quality
- Planning a documentation improvement initiative

## Onboarding Documentation Hierarchy

The most effective onboarding docs follow this progressive disclosure pattern:

| Level | Document | Time to Read | Purpose |
|-------|----------|-------------|---------|
| 1. Do | Quick Start / Setup Guide | 5 min | Get the code running |
| 2. See | Architecture Overview | 10 min | Understand the big picture |
| 3. Understand | Flow Walkthroughs | 15 min | See how code actually works |
| 4. Match | Coding Conventions | 10 min | Write code that fits |
| 5. Reference | API Docs / Quick Reference | Ongoing | Look up specifics |

**Rule:** New developers should be able to make their first PR within 2 hours of starting the onboarding guide.

## Architecture Diagram Conventions (ASCII)

Use these box-drawing characters for consistent ASCII diagrams:

```
Boxes:     ┌──────────┐
           │  Module  │
           └──────────┘

Arrows:    ──── (horizontal)
           │    (vertical)
           ├──  (branch right)
           └──  (branch right, last)
           ↓ ↑ → ← (directional)

Grouping:  ┌─────────────────────┐
           │  Group Name         │
           │  ┌────┐  ┌────┐    │
           │  │ A  │  │ B  │    │
           │  └────┘  └────┘    │
           └─────────────────────┘
```

## Flow Trace Template

```
[Action/Trigger]
  → File: path/to/file.ts
    Function: handlerName()
    Input: { description of input }
    Action: [what happens]
  → File: path/to/service.ts
    Function: processData()
    Action: [what happens]
    Side effect: [database write, API call, etc.]
  → Output: { description of output }
    Status: [HTTP status or result]
```

## Setup Guide Quality Checklist

| Check | Required | Description |
|-------|----------|-------------|
| Prerequisites listed with versions | Yes | Exact versions from project config |
| Clone and install in < 3 commands | Yes | No unnecessary steps |
| Environment variables documented | Yes | Every required var with example |
| Database setup automated | Yes | Single command (docker-compose or script) |
| Verification step included | Yes | How to confirm setup works |
| Common errors addressed | Yes | Top 3 troubleshooting entries |
| Time estimate provided | No | "This takes ~10 minutes" |
| Platform-specific notes | No | macOS vs Linux vs Windows if relevant |

## Convention Documentation Framework

Document conventions in this order (most impactful first):

1. **Naming conventions** -- Variables, functions, files, directories, database tables
2. **File organization** -- Where new files go, module structure pattern
3. **Import ordering** -- How imports should be organized
4. **Error handling** -- How to create, throw, catch, and return errors
5. **API patterns** -- Route definition, response format, validation
6. **Testing patterns** -- File naming, test structure, mocking approach
7. **Git conventions** -- Branch naming, commit messages, PR process

## Technical Writing Rules for Onboarding Docs

1. **Show, don't tell** -- Code examples > prose descriptions
2. **Use the second person** -- "You can run tests with..." not "Tests can be run with..."
3. **One command per step** -- Don't chain 5 commands in one bash block
4. **Explain WHY, not just HOW** -- "We use Prisma because it provides type-safe queries" not just "We use Prisma"
5. **Include expected output** -- After every command, show what success looks like
6. **Link, don't repeat** -- Reference detailed docs instead of duplicating content
7. **Keep it current** -- Outdated docs are worse than no docs (add "last verified" dates)

## Common Documentation Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **The Novel** | 50-page README nobody reads | Keep main guide under 2000 words, link to details |
| **The Skeleton** | "TODO: add docs" | Generate from code -- that's what /onboard does |
| **The Fossil** | Documentation from 2 years ago | Add "last verified" dates, automate generation |
| **The Treasure Hunt** | Docs scattered across 15 locations | Single onboarding guide with links to deep dives |
| **The Assumed Knowledge** | "Just run the thing" (no prerequisites) | List every prerequisite with version and install link |
