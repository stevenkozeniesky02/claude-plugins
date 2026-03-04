---
description: "Generate investor-ready pitch materials including deck outline, one-pager, elevator pitch, and investor FAQ"
argument-hint: "[--format deck|onepager|all] [--stage pre-seed|seed|series-a]"
---

# Pitch Craft

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.pitch/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.pitch/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress pitch session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Check for upstream context

Look for upstream plugin outputs that enrich the pitch:

- `.spark/business-case.md` -- Business case from idea-spark
- `.stack/tech-decisions.md` -- Tech stack decisions
- `.costforecast/report.md` -- Cost projections

If found, note them for use in Phase 1. If not found, the plugin works standalone but will ask the user more questions.

### 3. Initialize state

Create `.pitch/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "format": "all",
    "stage": "pre-seed"
  },
  "upstream": {
    "business_case": false,
    "tech_decisions": false,
    "cost_forecast": false
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--format` (deck, onepager, or all) and `--stage` (pre-seed, seed, or series-a).

---

## Phase 0: Context Gathering (Sequential)

If upstream files exist, read them. Otherwise, ask the user key questions:

1. What does your product do? (one sentence)
2. Who is the target customer?
3. What problem does it solve?
4. How do you make money? (or plan to)
5. What traction do you have? (users, revenue, waitlist)
6. How much are you raising and at what valuation?

Also scan the codebase for traction signals: user counts in configs, analytics integrations, payment integrations, feature completeness.

**Output file:** `.pitch/00-pitch-context.md`

```markdown
# Pitch Context

## Product
- **One-liner:** [product description]
- **Target customer:** [who]
- **Problem:** [what problem]
- **Solution:** [how it solves it]

## Business
- **Revenue model:** [how it makes money]
- **Traction:** [current metrics]
- **Stage:** [pre-seed/seed/series-a]
- **Ask:** [amount and terms if known]

## Upstream Data
- **Business case:** [available/not available]
- **Tech stack:** [available/not available]
- **Cost forecast:** [available/not available]

## Codebase Signals
- **Tech stack:** [detected]
- **Feature completeness:** [estimated]
- **Integrations:** [payment, analytics, auth detected]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.pitch/00-pitch-context.md` and any available upstream files.

Launch ALL agents in parallel using multiple Agent tool calls in a single message.

### 1a. Narrative Builder

```
Agent:
  subagent_type: "narrative-builder"
  description: "Craft the pitch story arc"
  prompt: |
    Craft a compelling pitch narrative for this product.

    ## Pitch Context
    [Insert full contents of .pitch/00-pitch-context.md]

    ## Business Case (if available)
    [Insert .spark/business-case.md contents or "Not available"]

    Follow your analysis strategy. Build the story arc: problem → solution → traction → ask.
    Output in your defined format.
```

### 1b. Market Sizer

```
Agent:
  subagent_type: "market-sizer"
  description: "Calculate TAM/SAM/SOM with sources"
  prompt: |
    Calculate TAM/SAM/SOM for this product with bottom-up and top-down estimates.

    ## Pitch Context
    [Insert full contents of .pitch/00-pitch-context.md]

    ## Business Case (if available)
    [Insert .spark/business-case.md contents or "Not available"]

    Follow your analysis strategy. Produce market size estimates with sources.
    Output in your defined format.
```

### 1c. Financial Modeler

```
Agent:
  subagent_type: "financial-modeler"
  description: "Build revenue projections and unit economics"
  prompt: |
    Build financial projections for this product.

    ## Pitch Context
    [Insert full contents of .pitch/00-pitch-context.md]

    ## Cost Forecast (if available)
    [Insert .costforecast/report.md contents or "Not available"]

    Follow your analysis strategy. Produce revenue projections, unit economics, and burn rate estimates.
    Output in your defined format.
```

### 1d. Deck Writer

```
Agent:
  subagent_type: "deck-writer"
  description: "Write slide-by-slide pitch deck content"
  prompt: |
    Write slide-by-slide content for a pitch deck.

    ## Pitch Context
    [Insert full contents of .pitch/00-pitch-context.md]

    Stage: [value of --stage flag]

    Follow your analysis strategy. Write content for each slide following the appropriate deck framework.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.pitch/01-narrative.md`
- `.pitch/01-market-size.md`
- `.pitch/01-financials.md`
- `.pitch/01-deck-content.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Pitch Materials Synthesis

Read ALL `.pitch/01-*.md` files plus `.pitch/00-pitch-context.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "pitch-synthesizer"
  description: "Produce all pitch formats from specialist outputs"
  prompt: |
    Synthesize all specialist outputs into polished pitch materials.

    ## Pitch Context
    [Insert contents of .pitch/00-pitch-context.md]

    ## Narrative
    [Insert contents of .pitch/01-narrative.md]

    ## Market Size
    [Insert contents of .pitch/01-market-size.md]

    ## Financials
    [Insert contents of .pitch/01-financials.md]

    ## Deck Content
    [Insert contents of .pitch/01-deck-content.md]

    Format: [value of --format flag]
    Stage: [value of --stage flag]

    Follow your synthesis strategy. Produce all requested pitch formats.
```

Save output to `.pitch/pitch-materials.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Pitch materials complete!

## Generated Materials
- Pitch deck outline: [N] slides
- One-pager: included
- Elevator pitch: [30-second and 60-second versions]
- Investor FAQ: [N] questions answered

## Key Numbers
- TAM: $[N]
- SAM: $[N]
- SOM: $[N]
- Revenue projection (Year 1): $[N]
- Ask: $[N] at [terms]

## Output
- Pitch context: .pitch/00-pitch-context.md
- Narrative: .pitch/01-narrative.md
- Market size: .pitch/01-market-size.md
- Financials: .pitch/01-financials.md
- Deck content: .pitch/01-deck-content.md
- All materials: .pitch/pitch-materials.md

## Next Step
Review the materials, customize with your specific metrics, and practice the elevator pitch. Use `/pricing` to develop your pricing strategy.
```
