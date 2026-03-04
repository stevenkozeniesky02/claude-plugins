---
name: dep-patterns
description: Dependency management best practices, version pinning strategies, license compatibility matrix, and maintenance schedules for keeping project dependencies healthy. Use when auditing or managing dependencies.
---

# Dependency Patterns

Best practices for managing dependencies and keeping them healthy.

## When to Use This Skill

- Running the `/checkup` command
- Adding new dependencies to a project
- Planning a dependency update cycle
- Resolving license compatibility questions

## Version Pinning Strategies

| Strategy | Syntax | When to Use | Risk |
|----------|--------|------------|------|
| Exact | `1.2.3` | Production critical deps | Low (no surprises) |
| Patch range | `~1.2.3` (>=1.2.3 <1.3.0) | Most dependencies | Low (bug fixes only) |
| Minor range | `^1.2.3` (>=1.2.3 <2.0.0) | Trusted, stable packages | Medium (new features) |
| Latest | `*` or `latest` | Never in production | High (anything can change) |

**Recommendation for solo devs:** Use `^` (caret/minor range) for most deps, exact pins for critical deps (database drivers, auth libraries).

## License Compatibility Matrix

```
Your Project License → Can use these dependency licenses:

MIT/ISC/BSD     → MIT, ISC, BSD, Apache-2.0, Unlicense, CC0
Apache-2.0      → MIT, ISC, BSD, Apache-2.0, Unlicense, CC0
GPL-3.0         → MIT, ISC, BSD, Apache-2.0, GPL-2.0+, GPL-3.0, LGPL, AGPL
AGPL-3.0        → Everything GPL can + AGPL
Proprietary     → MIT, ISC, BSD, Apache-2.0, Unlicense, CC0 (NOT GPL/AGPL)
```

### License Risk Levels

| License | Risk for Proprietary | Risk for Open Source | Notes |
|---------|---------------------|---------------------|-------|
| MIT | None | None | Most permissive |
| ISC | None | None | Simplified MIT |
| BSD-2/3 | None | None | Similar to MIT |
| Apache-2.0 | None | None | Includes patent grant |
| MPL-2.0 | Low | None | File-level copyleft |
| LGPL-2.1/3.0 | Medium | None | OK if dynamically linked |
| GPL-2.0/3.0 | **High** | None (if GPL-compatible) | Must open-source derivative works |
| AGPL-3.0 | **Critical** | None (if AGPL-compatible) | Network use triggers copyleft |
| SSPL | **Critical** | **High** | Not OSI-approved |
| No license | **High** | **High** | No permission to use |

## Dependency Health Indicators

### Healthy Dependency Signals

| Signal | What to Check |
|--------|--------------|
| Active maintenance | Commits within last 3 months |
| Responsive maintainer | Issues replied to within 2 weeks |
| Regular releases | At least 2 releases per year |
| Good documentation | README, API docs, changelog |
| Test coverage | CI badges, test suite visible |
| Download count | Growing or stable trend |

### Red Flags (Consider Replacing)

| Red Flag | What It Means |
|----------|--------------|
| No commits in 12+ months | Likely abandoned |
| 100+ open issues, no replies | Maintainer overwhelmed or gone |
| No CI/CD | Quality not verified |
| Single maintainer + no bus factor | Risk of sudden abandonment |
| Deprecated notice | Official end of life |
| Known vuln with no fix for 90+ days | Security risk |

## Common Dependency Anti-Patterns

1. **The Kitchen Sink** -- Installing a massive library for one function. Use: `lodash.get` instead of `lodash`, or just write the function.
2. **The Phantom Dependency** -- Importing a package that isn't in package.json (works because a transitive dep installs it). Breaks when that transitive dep updates.
3. **The Pinned Fossil** -- Exact-pinning everything and never updating. Accumulates vulnerabilities over time.
4. **The Bleeding Edge** -- Always using latest/canary versions. Breaks randomly.
5. **The Duplicate Utility** -- Having both `axios` and `node-fetch`, both `moment` and `dayjs`. Pick one.
6. **The Dev Leak** -- Dev dependencies imported in production code. Ships unnecessary code.

## Maintenance Schedule for Solo Devs

| Frequency | Action | Time |
|-----------|--------|------|
| After each coding session | `npm audit` / `pip-audit` | 1 min |
| Weekly | Review Dependabot/Renovate PRs | 10 min |
| Monthly | Run `/checkup`, apply safe updates | 30 min |
| Quarterly | Review major version updates, check for abandoned deps | 1 hour |
| After security advisories | Immediate patch for affected deps | Varies |

## Update Strategy Decision Tree

```
Is it a security patch?
  YES → Update immediately
  NO → Is it a patch version?
    YES → Update now (safe)
    NO → Is it a minor version?
      YES → Read changelog, update if no concerns
      NO → Major version?
        YES → Read migration guide
          → Breaking changes affect your code?
            YES → Schedule migration sprint
            NO → Update with integration tests
```
