---
description: "Validate and refine a business idea through multi-agent market research, competitive analysis, and feasibility assessment"
argument-hint: "[idea description in quotes]"
---

# Idea Spark

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Do NOT skip ahead, reorder, or merge phases.
2. **Write output files.** Each phase MUST produce its output file in `.spark/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed. Do NOT silently continue.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.spark/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress spark session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Initialize state

Create `.spark/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "idea": null
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

---

## Phase 0: Idea Intake (Conversational)

If the idea was provided as an argument to the command, write it directly into state and proceed.

If no idea was provided, use AskUserQuestion to gather the following — ask one question at a time:

1. **What** — What is the product or service? What problem does it solve?
2. **Who** — Who is the target customer? Be specific (role, company size, industry).
3. **Why now** — Why is this the right time for this product? What has changed?
4. **How** — How will this make money? What is the revenue hypothesis?

**Output file:** `.spark/00-idea-brief.md`

```markdown
# Idea Brief

## Product Concept
[What the product/service is]

## Problem
[The problem it solves]

## Target Customer
[Specific target customer profile]

## Timing Signal
[Why now — what has changed that makes this viable]

## Revenue Hypothesis
[How it will make money]

## Constraints
[Solo dev, bootstrap, time constraints, etc.]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`, set `flags.idea` to the product concept.

---

## Phase 1: Parallel Research

Read `.spark/00-idea-brief.md` to load the idea brief.

Launch ALL research agents in parallel using multiple Agent tool calls in a single message. Each agent receives the idea brief.

### 1a. Market Researcher

```
Agent:
  subagent_type: "market-researcher"
  description: "Research market size, growth trends, and timing for this idea"
  prompt: |
    Research the market opportunity for this business idea.

    ## Idea Brief
    [Insert full contents of .spark/00-idea-brief.md]

    Follow your analysis strategy. Use WebSearch to find real market data.
    Output in your defined format.
```

### 1b. Competitor Scout

```
Agent:
  subagent_type: "competitor-scout"
  description: "Find and analyze competing products and alternatives"
  prompt: |
    Find and analyze all relevant competitors for this business idea.

    ## Idea Brief
    [Insert full contents of .spark/00-idea-brief.md]

    Follow your analysis strategy. Use WebSearch and WebFetch to find real competitors.
    Output in your defined format.
```

### 1c. Revenue Modeler

```
Agent:
  subagent_type: "revenue-modeler"
  description: "Propose revenue models and unit economics"
  prompt: |
    Propose viable revenue models with unit economics for this business idea.

    ## Idea Brief
    [Insert full contents of .spark/00-idea-brief.md]

    Follow your analysis strategy. Propose 2-3 distinct models.
    Output in your defined format.
```

### 1d. Feasibility Analyst

```
Agent:
  subagent_type: "feasibility-analyst"
  description: "Assess technical feasibility for a solo developer"
  prompt: |
    Assess the technical feasibility of this business idea for a solo developer.

    ## Idea Brief
    [Insert full contents of .spark/00-idea-brief.md]

    Follow your analysis strategy. Be honest about timelines and complexity.
    Output in your defined format.
```

After ALL agents complete, save each agent's raw output to individual files:

- `.spark/01-market-research.md`
- `.spark/01-competitor-analysis.md`
- `.spark/01-revenue-models.md`
- `.spark/01-feasibility.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Business Case Synthesis

Read ALL `.spark/01-*.md` files.

Launch the synthesizer agent:

```
Agent:
  subagent_type: "spark-synthesizer"
  description: "Synthesize all research into a business case with verdict"
  prompt: |
    You have received research outputs from four specialist analysts. Synthesize them into a final business case with a GO/CAUTION/NO-GO verdict.

    ## Idea Brief
    [Insert contents of .spark/00-idea-brief.md]

    ## Market Research
    [Insert contents of .spark/01-market-research.md]

    ## Competitor Analysis
    [Insert contents of .spark/01-competitor-analysis.md]

    ## Revenue Models
    [Insert contents of .spark/01-revenue-models.md]

    ## Feasibility Assessment
    [Insert contents of .spark/01-feasibility.md]

    Follow your synthesis strategy. Produce the final business case in your defined output format.
```

Save the synthesizer's output to `.spark/business-case.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Idea validation complete!

## Verdict: [GO / CAUTION / NO-GO]
[2-3 sentence summary from business case]

## Output
- Idea brief: .spark/00-idea-brief.md
- Market research: .spark/01-market-research.md
- Competitor analysis: .spark/01-competitor-analysis.md
- Revenue models: .spark/01-revenue-models.md
- Feasibility assessment: .spark/01-feasibility.md
- Business case: .spark/business-case.md

## Next Step
If this idea looks promising, run `/stack` to choose the right technology stack for building it.
```
