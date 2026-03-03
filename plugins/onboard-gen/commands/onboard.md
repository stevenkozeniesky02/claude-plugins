---
description: "Generate contributor documentation, architecture walkthroughs, setup guides, and convention references by scanning your codebase"
argument-hint: "[--scope path/to/dir] [--format guide|wiki|readme]"
---

# Onboard Gen

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Phase 0 MUST complete before Phase 1 starts. Phase 1 MUST complete before Phase 2.
2. **Write output files.** Each phase MUST produce its output file in `.onboard/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.onboard/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress onboard session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Verify codebase exists

Confirm the working directory contains a codebase (look for package.json, pyproject.toml, go.mod, Cargo.toml, Makefile, or src/ directory). If no codebase is detected, inform the user and halt.

### 3. Initialize state

Create `.onboard/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "scope": null,
    "format": "guide"
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--scope` (limit to subdirectory) and `--format` (output style: guide, wiki, or readme).

---

## Phase 0: Codebase Survey (Sequential)

Perform a deep scan of the codebase to understand its structure, tech stack, existing documentation, and key entry points.

If `--scope` was specified, limit the scan to that directory but still note the full project context.

**Output file:** `.onboard/00-codebase-survey.md`

```markdown
# Codebase Survey

## Project Identity
- **Name:** [from package.json/pyproject.toml/etc.]
- **Language(s):** [detected languages]
- **Framework(s):** [detected frameworks]
- **Package manager:** [npm/pip/go mod/cargo/etc.]

## Project Structure
- **Root directories:** [top-level directories with purposes]
- **Source entry point(s):** [main files]
- **Test structure:** [test directory layout]
- **Config files:** [list of configuration files]

## Existing Documentation
- **README:** [exists? quality?]
- **API docs:** [exists? format?]
- **Architecture docs:** [exists?]
- **Contributing guide:** [exists?]
- **Inline docs:** [JSDoc/docstrings present?]

## Key Dependencies
- [Dependency 1] -- [purpose]
- [Dependency 2] -- [purpose]
- [Dependency 3] -- [purpose]

## Environment Setup Indicators
- **Environment variables:** [.env.example exists? How many vars?]
- **Database:** [detected type and setup requirements]
- **External services:** [APIs, auth providers, etc.]
- **Docker:** [Dockerfile/docker-compose present?]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Documentation Generation

Read `.onboard/00-codebase-survey.md` to load the codebase context.

Launch ALL documentation agents in parallel using multiple Agent tool calls in a single message.

### 1a. Architecture Documenter

```
Agent:
  subagent_type: "architecture-documenter"
  description: "Map and document system architecture"
  prompt: |
    Map and document the system architecture of this codebase.

    ## Codebase Survey
    [Insert full contents of .onboard/00-codebase-survey.md]

    Follow your analysis strategy. Produce a comprehensive architecture overview.
    Output in your defined format.
```

### 1b. Flow Tracer

```
Agent:
  subagent_type: "flow-tracer"
  description: "Trace key user flows through the codebase"
  prompt: |
    Trace the key user flows through this codebase.

    ## Codebase Survey
    [Insert full contents of .onboard/00-codebase-survey.md]

    Follow your analysis strategy. Trace 3-5 core flows from entry point to completion.
    Output in your defined format.
```

### 1c. Setup Guide Writer

```
Agent:
  subagent_type: "setup-guide-writer"
  description: "Document setup and development environment instructions"
  prompt: |
    Document the complete setup and development environment instructions for this codebase.

    ## Codebase Survey
    [Insert full contents of .onboard/00-codebase-survey.md]

    Follow your analysis strategy. Produce step-by-step instructions a new developer can follow.
    Output in your defined format.
```

### 1d. Convention Extractor

```
Agent:
  subagent_type: "convention-extractor"
  description: "Extract coding conventions and patterns from the codebase"
  prompt: |
    Extract the coding conventions, patterns, and style decisions from this codebase.

    ## Codebase Survey
    [Insert full contents of .onboard/00-codebase-survey.md]

    Follow your analysis strategy. Document the patterns a new contributor needs to follow.
    Output in your defined format.
```

After ALL agents complete, save each output:

- `.onboard/01-architecture.md`
- `.onboard/01-flows.md`
- `.onboard/01-setup-guide.md`
- `.onboard/01-conventions.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Documentation Synthesis

Read ALL `.onboard/01-*.md` files plus `.onboard/00-codebase-survey.md`.

Launch the synthesizer:

```
Agent:
  subagent_type: "onboard-synthesizer"
  description: "Synthesize all documentation into a polished onboarding suite"
  prompt: |
    Synthesize all documentation outputs into a polished onboarding documentation suite.

    ## Codebase Survey
    [Insert contents of .onboard/00-codebase-survey.md]

    ## Architecture Documentation
    [Insert contents of .onboard/01-architecture.md]

    ## Flow Traces
    [Insert contents of .onboard/01-flows.md]

    ## Setup Guide
    [Insert contents of .onboard/01-setup-guide.md]

    ## Coding Conventions
    [Insert contents of .onboard/01-conventions.md]

    Output format: [value of --format flag: guide|wiki|readme]

    Follow your synthesis strategy. Produce the complete onboarding documentation in your defined output format.
```

Save output to `.onboard/onboarding-guide.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Onboarding documentation complete!

## Generated Documentation
- Architecture overview with component diagram
- [N] key flow walkthroughs
- Step-by-step setup guide
- Coding conventions reference

## Output
- Codebase survey: .onboard/00-codebase-survey.md
- Architecture: .onboard/01-architecture.md
- Flow traces: .onboard/01-flows.md
- Setup guide: .onboard/01-setup-guide.md
- Conventions: .onboard/01-conventions.md
- Complete guide: .onboard/onboarding-guide.md

## Next Step
Review the generated documentation and customize it for your team. Consider adding it to your project's docs/ directory.
```
