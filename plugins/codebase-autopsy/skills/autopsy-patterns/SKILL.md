---
name: autopsy-patterns
description: Tech debt classification frameworks, risk scoring methodologies, refactoring prioritization strategies, and complexity thresholds for codebase health assessment. Use when analyzing tech debt or planning refactoring work.
---

# Autopsy Patterns

Frameworks for classifying, scoring, and prioritizing tech debt.

## When to Use This Skill

- Running the `/autopsy` command
- Manually assessing tech debt in a codebase
- Planning refactoring work
- Scoring risk of code issues

## Tech Debt Classification

| Type | Description | Examples | Typical Severity |
|------|-------------|----------|-----------------|
| **Deliberate-Prudent** | Known shortcuts taken consciously with a plan to fix | "Ship now, refactor in Sprint 3" | Medium (if tracked) |
| **Deliberate-Reckless** | Known shortcuts with no plan to fix | "We don't have time for tests" | High |
| **Inadvertent-Prudent** | Learned a better approach after building | "Now I know we should have used X" | Medium |
| **Inadvertent-Reckless** | Didn't know enough to do it right | Copy-paste from Stack Overflow without understanding | High |

Source: Martin Fowler's Tech Debt Quadrant

## Detonation Risk Scoring Framework

### Blast Radius Scale

| Score | Radius | Description |
|-------|--------|-------------|
| 1-2 | Single function | Bug affects one function, easy to isolate |
| 3-4 | Single module | Bug affects one module, limited blast |
| 5-6 | Cross-module | Bug propagates across module boundaries |
| 7-8 | System-wide | Bug affects entire application behavior |
| 9-10 | Data/Security | Data loss, corruption, or security breach |

### Probability Scale

| Score | Likelihood | Description |
|-------|-----------|-------------|
| 1-2 | Theoretical | Could happen but requires unusual conditions |
| 3-4 | Edge case | Triggers under specific, uncommon conditions |
| 5-6 | Moderate | Triggers under normal but infrequent operations |
| 7-8 | Likely | Will trigger under normal growth or usage patterns |
| 9-10 | Certain | Already triggering or mathematically inevitable |

### Detonation Score = Blast Radius x Probability (1-100)

| Score Range | Classification | Action |
|-------------|---------------|--------|
| 50-100 | Critical | Fix immediately, before any feature work |
| 25-49 | High | Fix this sprint |
| 10-24 | Medium | Schedule within 2-4 weeks |
| 1-9 | Low | Add to backlog |

## Complexity Thresholds

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| File length | < 300 lines | 300-500 lines | > 500 lines |
| Function length | < 30 lines | 30-50 lines | > 50 lines |
| Nesting depth | < 3 levels | 3-4 levels | > 4 levels |
| Cyclomatic complexity | < 10 | 10-20 | > 20 |
| Module coupling (imports) | < 5 | 5-10 | > 10 |
| Parameters per function | < 4 | 4-6 | > 6 |

## Refactoring Prioritization Matrix

Prioritize by: **(Impact / Effort) x Urgency**

| Priority | Impact | Effort | Urgency | Example |
|----------|--------|--------|---------|---------|
| P0 | High | Low | Now | Hardcoded secret in source |
| P1 | High | Medium | Soon | Missing error handling on payment endpoint |
| P2 | High | High | Soon | Abandoned TypeScript migration |
| P3 | Medium | Low | Flexible | Extract duplicated utility function |
| P4 | Medium | Medium | Flexible | Standardize logging pattern |
| P5 | Low | Any | Backlog | Rename inconsistent variables |

## Common Tech Debt Patterns

1. **The TODO Graveyard** -- TODOs older than 6 months that nobody will ever fix. Convert to issues or delete.
2. **The Copy-Paste Plague** -- Same logic in 3+ places. One gets updated, others don't. Extract to shared function.
3. **The God File** -- One file that does everything. Over 500 lines, impossible to test in isolation. Split by responsibility.
4. **The Zombie Feature** -- Feature flag that's been "temporary" for 6 months. Clean up or make permanent.
5. **The Invisible Dependency** -- Module A depends on Module B's internal implementation details. Breaks when B changes.
6. **The Half-Migration** -- Started migrating from X to Y, got 60% done, moved on. Worst of both worlds.
7. **The Silent Failure** -- Error caught and swallowed. System looks fine but data is corrupted or operations are lost.
