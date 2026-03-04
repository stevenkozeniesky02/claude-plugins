---
description: "Generate landing page copy and marketing content from your codebase"
argument-hint: "[--format copy|html|both] [--tone professional|casual|bold]"
---

# Ship Page

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.shippage/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.shippage/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress landing page session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Check for upstream context

Look for upstream plugin outputs:

- `.spark/business-case.md` -- Business case from idea-spark
- `.pricing/strategy.md` -- Pricing strategy from price-model
- `.pitch/pitch-materials.md` -- Pitch materials from pitch-craft

If found, note them for use in Phase 1.

### 3. Initialize state

Create `.shippage/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "format": "both",
    "tone": "professional"
  },
  "upstream": {
    "business_case": false,
    "pricing": false,
    "pitch": false
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--format` (copy, html, or both) and `--tone` (professional, casual, or bold).

---

## Phase 0: Product Survey (Sequential)

Scan the codebase to identify all user-facing features, read upstream context, and gather product positioning information.

**Output file:** `.shippage/00-product-survey.md`

```markdown
# Product Survey

## Product Identity
- **Name:** [from package.json, README, or user input]
- **Tagline:** [from README or upstream pitch materials]
- **Description:** [what the product does]
- **Target audience:** [who it's for]

## Features Detected
[List of user-facing features found in the codebase]

## Tech Stack
[Relevant tech stack details for "built with" section]

## Pricing
[From .pricing/strategy.md or "Not available"]

## Upstream Data
- **Business case:** [available/not available]
- **Pricing strategy:** [available/not available]
- **Pitch materials:** [available/not available]

## Tone
[Selected tone: professional/casual/bold]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Analysis

Read `.shippage/00-product-survey.md` and any available upstream files.

Launch ALL agents in parallel using multiple Agent tool calls in a single message.

### 1a. Feature Extractor

```
Agent:
  subagent_type: "feature-extractor"
  description: "Extract all user-facing features from the codebase"
  prompt: |
    Scan the codebase to identify and catalog all user-facing features.

    ## Product Survey
    [Insert full contents of .shippage/00-product-survey.md]

    Follow your analysis strategy. Produce a comprehensive feature inventory.
    Output in your defined format.
```

### 1b. Benefit Mapper

```
Agent:
  subagent_type: "benefit-mapper"
  description: "Map features to user benefits and value propositions"
  prompt: |
    Transform features into compelling user benefits and value propositions.

    ## Product Survey
    [Insert full contents of .shippage/00-product-survey.md]

    ## Business Case (if available)
    [Insert .spark/business-case.md contents or "Not available"]

    Follow your analysis strategy. Map every feature to a user benefit.
    Output in your defined format.
```

### 1c. Copywriter

```
Agent:
  subagent_type: "copywriter"
  description: "Write landing page copy sections"
  prompt: |
    Write conversion-optimized landing page copy for this product.

    ## Product Survey
    [Insert full contents of .shippage/00-product-survey.md]

    ## Pitch Materials (if available)
    [Insert .pitch/pitch-materials.md contents or "Not available"]

    Tone: [value of --tone flag]

    Follow your analysis strategy. Write all landing page sections.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.shippage/01-features.md`
- `.shippage/01-benefits.md`
- `.shippage/01-copy.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Landing Page Assembly

Read ALL `.shippage/01-*.md` files plus `.shippage/00-product-survey.md`.

Launch the assembler:

```
Agent:
  subagent_type: "page-assembler"
  description: "Assemble final landing page copy and optional HTML"
  prompt: |
    Assemble the final landing page from all copy and content.

    ## Product Survey
    [Insert contents of .shippage/00-product-survey.md]

    ## Features
    [Insert contents of .shippage/01-features.md]

    ## Benefits
    [Insert contents of .shippage/01-benefits.md]

    ## Copy
    [Insert contents of .shippage/01-copy.md]

    ## Pricing (if available)
    [Insert .pricing/strategy.md contents or "Not available"]

    Format: [value of --format flag]
    Tone: [value of --tone flag]

    Follow your assembly strategy. Produce the final landing page output.
```

Save output to `.shippage/landing-page.md` and optionally `.shippage/index.html`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Landing page complete!

## Page Sections
- Hero: [headline preview]
- Features: [N] features highlighted
- Social proof: [included/placeholder]
- Pricing: [included/not included]
- CTA: [primary call to action]

## Output
- Product survey: .shippage/00-product-survey.md
- Features: .shippage/01-features.md
- Benefits: .shippage/01-benefits.md
- Copy: .shippage/01-copy.md
- Landing page: .shippage/landing-page.md
[- HTML: .shippage/index.html (if --format html or both)]

## Next Step
Review the copy, add your own testimonials and screenshots, and deploy. Use `/comply` to generate legal pages (privacy policy, terms of service).
```
