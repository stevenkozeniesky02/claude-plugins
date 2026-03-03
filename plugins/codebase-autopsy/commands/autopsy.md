---
description: "Perform deep tech debt analysis with risk-scored time bombs, complexity mapping, and prioritized refactoring recommendations"
argument-hint: "[--scope path/to/dir] [--severity critical|high|medium|all]"
---

# Codebase Autopsy

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.autopsy/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.autopsy/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress autopsy session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify codebase exists

Confirm the working directory contains a codebase (look for package.json, pyproject.toml, go.mod, Cargo.toml, Makefile, or src/ directory). If no codebase is detected, inform the user and halt.

### 3. Initialize state

Create `.autopsy/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "scope": null,
    "severity": "all"
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--scope` (limit analysis to a subdirectory) and `--severity` (filter output by severity level).

---

## Phase 0: Codebase Survey (Sequential)

Scan the codebase to understand its structure, size, and tech stack.

If `--scope` was specified, limit the scan to that directory.

**Output file:** `.autopsy/00-codebase-survey.md`

```markdown
# Codebase Survey

## Overview
- **Language(s):** [detected languages]
- **Framework(s):** [detected frameworks]
- **Total files:** [count]
- **Total lines of code:** [estimated]
- **Age of codebase:** [from git history if available]

## Structure
- **Source directories:** [src/, app/, lib/, etc.]
- **Test coverage:** [test files found? approximate ratio?]
- **Config files:** [list]
- **Third-party integrations:** [detected from imports/configs]

## Scope
[Full codebase or limited to --scope path]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.autopsy/00-codebase-survey.md` to load the codebase context.

Launch ALL analysis agents in parallel using multiple Agent tool calls in a single message.

### 1a. Debt Detector

```
Agent:
  subagent_type: "debt-detector"
  description: "Find tech debt indicators in the codebase"
  prompt: |
    Find all tech debt indicators in this codebase.

    ## Codebase Survey
    [Insert full contents of .autopsy/00-codebase-survey.md]

    Follow your analysis strategy. Search for TODO/FIXME/HACK comments, hardcoded values, copy-paste code, workarounds, and shortcuts.
    Output in your defined format.
```

### 1b. Complexity Analyst

```
Agent:
  subagent_type: "complexity-analyst"
  description: "Measure code complexity across the codebase"
  prompt: |
    Analyze code complexity across this codebase.

    ## Codebase Survey
    [Insert full contents of .autopsy/00-codebase-survey.md]

    Follow your analysis strategy. Measure function length, nesting depth, coupling, and cohesion.
    Output in your defined format.
```

### 1c. Pattern Archaeologist

```
Agent:
  subagent_type: "pattern-archaeologist"
  description: "Find outdated patterns and legacy conventions"
  prompt: |
    Identify outdated patterns, deprecated API usage, and legacy conventions in this codebase.

    ## Codebase Survey
    [Insert full contents of .autopsy/00-codebase-survey.md]

    Follow your analysis strategy. Look for patterns that were once best practice but are now anti-patterns.
    Output in your defined format.
```

### 1d. Risk Scorer

```
Agent:
  subagent_type: "risk-scorer"
  description: "Score risks and identify time bombs in the codebase"
  prompt: |
    Identify and score risks in this codebase -- find the "time bombs" that will cause problems if not addressed.

    ## Codebase Survey
    [Insert full contents of .autopsy/00-codebase-survey.md]

    Follow your analysis strategy. Assign detonation risk scores based on blast radius and probability.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.autopsy/01-debt-findings.md`
- `.autopsy/01-complexity-analysis.md`
- `.autopsy/01-pattern-archaeology.md`
- `.autopsy/01-risk-scores.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Autopsy Report Synthesis

Read ALL `.autopsy/01-*.md` files plus `.autopsy/00-codebase-survey.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "autopsy-synthesizer"
  description: "Synthesize all findings into a prioritized autopsy report"
  prompt: |
    Synthesize all tech debt findings into a prioritized autopsy report.

    ## Codebase Survey
    [Insert contents of .autopsy/00-codebase-survey.md]

    ## Tech Debt Findings
    [Insert contents of .autopsy/01-debt-findings.md]

    ## Complexity Analysis
    [Insert contents of .autopsy/01-complexity-analysis.md]

    ## Pattern Archaeology
    [Insert contents of .autopsy/01-pattern-archaeology.md]

    ## Risk Scores
    [Insert contents of .autopsy/01-risk-scores.md]

    Severity filter: [value of --severity flag]

    Follow your synthesis strategy. Produce the final autopsy report in your defined output format.
```

Save output to `.autopsy/report.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Codebase autopsy complete!

## Health Verdict: [HEALTHY / DEGRADING / CRITICAL]
[2-3 sentence summary]

## Key Findings
- Time bombs: [N] (Critical: [N], High: [N], Medium: [N])
- Complexity hotspots: [N] files above threshold
- Outdated patterns: [N] found
- Tech debt items: [N] total

## Top 3 Priority Fixes
1. [Fix 1] -- [effort estimate]
2. [Fix 2] -- [effort estimate]
3. [Fix 3] -- [effort estimate]

## Output
- Codebase survey: .autopsy/00-codebase-survey.md
- Debt findings: .autopsy/01-debt-findings.md
- Complexity analysis: .autopsy/01-complexity-analysis.md
- Pattern archaeology: .autopsy/01-pattern-archaeology.md
- Risk scores: .autopsy/01-risk-scores.md
- Full report: .autopsy/report.md

## Next Step
Address the highest-priority time bombs first. Re-run `/autopsy` after fixes to track improvement.
```
