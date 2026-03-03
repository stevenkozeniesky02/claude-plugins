---
description: "Scan a codebase to discover actionable improvement ideas across Code, Security, Performance, Docs, UX, and dynamically detected categories with competitive market research"
argument-hint: "[--categories 'DevOps,Testing'] [--depth deep|quick]"
---

# Codebase Ideation Discovery

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Do NOT skip ahead, reorder, or merge phases.
2. **Write output files.** Each phase MUST produce its output file in `.ideation/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed. Do NOT silently continue.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.

## Pre-flight Checks

### 1. Check for existing session

Check if `.ideation/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress ideation session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Initialize state

Create `.ideation/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "extra_categories": [],
    "depth": "deep"
  },
  "current_phase": 0,
  "completed_phases": [],
  "files_created": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

Parse arguments for `--categories` and `--depth` flags. Update the flags object.

---

## Phase 0: Project Scan

Before launching specialists, do a quick structural scan to produce context that all agents will receive.

Use Glob, Grep, and Read to determine:

1. **Stack** — Languages, frameworks, package managers (check package.json, pyproject.toml, go.mod, Cargo.toml, etc.)
2. **Domain** — What the project does (read README, main entry points, project description)
3. **Structure** — Top-level directory layout, key modules, entry points
4. **Size** — Approximate file count, line count, number of modules
5. **Has frontend?** — Check for React/Vue/Angular/Svelte/HTML files
6. **Test coverage** — Check for test directories, test file count

**Output file:** `.ideation/00-project-context.md`

```markdown
# Project Context

- **Name:** [from package metadata or directory name]
- **Domain:** [what the project does]
- **Stack:** [languages, frameworks, key dependencies]
- **Structure:** [top-level layout summary]
- **Size:** [file count, approximate LOC]
- **Has frontend:** yes/no
- **Test files:** [count]
- **Extra categories requested:** [from --categories flag]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`.

---

## Phase 1: Parallel Specialist Analysis

Read `.ideation/00-project-context.md` to load project context.

Launch ALL specialist agents in parallel using multiple Agent tool calls in a single message. Each agent receives the project context.

**Important:** If `Has frontend: no` in the project context, do NOT launch the `ux-analyst`. Launch only the applicable agents.

### 1a. Code Analyst

```
Agent:
  subagent_type: "code-analyst"
  description: "Scan codebase for code improvement ideas"
  prompt: |
    Scan this codebase and produce 5+ actionable improvement ideas for the Code category.

    ## Project Context
    [Insert full contents of .ideation/00-project-context.md]

    Follow your analysis strategy. Every idea must reference specific files in this project.
    Output your ideas in the actionable card format defined in your instructions.
```

### 1b. Security Analyst

```
Agent:
  subagent_type: "security-analyst"
  description: "Scan codebase for security improvement ideas"
  prompt: |
    Scan this codebase and produce 5+ actionable improvement ideas for the Security category.

    ## Project Context
    [Insert full contents of .ideation/00-project-context.md]

    Follow your analysis strategy. Every idea must reference specific files in this project.
    Output your ideas in the actionable card format defined in your instructions.
```

### 1c. Performance Analyst

```
Agent:
  subagent_type: "performance-analyst"
  description: "Scan codebase for performance improvement ideas"
  prompt: |
    Scan this codebase and produce 5+ actionable improvement ideas for the Performance category.

    ## Project Context
    [Insert full contents of .ideation/00-project-context.md]

    Follow your analysis strategy. Every idea must reference specific files in this project.
    Output your ideas in the actionable card format defined in your instructions.
```

### 1d. Docs Analyst

```
Agent:
  subagent_type: "docs-analyst"
  description: "Scan codebase for documentation improvement ideas"
  prompt: |
    Scan this codebase and produce 5+ actionable improvement ideas for the Docs category.

    ## Project Context
    [Insert full contents of .ideation/00-project-context.md]

    Follow your analysis strategy. Every idea must reference specific files in this project.
    Output your ideas in the actionable card format defined in your instructions.
```

### 1e. UX Analyst (only if frontend detected)

```
Agent:
  subagent_type: "ux-analyst"
  description: "Scan frontend for UX improvement ideas"
  prompt: |
    Scan this codebase's frontend layer and produce 5+ actionable improvement ideas for the UI/UX category.

    ## Project Context
    [Insert full contents of .ideation/00-project-context.md]

    If there is no frontend, output "No frontend detected. Skipping UX analysis." and stop.
    Otherwise, follow your analysis strategy. Every idea must reference specific components.
    Output your ideas in the actionable card format defined in your instructions.
```

### 1f. Market Research Analyst

```
Agent:
  subagent_type: "market-research-analyst"
  description: "Research competitive landscape for this project"
  prompt: |
    Analyze this project's domain and research the competitive landscape.

    ## Project Context
    [Insert full contents of .ideation/00-project-context.md]

    Follow your analysis strategy:
    1. Detect the domain from the project context
    2. Use WebSearch to find competitors and industry trends
    3. Produce a market context brief with 3-5 market-informed ideas

    Output in your defined format.
```

After ALL agents complete, save each agent's raw output to individual files:

- `.ideation/01-code-raw.md`
- `.ideation/01-security-raw.md`
- `.ideation/01-performance-raw.md`
- `.ideation/01-docs-raw.md`
- `.ideation/01-ux-raw.md` (if applicable)
- `.ideation/01-market-raw.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Curation and Synthesis

Read ALL `.ideation/01-*-raw.md` files.

Launch the curator agent:

```
Agent:
  subagent_type: "idea-curator"
  description: "Curate and synthesize ideation results"
  prompt: |
    You have received raw idea outputs from multiple specialist analysts. Synthesize them into a final ideation report.

    ## Project Context
    [Insert contents of .ideation/00-project-context.md]

    ## Code Analyst Output
    [Insert contents of .ideation/01-code-raw.md]

    ## Security Analyst Output
    [Insert contents of .ideation/01-security-raw.md]

    ## Performance Analyst Output
    [Insert contents of .ideation/01-performance-raw.md]

    ## Docs Analyst Output
    [Insert contents of .ideation/01-docs-raw.md]

    ## UX Analyst Output
    [Insert contents of .ideation/01-ux-raw.md OR "Skipped — no frontend"]

    ## Market Research Output
    [Insert contents of .ideation/01-market-raw.md]

    ## Extra Categories Requested
    [From state.json flags.extra_categories]

    Follow your synthesis strategy. Produce the final ideation report in your defined output format.
```

Save the curator's output to `.ideation/ideas.md`.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Ideation discovery complete!

## Output
- Project context: .ideation/00-project-context.md
- Raw specialist outputs: .ideation/01-*-raw.md
- Final ideation report: .ideation/ideas.md

## Summary
- Categories analyzed: [list]
- Total ideas: [count]
- P1 ideas: [count] | P2: [count] | P3: [count]

Open .ideation/ideas.md to review all ideas.
```
