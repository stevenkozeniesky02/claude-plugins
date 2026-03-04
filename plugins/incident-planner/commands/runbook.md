---
description: "Generate runbooks and incident response playbooks from architecture analysis"
argument-hint: "[--scope path/to/dir] [--severity critical|high|all]"
---

# Incident Planner

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.runbooks/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.runbooks/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress runbook session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify codebase exists

Confirm the working directory contains a codebase (look for package.json, pyproject.toml, go.mod, Cargo.toml, Makefile, docker-compose.yml, or src/ directory). If no codebase is detected, inform the user and halt.

### 3. Initialize state

Create `.runbooks/` directory and `state.json`:

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

## Phase 0: Architecture Survey (Sequential)

Scan the codebase to map all services, dependencies, communication paths, and infrastructure components.

If `--scope` was specified, limit the scan to that directory.

**Output file:** `.runbooks/00-architecture-survey.md`

```markdown
# Architecture Survey

## Overview
- **Application type:** [monolith / microservices / serverless / hybrid]
- **Language(s):** [detected languages]
- **Framework(s):** [detected frameworks]
- **Database(s):** [detected from configs/code]
- **External services:** [APIs, SaaS integrations]

## Components
- **Services:** [list of services/modules]
- **Data stores:** [databases, caches, queues]
- **External dependencies:** [third-party APIs, services]
- **Infrastructure:** [Docker, K8s, cloud services detected]

## Communication Paths
- **HTTP endpoints:** [API routes]
- **Message queues:** [if any]
- **Background jobs:** [if any]
- **Cron/scheduled tasks:** [if any]

## Scope
[Full architecture or limited to --scope path]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.runbooks/00-architecture-survey.md` to load the architecture context.

Launch ALL analysis agents in parallel using multiple Agent tool calls in a single message.

### 1a. Architecture Mapper

```
Agent:
  subagent_type: "architecture-mapper"
  description: "Map all services, dependencies, and communication paths"
  prompt: |
    Map the complete architecture of this application including all services, dependencies, and communication paths.

    ## Architecture Survey
    [Insert full contents of .runbooks/00-architecture-survey.md]

    Follow your analysis strategy. Produce a detailed dependency graph and data flow map.
    Output in your defined format.
```

### 1b. Failure Mode Analyst

```
Agent:
  subagent_type: "failure-mode-analyst"
  description: "Identify failure modes for each component"
  prompt: |
    Identify all potential failure modes for each component in this architecture.

    ## Architecture Survey
    [Insert full contents of .runbooks/00-architecture-survey.md]

    Follow your analysis strategy. Analyze crash, timeout, data corruption, and capacity failure scenarios.
    Output in your defined format.
```

### 1c. Runbook Writer

```
Agent:
  subagent_type: "runbook-writer"
  description: "Generate step-by-step runbooks for failure scenarios"
  prompt: |
    Generate step-by-step operational runbooks for the most critical failure scenarios.

    ## Architecture Survey
    [Insert full contents of .runbooks/00-architecture-survey.md]

    Follow your analysis strategy. Write clear, actionable runbooks with diagnosis, mitigation, and recovery steps.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.runbooks/01-architecture-map.md`
- `.runbooks/01-failure-modes.md`
- `.runbooks/01-runbooks.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Incident Response Plan Synthesis

Read ALL `.runbooks/01-*.md` files plus `.runbooks/00-architecture-survey.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "runbook-synthesizer"
  description: "Create master incident response plan and severity classification"
  prompt: |
    Synthesize all findings into a master incident response plan.

    ## Architecture Survey
    [Insert contents of .runbooks/00-architecture-survey.md]

    ## Architecture Map
    [Insert contents of .runbooks/01-architecture-map.md]

    ## Failure Modes
    [Insert contents of .runbooks/01-failure-modes.md]

    ## Runbooks
    [Insert contents of .runbooks/01-runbooks.md]

    Severity filter: [value of --severity flag]

    Follow your synthesis strategy. Produce the master incident response plan in your defined output format.
```

Save output to `.runbooks/incident-response-plan.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Incident planning complete!

## Summary
- **Components mapped:** [N]
- **Failure modes identified:** [N]
- **Runbooks generated:** [N]
- **Severity levels:** [N] Critical, [N] High, [N] Medium

## Top 3 Failure Scenarios
1. [Scenario 1] -- Severity: [level] -- Runbook ready
2. [Scenario 2] -- Severity: [level] -- Runbook ready
3. [Scenario 3] -- Severity: [level] -- Runbook ready

## Output
- Architecture survey: .runbooks/00-architecture-survey.md
- Architecture map: .runbooks/01-architecture-map.md
- Failure modes: .runbooks/01-failure-modes.md
- Runbooks: .runbooks/01-runbooks.md
- Master plan: .runbooks/incident-response-plan.md

## Next Step
Review the runbooks with your team. Practice the critical scenarios. Set up monitoring alerts for the failure modes identified.
```
