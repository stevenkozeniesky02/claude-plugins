---
description: "Identify scaling bottlenecks before they hit production with capacity modeling at 10x/100x/1000x"
argument-hint: "[--scope path/to/dir] [--baseline N-users]"
---

# Scale Simulator

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.scalesim/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.scalesim/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress scale simulation:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify codebase exists

Confirm the working directory contains a codebase (look for package.json, pyproject.toml, go.mod, Cargo.toml, Makefile, docker-compose.yml, or src/ directory). If no codebase is detected, inform the user and halt.

### 3. Initialize state

Create `.scalesim/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "scope": null,
    "baseline": 100
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--scope` (limit analysis to a subdirectory) and `--baseline` (current user count, defaults to 100).

---

## Phase 0: Infrastructure Survey (Sequential)

Scan the codebase to understand its architecture, data stores, external integrations, and current scaling posture.

If `--scope` was specified, limit the scan to that directory.

**Output file:** `.scalesim/00-infra-survey.md`

```markdown
# Infrastructure Survey

## Overview
- **Application type:** [monolith / microservices / serverless / hybrid]
- **Language(s):** [detected languages]
- **Framework(s):** [detected frameworks]
- **Baseline users:** [from --baseline or default 100]

## Data Stores
- **Primary database:** [type, connection config]
- **Cache:** [Redis/Memcached/none]
- **Search:** [Elasticsearch/Algolia/none]
- **File storage:** [S3/local/none]
- **Message queue:** [RabbitMQ/SQS/Redis/none]

## External Services
- **APIs called:** [list with rate limit info if found]
- **Auth provider:** [self-hosted/Auth0/Firebase/etc.]
- **CDN:** [CloudFront/Cloudflare/none]

## Current Scaling Posture
- **Horizontal scaling:** [stateless/stateful/unknown]
- **Connection pooling:** [configured/missing]
- **Caching strategy:** [aggressive/minimal/none]
- **Background processing:** [job queue/synchronous]

## Scope
[Full application or limited to --scope path]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.scalesim/00-infra-survey.md` to load the infrastructure context.

Launch ALL analysis agents in parallel using multiple Agent tool calls in a single message.

### 1a. Bottleneck Hunter

```
Agent:
  subagent_type: "bottleneck-hunter"
  description: "Find single points of failure and synchronous bottlenecks"
  prompt: |
    Identify scaling bottlenecks in this application architecture.

    ## Infrastructure Survey
    [Insert full contents of .scalesim/00-infra-survey.md]

    Follow your analysis strategy. Find single points of failure, synchronous bottlenecks, and resource contention.
    Output in your defined format.
```

### 1b. Database Profiler

```
Agent:
  subagent_type: "database-profiler"
  description: "Analyze database schemas for scaling issues"
  prompt: |
    Analyze the database layer for scaling bottlenecks.

    ## Infrastructure Survey
    [Insert full contents of .scalesim/00-infra-survey.md]

    Follow your analysis strategy. Check for N+1 queries, missing indexes, unbounded queries, and connection limits.
    Output in your defined format.
```

### 1c. Queue Analyst

```
Agent:
  subagent_type: "queue-analyst"
  description: "Analyze message queue and background job configurations"
  prompt: |
    Analyze message queue configurations and background processing for scaling limits.

    ## Infrastructure Survey
    [Insert full contents of .scalesim/00-infra-survey.md]

    Follow your analysis strategy. Check throughput limits, backpressure handling, and dead letter configurations.
    Output in your defined format.
```

### 1d. Capacity Modeler

```
Agent:
  subagent_type: "capacity-modeler"
  description: "Model what breaks at 10x, 100x, and 1000x scale"
  prompt: |
    Model the application's behavior at 10x, 100x, and 1000x the baseline user count.

    ## Infrastructure Survey
    [Insert full contents of .scalesim/00-infra-survey.md]

    Follow your analysis strategy. Identify what breaks first at each scale tier with specific numbers.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.scalesim/01-bottlenecks.md`
- `.scalesim/01-database-profile.md`
- `.scalesim/01-queue-analysis.md`
- `.scalesim/01-capacity-model.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Scaling Report Synthesis

Read ALL `.scalesim/01-*.md` files plus `.scalesim/00-infra-survey.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "scale-synthesizer"
  description: "Synthesize findings into a scaling readiness report"
  prompt: |
    Synthesize all scaling analysis into a prioritized scaling readiness report.

    ## Infrastructure Survey
    [Insert contents of .scalesim/00-infra-survey.md]

    ## Bottlenecks
    [Insert contents of .scalesim/01-bottlenecks.md]

    ## Database Profile
    [Insert contents of .scalesim/01-database-profile.md]

    ## Queue Analysis
    [Insert contents of .scalesim/01-queue-analysis.md]

    ## Capacity Model
    [Insert contents of .scalesim/01-capacity-model.md]

    Follow your synthesis strategy. Produce the final scaling report in your defined output format.
```

Save output to `.scalesim/report.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Scale simulation complete!

## Scaling Ceiling: [N] users (estimated)
[2-3 sentence summary of what breaks first]

## Bottleneck Map
- Bottlenecks found: [N]
- First to break: [component] at [N] users
- Database issues: [N]
- Queue issues: [N]

## Scale Tiers
- 10x ([N] users): [OK / Issues]
- 100x ([N] users): [OK / Issues / Breaks]
- 1000x ([N] users): [Breaks at ...]

## Top 3 Priority Fixes
1. [Fix 1] -- unlocks scaling to [N] users
2. [Fix 2] -- removes [bottleneck]
3. [Fix 3] -- improves [capacity]

## Output
- Infrastructure survey: .scalesim/00-infra-survey.md
- Bottlenecks: .scalesim/01-bottlenecks.md
- Database profile: .scalesim/01-database-profile.md
- Queue analysis: .scalesim/01-queue-analysis.md
- Capacity model: .scalesim/01-capacity-model.md
- Full report: .scalesim/report.md

## Next Step
Fix the first bottleneck to raise your scaling ceiling. Re-run `/simulate` after changes to verify improvement.
```
