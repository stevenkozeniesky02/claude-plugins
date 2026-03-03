---
description: "Estimate infrastructure and API costs at different user scales with cost cliffs, optimization tips, and burn rate projections"
argument-hint: "[--scale 100|1K|10K|100K] [--budget low|medium|high]"
---

# Cost Forecast

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.costforecast/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.costforecast/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress cost forecast session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Check for upstream context

Check if `.stack/tech-decisions.md` exists. If it does, read it -- this provides valuable context about the chosen tech stack and infrastructure.

### 3. Verify codebase exists

Confirm the working directory contains a codebase. If no codebase is detected, the tool can still work with tech stack info or user-provided details.

### 4. Initialize state

Create `.costforecast/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "scale": null,
    "budget": "medium",
    "has_tech_decisions": false
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--scale` and `--budget` flags. Set `has_tech_decisions` based on upstream check.

---

## Phase 0: Infrastructure Discovery (Sequential)

Scan the codebase and any upstream context to understand what infrastructure and services are being used.

**If `.stack/tech-decisions.md` exists:**
Extract infrastructure details from tech stack decisions.

**Otherwise:**
Scan the codebase for infrastructure indicators: Docker configs, deployment configs, database connection strings, API client libraries, cloud SDK imports, CDN configs, queue configs, storage bucket references.

Ask the user to confirm or supplement with AskUserQuestion if key infrastructure details are unclear:
1. **What hosting platform?** (Vercel, AWS, Railway, Fly.io, etc.)
2. **Expected user scale?** (if --scale not provided)
3. **Any fixed costs already committed to?** (domains, premium services, etc.)

**Output file:** `.costforecast/00-infrastructure-inventory.md`

```markdown
# Infrastructure Inventory

## Hosting / Compute
- **Platform:** [detected or specified]
- **Type:** [serverless, container, VM, PaaS]
- **Current tier:** [free, paid -- if detectable]

## Database
- **Type:** [PostgreSQL, MySQL, MongoDB, etc.]
- **Provider:** [Supabase, PlanetScale, Neon, self-hosted, etc.]
- **Current tier:** [free, paid]

## Third-Party APIs
- [API 1] -- [purpose, e.g., "OpenAI for chat completions"]
- [API 2] -- [purpose]
- [API 3] -- [purpose]

## External Services
- **Auth:** [Supabase Auth, Auth0, Clerk, custom]
- **Email:** [Resend, SendGrid, SES, etc.]
- **Storage:** [S3, Cloudflare R2, Supabase Storage, etc.]
- **CDN:** [Cloudflare, CloudFront, Vercel, etc.]
- **Monitoring:** [Sentry, Datadog, etc.]
- **Queue/Background:** [BullMQ, SQS, none]

## Fixed Costs
- [Domain: $X/yr]
- [Other committed costs]

## Scale Targets
- **Current:** [N] users
- **Target:** [from --scale or user input]

## Upstream Context
[Summary from .stack/tech-decisions.md if available, or "No upstream context"]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Cost Analysis

Read `.costforecast/00-infrastructure-inventory.md` to load the infrastructure context.

Launch ALL cost analysis agents in parallel using multiple Agent tool calls in a single message.

### 1a. Infra Cost Analyst

```
Agent:
  subagent_type: "infra-cost-analyst"
  description: "Analyze hosting, database, and infrastructure costs"
  prompt: |
    Analyze the infrastructure costs for this project at different user scales.

    ## Infrastructure Inventory
    [Insert full contents of .costforecast/00-infrastructure-inventory.md]

    Follow your analysis strategy. Research current pricing for all infrastructure components.
    Output in your defined format.
```

### 1b. API Cost Analyst

```
Agent:
  subagent_type: "api-cost-analyst"
  description: "Analyze third-party API and service costs"
  prompt: |
    Analyze the third-party API and external service costs for this project at different user scales.

    ## Infrastructure Inventory
    [Insert full contents of .costforecast/00-infrastructure-inventory.md]

    Follow your analysis strategy. Research current pricing for all APIs and services.
    Output in your defined format.
```

### 1c. Scale Modeler

```
Agent:
  subagent_type: "scale-modeler"
  description: "Model costs at multiple user tiers and identify cost cliffs"
  prompt: |
    Model infrastructure and API costs at 100, 1K, 10K, and 100K user tiers. Identify cost cliffs where pricing jumps dramatically.

    ## Infrastructure Inventory
    [Insert full contents of .costforecast/00-infrastructure-inventory.md]

    Follow your analysis strategy. Produce cost projections at each tier.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.costforecast/01-infra-costs.md`
- `.costforecast/01-api-costs.md`
- `.costforecast/01-scale-model.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Cost Report Synthesis

Read ALL `.costforecast/01-*.md` files plus `.costforecast/00-infrastructure-inventory.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "cost-synthesizer"
  description: "Synthesize cost analysis into forecast report with optimizations"
  prompt: |
    Synthesize all cost analyses into a comprehensive cost forecast report with optimization recommendations.

    ## Infrastructure Inventory
    [Insert contents of .costforecast/00-infrastructure-inventory.md]

    ## Infrastructure Costs
    [Insert contents of .costforecast/01-infra-costs.md]

    ## API Costs
    [Insert contents of .costforecast/01-api-costs.md]

    ## Scale Model
    [Insert contents of .costforecast/01-scale-model.md]

    Budget level: [value of --budget flag]

    Follow your synthesis strategy. Produce the final cost forecast in your defined output format.
```

Save output to `.costforecast/report.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Cost forecast complete!

## Monthly Cost Summary

| Scale | Infra | APIs | Total |
|-------|-------|------|-------|
| 100 users | $[X] | $[X] | $[X]/mo |
| 1K users | $[X] | $[X] | $[X]/mo |
| 10K users | $[X] | $[X] | $[X]/mo |
| 100K users | $[X] | $[X] | $[X]/mo |

## Cost Cliffs
[List any dramatic price jumps]

## Top Optimization
[Single most impactful cost saving]

## Output
- Infrastructure inventory: .costforecast/00-infrastructure-inventory.md
- Infra costs: .costforecast/01-infra-costs.md
- API costs: .costforecast/01-api-costs.md
- Scale model: .costforecast/01-scale-model.md
- Full report: .costforecast/report.md

## Next Step
Review optimization recommendations in the report. Run `/preflight` when ready to launch.
```
