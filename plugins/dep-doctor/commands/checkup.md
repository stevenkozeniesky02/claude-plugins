---
description: "Perform a dependency health audit with CVE scanning, freshness checks, license auditing, and bloat detection"
argument-hint: "[--scope runtime|dev|all] [--fix]"
---

# Dependency Doctor

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.depdoctor/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.depdoctor/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress dependency checkup:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify dependency manifest exists

Confirm the working directory contains a dependency manifest (package.json, requirements.txt, pyproject.toml, go.mod, Cargo.toml, Gemfile, pom.xml, build.gradle). If no manifest is found, inform the user and halt.

### 3. Initialize state

Create `.depdoctor/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "scope": "all",
    "fix": false
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--scope` (runtime, dev, or all dependencies) and `--fix` (attempt to auto-fix safe upgrades).

---

## Phase 0: Dependency Inventory (Sequential)

Scan the project to catalog all dependencies, their versions, and lock file status.

**Output file:** `.depdoctor/00-dependency-inventory.md`

```markdown
# Dependency Inventory

## Overview
- **Package manager:** [npm/pip/cargo/go mod/etc.]
- **Manifest file:** [package.json/pyproject.toml/etc.]
- **Lock file:** [Present/Missing]
- **Total dependencies:** [N]
- **Runtime dependencies:** [N]
- **Dev dependencies:** [N]
- **Transitive dependencies:** [N estimated from lock file]

## Direct Dependencies

| Package | Version | Type | Pinning |
|---------|---------|------|---------|
| [name] | [version] | runtime/dev | exact/range/unpinned |

## Scope
[all / runtime only / dev only per --scope flag]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.depdoctor/00-dependency-inventory.md` to load the dependency context.

Launch ALL analysis agents in parallel using multiple Agent tool calls in a single message.

### 1a. Vulnerability Scanner

```
Agent:
  subagent_type: "vuln-scanner"
  description: "Scan dependencies for known CVEs"
  prompt: |
    Scan all dependencies in this project for known security vulnerabilities.

    ## Dependency Inventory
    [Insert full contents of .depdoctor/00-dependency-inventory.md]

    Follow your analysis strategy. Check for CVEs using npm audit, pip-audit, or equivalent tools.
    Output in your defined format.
```

### 1b. Freshness Checker

```
Agent:
  subagent_type: "freshness-checker"
  description: "Check dependency versions and identify outdated packages"
  prompt: |
    Check all dependencies for freshness and identify outdated packages.

    ## Dependency Inventory
    [Insert full contents of .depdoctor/00-dependency-inventory.md]

    Follow your analysis strategy. Identify outdated deps, check for breaking changes, and flag abandoned packages.
    Output in your defined format.
```

### 1c. License Auditor

```
Agent:
  subagent_type: "license-auditor"
  description: "Audit dependency licenses for compatibility"
  prompt: |
    Audit all dependency licenses for compatibility and compliance risks.

    ## Dependency Inventory
    [Insert full contents of .depdoctor/00-dependency-inventory.md]

    Follow your analysis strategy. Check license types, flag copyleft in proprietary projects, identify unknown licenses.
    Output in your defined format.
```

### 1d. Bloat Detector

```
Agent:
  subagent_type: "bloat-detector"
  description: "Find unused and bloated dependencies"
  prompt: |
    Find unused dependencies, transitive bloat, and duplicate packages in this project.

    ## Dependency Inventory
    [Insert full contents of .depdoctor/00-dependency-inventory.md]

    Follow your analysis strategy. Identify deps that are imported nowhere, duplicates, and heavy transitive trees.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.depdoctor/01-vulnerabilities.md`
- `.depdoctor/01-freshness.md`
- `.depdoctor/01-licenses.md`
- `.depdoctor/01-bloat.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Health Report Synthesis

Read ALL `.depdoctor/01-*.md` files plus `.depdoctor/00-dependency-inventory.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "dep-synthesizer"
  description: "Synthesize findings into a dependency health scorecard"
  prompt: |
    Synthesize all dependency health findings into a prioritized health report.

    ## Dependency Inventory
    [Insert contents of .depdoctor/00-dependency-inventory.md]

    ## Vulnerabilities
    [Insert contents of .depdoctor/01-vulnerabilities.md]

    ## Freshness
    [Insert contents of .depdoctor/01-freshness.md]

    ## Licenses
    [Insert contents of .depdoctor/01-licenses.md]

    ## Bloat
    [Insert contents of .depdoctor/01-bloat.md]

    Auto-fix mode: [value of --fix flag]

    Follow your synthesis strategy. Produce the final dependency health report in your defined output format.
```

Save output to `.depdoctor/report.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Dependency checkup complete!

## Health Grade: [A / B / C / D / F]
[2-3 sentence summary]

## Key Findings
- Vulnerabilities: [N] (Critical: [N], High: [N], Medium: [N])
- Outdated packages: [N] (Major: [N], Minor: [N], Patch: [N])
- License issues: [N]
- Unused dependencies: [N]
- Bloated dependencies: [N]

## Top 3 Priority Actions
1. [Action 1] -- [urgency]
2. [Action 2] -- [urgency]
3. [Action 3] -- [urgency]

## Output
- Dependency inventory: .depdoctor/00-dependency-inventory.md
- Vulnerabilities: .depdoctor/01-vulnerabilities.md
- Freshness: .depdoctor/01-freshness.md
- Licenses: .depdoctor/01-licenses.md
- Bloat: .depdoctor/01-bloat.md
- Full report: .depdoctor/report.md

## Next Step
Address critical vulnerabilities first, then remove unused dependencies. Re-run `/checkup` after changes.
```
