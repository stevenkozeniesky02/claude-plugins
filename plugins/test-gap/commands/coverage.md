---
description: "Find untested critical paths and generate test strategies with risk-ranked coverage maps"
argument-hint: "[--scope path/to/dir] [--generate-skeletons]"
---

# Test Gap Analysis

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.testgap/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.testgap/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress test gap session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify codebase exists

Confirm the working directory contains a codebase (look for package.json, pyproject.toml, go.mod, Cargo.toml, Makefile, or src/ directory). If no codebase is detected, inform the user and halt.

### 3. Initialize state

Create `.testgap/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "scope": null,
    "generate_skeletons": false
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--scope` (limit analysis to a subdirectory) and `--generate-skeletons` (output skeleton test files).

---

## Phase 0: Codebase Survey (Sequential)

Scan the codebase to understand its structure, testing framework, and existing test coverage.

If `--scope` was specified, limit the scan to that directory.

**Output file:** `.testgap/00-codebase-survey.md`

```markdown
# Codebase Survey

## Overview
- **Language(s):** [detected languages]
- **Framework(s):** [detected frameworks]
- **Test framework:** [Jest/Vitest/pytest/Go test/etc.]
- **Test runner command:** [npm test / pytest / etc.]
- **Total source files:** [count]
- **Total test files:** [count]
- **Test-to-source ratio:** [percentage]

## Structure
- **Source directories:** [src/, app/, lib/, etc.]
- **Test directories:** [tests/, __tests__/, *.test.*, etc.]
- **Test file pattern:** [*.test.ts / test_*.py / *_test.go]
- **Config files:** [jest.config, pytest.ini, vitest.config, etc.]

## Scope
[Full codebase or limited to --scope path]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.testgap/00-codebase-survey.md` to load the codebase context.

Launch ALL analysis agents in parallel using multiple Agent tool calls in a single message.

### 1a. Coverage Mapper

```
Agent:
  subagent_type: "coverage-mapper"
  description: "Map test coverage against source modules"
  prompt: |
    Map existing test coverage against all source modules in this codebase.

    ## Codebase Survey
    [Insert full contents of .testgap/00-codebase-survey.md]

    Follow your analysis strategy. Identify which modules have tests, which don't, and the quality of existing coverage.
    Output in your defined format.
```

### 1b. Risk Path Finder

```
Agent:
  subagent_type: "risk-path-finder"
  description: "Identify high-risk untested code paths"
  prompt: |
    Identify high-risk untested code paths in this codebase.

    ## Codebase Survey
    [Insert full contents of .testgap/00-codebase-survey.md]

    Follow your analysis strategy. Focus on auth, payments, data mutations, error handlers, and other critical paths.
    Output in your defined format.
```

### 1c. Test Strategist

```
Agent:
  subagent_type: "test-strategist"
  description: "Recommend test strategies per untested path"
  prompt: |
    Recommend test strategies for untested code paths in this codebase.

    ## Codebase Survey
    [Insert full contents of .testgap/00-codebase-survey.md]

    Follow your analysis strategy. Recommend test types (unit, integration, e2e, property-based) per path.
    Output in your defined format.
```

### 1d. Skeleton Writer

```
Agent:
  subagent_type: "skeleton-writer"
  description: "Generate skeleton test files with test names"
  prompt: |
    Generate skeleton test file outlines for untested modules in this codebase.

    ## Codebase Survey
    [Insert full contents of .testgap/00-codebase-survey.md]

    Follow your analysis strategy. Generate test file paths, describe blocks, and test case names.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.testgap/01-coverage-map.md`
- `.testgap/01-risk-paths.md`
- `.testgap/01-test-strategies.md`
- `.testgap/01-test-skeletons.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Coverage Report Synthesis

Read ALL `.testgap/01-*.md` files plus `.testgap/00-codebase-survey.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "coverage-synthesizer"
  description: "Synthesize findings into a prioritized coverage improvement plan"
  prompt: |
    Synthesize all test gap findings into a prioritized coverage improvement plan.

    ## Codebase Survey
    [Insert contents of .testgap/00-codebase-survey.md]

    ## Coverage Map
    [Insert contents of .testgap/01-coverage-map.md]

    ## Risk Paths
    [Insert contents of .testgap/01-risk-paths.md]

    ## Test Strategies
    [Insert contents of .testgap/01-test-strategies.md]

    ## Test Skeletons
    [Insert contents of .testgap/01-test-skeletons.md]

    Generate skeletons: [value of --generate-skeletons flag]

    Follow your synthesis strategy. Produce the final coverage improvement report in your defined output format.
```

Save output to `.testgap/report.md`.

If `--generate-skeletons` was set, also write the skeleton test files to the codebase.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Test gap analysis complete!

## Coverage Overview
- **Source modules:** [N]
- **Modules with tests:** [N] ([%])
- **Modules without tests:** [N] ([%])
- **High-risk untested paths:** [N]

## Top 5 Priority Gaps
1. [Module/path] -- [risk level] -- [recommended test type]
2. [Module/path] -- [risk level] -- [recommended test type]
3. [Module/path] -- [risk level] -- [recommended test type]
4. [Module/path] -- [risk level] -- [recommended test type]
5. [Module/path] -- [risk level] -- [recommended test type]

## Output
- Codebase survey: .testgap/00-codebase-survey.md
- Coverage map: .testgap/01-coverage-map.md
- Risk paths: .testgap/01-risk-paths.md
- Test strategies: .testgap/01-test-strategies.md
- Test skeletons: .testgap/01-test-skeletons.md
- Full report: .testgap/report.md

## Next Step
Start with the highest-risk untested paths. Use `/build` to implement tests following the recommended strategies.
```
