---
description: "Research and recommend the optimal tech stack for a project through multi-agent evaluation of frameworks, databases, hosting, and developer experience"
argument-hint: "[--requirements 'description'] [--budget low|medium|high]"
---

# Stack Pick

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Do NOT skip ahead, reorder, or merge phases.
2. **Write output files.** Each phase MUST produce its output file in `.stack/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed. Do NOT silently continue.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.stack/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress stack pick session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Check upstream context

Check if `.spark/business-case.md` exists:

- If it exists: Read it and extract product concept, technical requirements, feasibility notes, and recommended business model. Set `has_business_case` flag to `true`.
- If it does not exist: Set `has_business_case` flag to `false`. This is fine — stack-pick works standalone.

### 3. Initialize state

Create `.stack/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "requirements": null,
    "budget": "medium",
    "has_business_case": false
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

---

## Phase 0: Requirements Gathering (Conversational)

If an upstream `.spark/business-case.md` was found, extract the following from it:

- Product type and key features
- Expected scale and complexity
- Technical feasibility notes
- Any technology preferences or constraints mentioned

If `--requirements` was provided as an argument, use that directly.

If neither an upstream business case nor `--requirements` are available, use AskUserQuestion to gather the following — ask one question at a time:

1. **What are you building?** — What is the product? Web app, mobile app, API, CLI tool, etc.
2. **Expected scale** — How many users/requests do you anticipate? (start small — what does month 1 look like vs. year 1?)
3. **Budget** — What is your infrastructure budget? (free tier only, <$50/mo, <$200/mo, no limit)
4. **Performance requirements** — Any specific latency, throughput, or real-time requirements?
5. **Preferences** — Any languages/frameworks you already know or want to use? Any you want to avoid?

**Output file:** `.stack/00-requirements.md`

```markdown
# Stack Requirements

## Product Type
[Web app / Mobile app / API / CLI / Desktop / Hybrid]

## Scale Requirements
- **Month 1:** [Expected users/load]
- **Year 1:** [Growth target]
- **Year 3:** [Aspirational scale]

## Budget
**Level:** [Free / Low (<$50/mo) / Medium (<$200/mo) / High (no limit)]

## Performance Requirements
[Latency, throughput, real-time, offline, etc.]

## Key Features Driving Stack Choice
1. [Feature 1 and why it matters for stack]
2. [Feature 2 and why it matters for stack]
3. [Feature 3 and why it matters for stack]

## Constraints
- **Must use:** [Languages/frameworks the founder knows or requires]
- **Avoid:** [Technologies to avoid and why]
- **Timeline:** [MVP deadline if any]

## Upstream Context
[Summary from business case if available, or "No upstream context — standalone evaluation"]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`, set `flags.requirements` to the product type, set `flags.budget` to the budget level, set `flags.has_business_case` appropriately.

---

## Phase 1: Parallel Research

Read `.stack/00-requirements.md` to load the requirements.

Launch ALL research agents in parallel using multiple Agent tool calls in a single message. Each agent receives the requirements.

### 1a. Backend Researcher

```
Agent:
  subagent_type: "backend-researcher"
  description: "Research and evaluate backend framework options for this project"
  prompt: |
    Research and evaluate backend framework options for this project.

    ## Requirements
    [Insert full contents of .stack/00-requirements.md]

    Follow your analysis strategy. Use WebSearch to find real benchmarks and ecosystem data.
    Output in your defined format.
```

### 1b. Frontend Researcher

```
Agent:
  subagent_type: "frontend-researcher"
  description: "Research and evaluate frontend framework options for this project"
  prompt: |
    Research and evaluate frontend framework options for this project.

    ## Requirements
    [Insert full contents of .stack/00-requirements.md]

    Follow your analysis strategy. Use WebSearch to find real data on frameworks and tooling.
    Output in your defined format.
```

### 1c. Infrastructure Researcher

```
Agent:
  subagent_type: "infra-researcher"
  description: "Research hosting, database, and infrastructure options for this project"
  prompt: |
    Research hosting, database, and infrastructure options for this project.

    ## Requirements
    [Insert full contents of .stack/00-requirements.md]

    Follow your analysis strategy. Use WebSearch to find current pricing and free tier details.
    Output in your defined format.
```

### 1d. DX Analyst

```
Agent:
  subagent_type: "dx-analyst"
  description: "Assess developer experience across all technology options"
  prompt: |
    Assess the developer experience across all technology options being considered for this project.

    ## Requirements
    [Insert full contents of .stack/00-requirements.md]

    Follow your analysis strategy. Use WebSearch to find real developer feedback.
    Output in your defined format.
```

After ALL agents complete, save each agent's raw output to individual files:

- `.stack/01-backend-research.md`
- `.stack/01-frontend-research.md`
- `.stack/01-infra-research.md`
- `.stack/01-dx-assessment.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Stack Synthesis

Read ALL `.stack/01-*.md` files.

Launch the synthesizer agent:

```
Agent:
  subagent_type: "stack-synthesizer"
  description: "Synthesize all research into 2-3 complete stack recommendations"
  prompt: |
    You have received research outputs from four specialist analysts. Synthesize them into 2-3 complete, coherent tech stack recommendations ranked by fit.

    ## Requirements
    [Insert contents of .stack/00-requirements.md]

    ## Backend Research
    [Insert contents of .stack/01-backend-research.md]

    ## Frontend Research
    [Insert contents of .stack/01-frontend-research.md]

    ## Infrastructure Research
    [Insert contents of .stack/01-infra-research.md]

    ## DX Assessment
    [Insert contents of .stack/01-dx-assessment.md]

    Follow your synthesis strategy. Produce the final tech stack decisions in your defined output format.
```

Save the synthesizer's output to `.stack/tech-decisions.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Stack evaluation complete!

## Recommended Stack: [Stack Name]
[2-3 sentence summary of the recommended stack and why it fits]

## Output
- Requirements: .stack/00-requirements.md
- Backend research: .stack/01-backend-research.md
- Frontend research: .stack/01-frontend-research.md
- Infrastructure research: .stack/01-infra-research.md
- DX assessment: .stack/01-dx-assessment.md
- Tech decisions: .stack/tech-decisions.md

## Next Step
Run `/scaffold` to scaffold the project using the recommended stack.
```
