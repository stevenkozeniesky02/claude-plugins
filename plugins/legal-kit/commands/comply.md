---
description: "Generate compliance documentation by scanning codebase data handling patterns"
argument-hint: "[--scope privacy|terms|cookies|all] [--regulation gdpr|ccpa|both]"
---

# /comply - Compliance Documentation Generator

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.legalkit/` before the next phase begins. Read from prior phase files — do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan — execute it.

## IMPORTANT DISCLAIMER

**Output from this tool is a starting point for compliance documentation — NOT legal advice.** Always have a qualified attorney review all generated documents before publishing or relying on them. Regulations vary by jurisdiction, industry, and specific business circumstances.

## Pre-flight Checks

### 1. Check for existing session

Check if `.legalkit/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress compliance session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify codebase

Check that the current directory contains a codebase worth scanning:

- Look for `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `*.csproj`, or similar
- If no project files found, warn the user and ask if they want to proceed with manual input

### 3. Check upstream context

Check for outputs from prior plugins that provide useful context:

- `.spark/business-case.md` — Business context and target audience
- `.pricing/pricing-strategy.md` — Pricing model (affects terms of service)
- `.costforecast/report.md` — Infrastructure details (affects data processing disclosures)

If found, note them for agents to reference.

### 4. Initialize state

Create `.legalkit/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "scope": "all",
    "regulation": "both"
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--scope` and `--regulation` flags.

---

## Phase 0: Data Handling Survey (Sequential)

This phase runs FIRST and ALONE. Its output feeds all Phase 1 agents.

### Ask the user key questions

Before scanning, ask the user (use AskUserQuestion):

1. **Business jurisdiction** — "Where is your company legally incorporated?" (Options: US, EU, UK, Other)
2. **Target audience regions** — "Where are your users located?" (Options: US only, EU/EEA, Global, Other)
3. **Data sensitivity** — "What type of personal data does your app handle?" (Options: Basic (email/name), Sensitive (financial/health), Minimal (no PII), Not sure)

### Launch data flow scanner

```
Agent:
  subagent_type: "general-purpose"
  description: "Survey codebase data handling patterns"
  prompt: |
    You are a data-flow-mapper agent. Scan this codebase to identify all personal data handling patterns.

    [Insert data-flow-mapper.md agent instructions here]

    ## User Context
    - Jurisdiction: [from user answers]
    - Target regions: [from user answers]
    - Data sensitivity: [from user answers]

    ## Upstream Context
    [Insert any upstream plugin outputs found]
```

Save output to `.legalkit/00-data-flow-map.md`.

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Compliance Analysis

Read `.legalkit/00-data-flow-map.md` to load the data flow map.

Launch specialists in parallel based on `--scope` flag. If scope is `all`, launch all three.

### 1a. Regulation Matcher (always runs)

```
Agent:
  subagent_type: "general-purpose"
  description: "Match data flows to regulatory requirements"
  prompt: |
    You are a regulation-matcher agent. Analyze data flows against regulatory requirements.

    [Insert regulation-matcher.md agent instructions here]

    ## Data Flow Map
    [Insert full contents of .legalkit/00-data-flow-map.md]

    ## Configuration
    - Regulation focus: [from state.json flags.regulation]
    - Jurisdiction: [from user answers]
    - Target regions: [from user answers]
```

### 1b. Doc Drafter (scope: privacy, terms, cookies, or all)

```
Agent:
  subagent_type: "general-purpose"
  description: "Draft compliance documents"
  prompt: |
    You are a doc-drafter agent. Draft compliance documentation.

    [Insert doc-drafter.md agent instructions here]

    ## Data Flow Map
    [Insert full contents of .legalkit/00-data-flow-map.md]

    ## Configuration
    - Scope: [from state.json flags.scope]
    - Regulation: [from state.json flags.regulation]
    - Jurisdiction: [from user answers]
```

After ALL agents complete, save each output:

- `.legalkit/01-regulation-analysis.md`
- `.legalkit/01-draft-documents.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Compliance Synthesis

Read ALL `.legalkit/01-*.md` files plus `.legalkit/00-data-flow-map.md`.

Launch the compliance synthesizer:

```
Agent:
  subagent_type: "general-purpose"
  description: "Synthesize final compliance documentation package"
  prompt: |
    You are a compliance-synthesizer agent. Produce the final compliance documentation package.

    [Insert compliance-synthesizer.md agent instructions here]

    ## Data Flow Map
    [Insert contents of .legalkit/00-data-flow-map.md]

    ## Regulation Analysis
    [Insert contents of .legalkit/01-regulation-analysis.md]

    ## Draft Documents
    [Insert contents of .legalkit/01-draft-documents.md]

    ## Configuration
    - Scope: [from state.json flags.scope]
    - Regulation: [from state.json flags.regulation]
```

Save output to `.legalkit/compliance-package.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`.

---

## Completion

Present the final summary:

```
Compliance documentation generation complete!

⚠️  DISCLAIMER: This output is a STARTING POINT — not legal advice.
    Have a qualified attorney review all documents before publishing.

## Output
- Data flow map: .legalkit/00-data-flow-map.md
- Regulation analysis: .legalkit/01-regulation-analysis.md
- Draft documents: .legalkit/01-draft-documents.md
- Final package: .legalkit/compliance-package.md

## Documents Generated
[List documents based on --scope flag]

## Compliance Gaps Found
[Summary of gaps requiring attention]

## Next Steps
1. Have an attorney review all generated documents
2. Fill in [PLACEHOLDER] sections with your specific details
3. Address the compliance gaps listed above
4. Set up recurring review schedule (quarterly recommended)
```
