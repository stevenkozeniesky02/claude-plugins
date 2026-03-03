---
description: "Run a production readiness audit with 50+ checks across security, ops, reliability, and documentation to produce a GO/NO-GO scorecard"
argument-hint: "[--strict] [--category security|ops|reliability|docs]"
---

# Launch Check

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.preflight/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.preflight/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress preflight session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify codebase exists

Confirm the working directory contains a codebase (look for package.json, pyproject.toml, go.mod, Cargo.toml, Makefile, or src/ directory). If no codebase is detected, inform the user and halt.

### 3. Initialize state

Create `.preflight/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "strict": false,
    "category": null
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--strict` (raises warnings to failures) and `--category` (run only one category of checks).

---

## Phase 0: Codebase Survey (Sequential)

Scan the codebase to understand its structure and tech stack. This gives agents the context they need.

**Output file:** `.preflight/00-codebase-survey.md`

```markdown
# Codebase Survey

## Tech Stack Detected
- **Language(s):** [detected languages]
- **Framework(s):** [detected frameworks]
- **Package manager:** [npm/pip/go mod/cargo/etc.]
- **Database:** [if detectable from configs or dependencies]

## Project Structure
- **Entry point(s):** [main files]
- **Source directories:** [src/, app/, lib/, etc.]
- **Test directories:** [tests/, __tests__/, spec/, etc.]
- **Config files:** [list of config files found]
- **CI/CD:** [.github/workflows/, .gitlab-ci.yml, etc.]
- **Docker:** [Dockerfile, docker-compose.yml, etc.]

## Dependency Summary
- **Total dependencies:** [count]
- **Dev dependencies:** [count]
- **Lock file present:** [yes/no]

## Documentation Found
- **README:** [yes/no]
- **API docs:** [yes/no]
- **Contributing guide:** [yes/no]
- **Changelog:** [yes/no]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Audits

Read `.preflight/00-codebase-survey.md` to load the codebase context.

Launch ALL checker agents in parallel using multiple Agent tool calls in a single message. If `--category` was specified, launch only the matching agent.

### 1a. Security Checker

```
Agent:
  subagent_type: "security-checker"
  description: "Audit security posture of the codebase"
  prompt: |
    Audit the security posture of this codebase for production readiness.

    ## Codebase Survey
    [Insert full contents of .preflight/00-codebase-survey.md]

    Follow your analysis strategy. Check every item in your security checklist.
    Output in your defined format.
```

### 1b. Ops Checker

```
Agent:
  subagent_type: "ops-checker"
  description: "Audit operational readiness of the codebase"
  prompt: |
    Audit the operational readiness of this codebase for production deployment.

    ## Codebase Survey
    [Insert full contents of .preflight/00-codebase-survey.md]

    Follow your analysis strategy. Check CI/CD, Docker, deployment, and configuration.
    Output in your defined format.
```

### 1c. Reliability Checker

```
Agent:
  subagent_type: "reliability-checker"
  description: "Audit reliability and error handling in the codebase"
  prompt: |
    Audit the reliability and error handling of this codebase for production readiness.

    ## Codebase Survey
    [Insert full contents of .preflight/00-codebase-survey.md]

    Follow your analysis strategy. Check error handling, logging, retries, and graceful degradation.
    Output in your defined format.
```

### 1d. Docs Checker

```
Agent:
  subagent_type: "docs-checker"
  description: "Audit documentation completeness"
  prompt: |
    Audit the documentation completeness of this codebase for production readiness.

    ## Codebase Survey
    [Insert full contents of .preflight/00-codebase-survey.md]

    Follow your analysis strategy. Check README, API docs, setup guide, and architecture docs.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.preflight/01-security-audit.md`
- `.preflight/01-ops-audit.md`
- `.preflight/01-reliability-audit.md`
- `.preflight/01-docs-audit.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Scorecard Synthesis

Read ALL `.preflight/01-*.md` files plus `.preflight/00-codebase-survey.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "preflight-synthesizer"
  description: "Synthesize audit results into production readiness scorecard"
  prompt: |
    Synthesize all audit results into a production readiness scorecard with a GO/NO-GO verdict.

    ## Codebase Survey
    [Insert contents of .preflight/00-codebase-survey.md]

    ## Security Audit
    [Insert contents of .preflight/01-security-audit.md]

    ## Ops Audit
    [Insert contents of .preflight/01-ops-audit.md]

    ## Reliability Audit
    [Insert contents of .preflight/01-reliability-audit.md]

    ## Documentation Audit
    [Insert contents of .preflight/01-docs-audit.md]

    Strict mode: [yes/no based on --strict flag]

    Follow your synthesis strategy. Produce the final scorecard in your defined output format.
```

Save output to `.preflight/report.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Preflight check complete!

## Verdict: [GO / CONDITIONAL GO / NO-GO]
[2-3 sentence summary from scorecard]

## Scorecard
- Security: [PASS/WARN/FAIL] ([X]/[Y] checks passed)
- Ops: [PASS/WARN/FAIL] ([X]/[Y] checks passed)
- Reliability: [PASS/WARN/FAIL] ([X]/[Y] checks passed)
- Documentation: [PASS/WARN/FAIL] ([X]/[Y] checks passed)

## Blocking Issues: [N]
## Warnings: [N]

## Output
- Codebase survey: .preflight/00-codebase-survey.md
- Security audit: .preflight/01-security-audit.md
- Ops audit: .preflight/01-ops-audit.md
- Reliability audit: .preflight/01-reliability-audit.md
- Documentation audit: .preflight/01-docs-audit.md
- Full report: .preflight/report.md

## Next Step
Fix blocking issues and re-run `/preflight` to verify.
```
