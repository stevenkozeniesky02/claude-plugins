---
description: "Recommend pricing strategy based on costs, competitors, and value with tier design and margin analysis"
argument-hint: "[--model subscription|usage|transaction|freemium] [--target b2b|b2c|prosumer]"
---

# Price Model

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.pricing/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.pricing/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress pricing session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Check for upstream context

Look for upstream plugin outputs:

- `.costforecast/report.md` -- Cost projections from cost-forecast
- `.spark/business-case.md` -- Business case from idea-spark
- `.stack/tech-decisions.md` -- Tech stack decisions

If found, note them for use in Phase 1.

### 3. Initialize state

Create `.pricing/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "model": null,
    "target": null
  },
  "upstream": {
    "cost_forecast": false,
    "business_case": false,
    "tech_decisions": false
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--model` (subscription, usage, transaction, freemium) and `--target` (b2b, b2c, prosumer).

---

## Phase 0: Pricing Context (Sequential)

Gather context about the product, target market, and costs. If upstream files exist, read them. Otherwise, ask the user:

1. What does your product do?
2. Who is the target customer? (B2B, B2C, prosumer)
3. What's the primary value you deliver?
4. What pricing model do you prefer? (subscription, usage-based, freemium, transaction)
5. What do competitors charge? (if known)
6. What are your costs per user? (if known)

Scan the codebase for pricing signals: feature flags, plan tiers, usage limits in code.

**Output file:** `.pricing/00-pricing-context.md`

```markdown
# Pricing Context

## Product
- **Product:** [description]
- **Target market:** [B2B/B2C/prosumer]
- **Primary value metric:** [what users pay for]
- **Preferred model:** [subscription/usage/transaction/freemium]

## Cost Basis
- **Cost per user/month:** $[N] (from cost-forecast or estimate)
- **Infrastructure costs:** [summary]
- **API/third-party costs:** [summary]

## Current Pricing (if any)
[Detected from codebase or user input]

## Competitor Pricing (if known)
[From user input or business case]

## Feature Inventory
[Features detected in codebase that could be gated by tier]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.pricing/00-pricing-context.md` and any available upstream files.

Launch ALL agents in parallel using multiple Agent tool calls in a single message.

### 1a. Competitor Pricing Analyst

```
Agent:
  subagent_type: "competitor-pricing-analyst"
  description: "Research competitor pricing and positioning"
  prompt: |
    Research competitor pricing, packaging, and positioning for this product.

    ## Pricing Context
    [Insert full contents of .pricing/00-pricing-context.md]

    ## Business Case (if available)
    [Insert .spark/business-case.md contents or "Not available"]

    Follow your analysis strategy. Produce a competitive pricing landscape.
    Output in your defined format.
```

### 1b. Unit Economics Modeler

```
Agent:
  subagent_type: "unit-economics-modeler"
  description: "Calculate COGS, contribution margin, and break-even"
  prompt: |
    Calculate unit economics for this product.

    ## Pricing Context
    [Insert full contents of .pricing/00-pricing-context.md]

    ## Cost Forecast (if available)
    [Insert .costforecast/report.md contents or "Not available"]

    Follow your analysis strategy. Produce COGS, margins, and break-even analysis.
    Output in your defined format.
```

### 1c. Value Metric Analyst

```
Agent:
  subagent_type: "value-metric-analyst"
  description: "Identify the right value metric and tier structure"
  prompt: |
    Identify the optimal value metric and tier structure for this product.

    ## Pricing Context
    [Insert full contents of .pricing/00-pricing-context.md]

    Follow your analysis strategy. Determine what customers should pay for and how tiers should be structured.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.pricing/01-competitor-pricing.md`
- `.pricing/01-unit-economics.md`
- `.pricing/01-value-metrics.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Pricing Strategy Synthesis

Read ALL `.pricing/01-*.md` files plus `.pricing/00-pricing-context.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "pricing-synthesizer"
  description: "Produce pricing strategy with tier recommendations"
  prompt: |
    Synthesize all analysis into a pricing strategy recommendation.

    ## Pricing Context
    [Insert contents of .pricing/00-pricing-context.md]

    ## Competitor Pricing
    [Insert contents of .pricing/01-competitor-pricing.md]

    ## Unit Economics
    [Insert contents of .pricing/01-unit-economics.md]

    ## Value Metrics
    [Insert contents of .pricing/01-value-metrics.md]

    Follow your synthesis strategy. Produce the final pricing strategy in your defined output format.
```

Save output to `.pricing/strategy.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Pricing strategy complete!

## Recommended Model: [model type]

## Tier Summary
| Tier | Price | Target | Key Feature Gate |
|------|-------|--------|-----------------|
| [Free/Starter] | $[N]/mo | [who] | [limit] |
| [Pro] | $[N]/mo | [who] | [feature] |
| [Enterprise] | $[N]/mo | [who] | [feature] |

## Key Metrics
- Gross margin: [N]%
- Break-even: [N] paying customers
- Target ARPU: $[N]/mo

## Output
- Pricing context: .pricing/00-pricing-context.md
- Competitor analysis: .pricing/01-competitor-pricing.md
- Unit economics: .pricing/01-unit-economics.md
- Value metrics: .pricing/01-value-metrics.md
- Full strategy: .pricing/strategy.md

## Next Step
Validate pricing with 5-10 potential customers before launching. Use `/landing` to create a landing page with the pricing.
```
