---
description: "Scan a codebase (or bootstrap a greenfield project) to produce a multi-phase roadmap with dependency tracking, market research, and exit criteria per phase"
argument-hint: "[--goal 'description'] [--horizon short|full] [--format md|json]"
---

# Codebase Roadmap Generator

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.roadmap/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.roadmap/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress roadmap session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Initialize state

Create `.roadmap/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "goal": null,
    "horizon": "full",
    "format": "md"
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--goal`, `--horizon`, and `--format` flags.

---

## Phase 0: Project Scan (Sequential)

This phase runs FIRST and ALONE. Its output feeds all Phase 1 agents.

Launch the project scanner:

```
Agent:
  subagent_type: "project-scanner"
  description: "Deep structural scan of the codebase"
  prompt: |
    Perform a comprehensive structural analysis of this codebase.

    Follow your analysis strategy to produce a complete project profile.
    Be thorough — downstream analysts depend on your assessment.
```

Save output to `.roadmap/00-project-profile.md`.

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Domain Analysis

Read `.roadmap/00-project-profile.md` to load the project profile.

Launch ALL domain analysts in parallel using multiple Agent tool calls in a single message. Each receives the project profile.

**Important:** If the project profile shows `Has frontend: no`, do NOT launch `frontend-analyst`.

### 1a. Backend Analyst

```
Agent:
  subagent_type: "backend-analyst"
  description: "Analyze backend for roadmap milestones"
  prompt: |
    Analyze this project's backend and propose prioritized milestones for the roadmap.

    ## Project Profile
    [Insert full contents of .roadmap/00-project-profile.md]

    ## Goal (if specified)
    [From state.json flags.goal, or "None — infer natural evolution"]

    Follow your analysis strategy. Produce 5-10 milestones with dependencies.
```

### 1b. Frontend Analyst (if applicable)

```
Agent:
  subagent_type: "frontend-analyst"
  description: "Analyze frontend for roadmap milestones"
  prompt: |
    Analyze this project's frontend and propose prioritized milestones for the roadmap.

    ## Project Profile
    [Insert full contents of .roadmap/00-project-profile.md]

    ## Goal (if specified)
    [From state.json flags.goal, or "None — infer natural evolution"]

    Follow your analysis strategy. Produce 5-10 milestones with dependencies.
```

### 1c. Infra Analyst

```
Agent:
  subagent_type: "infra-analyst"
  description: "Analyze infrastructure for roadmap milestones"
  prompt: |
    Analyze this project's infrastructure and operational maturity. Propose prioritized milestones.

    ## Project Profile
    [Insert full contents of .roadmap/00-project-profile.md]

    ## Goal (if specified)
    [From state.json flags.goal, or "None — infer natural evolution"]

    Follow your analysis strategy. Produce 5-10 milestones with dependencies.
```

### 1d. Security Analyst

```
Agent:
  subagent_type: "security-analyst"
  description: "Analyze security posture for roadmap milestones"
  prompt: |
    Analyze this project's security posture. Propose prioritized milestones.

    ## Project Profile
    [Insert full contents of .roadmap/00-project-profile.md]

    ## Goal (if specified)
    [From state.json flags.goal, or "None — infer natural evolution"]

    Follow your analysis strategy. Produce 5-10 milestones with dependencies.
```

### 1e. Market Research Analyst

```
Agent:
  subagent_type: "market-research-analyst"
  description: "Research competitive landscape for roadmap context"
  prompt: |
    Research the competitive landscape for this project to inform roadmap prioritization.

    ## Project Profile
    [Insert full contents of .roadmap/00-project-profile.md]

    ## Goal (if specified)
    [From state.json flags.goal, or "None — infer natural evolution"]

    Follow your analysis strategy. Produce market context with prioritization recommendations.
```

After ALL agents complete, save each output:

- `.roadmap/01-backend.md`
- `.roadmap/01-frontend.md` (if applicable)
- `.roadmap/01-infra.md`
- `.roadmap/01-security.md`
- `.roadmap/01-market.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Roadmap Synthesis

Read ALL `.roadmap/01-*.md` files plus `.roadmap/00-project-profile.md`.

Launch the roadmap synthesizer:

```
Agent:
  subagent_type: "roadmap-synthesizer"
  description: "Synthesize multi-phase roadmap from domain analyses"
  prompt: |
    Synthesize a unified multi-phase roadmap from these domain analyses.

    ## Project Profile
    [Insert contents of .roadmap/00-project-profile.md]

    ## Backend Analysis
    [Insert contents of .roadmap/01-backend.md]

    ## Frontend Analysis
    [Insert contents of .roadmap/01-frontend.md OR "No frontend — skipped"]

    ## Infrastructure Analysis
    [Insert contents of .roadmap/01-infra.md]

    ## Security Analysis
    [Insert contents of .roadmap/01-security.md]

    ## Market Context
    [Insert contents of .roadmap/01-market.md]

    ## Configuration
    - Goal: [from state.json flags.goal or "Infer natural evolution"]
    - Horizon: [from state.json flags.horizon]

    Follow your synthesis strategy. Build the dependency graph, group milestones into phases,
    define exit criteria, and produce the final roadmap.
```

Save output to `.roadmap/roadmap.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`.

---

## Completion

Present the final summary:

```
Roadmap generation complete!

## Output
- Project profile: .roadmap/00-project-profile.md
- Domain analyses: .roadmap/01-*.md
- Final roadmap: .roadmap/roadmap.md

## Summary
- Phases: [count]
- Total milestones: [count]
- Estimated timeline: [from roadmap]
- Maturity path: [current] -> [target]

Open .roadmap/roadmap.md to review the full roadmap.
```
