---
description: "Bootstrap a project skeleton from tech stack decisions with directory structure, CI/CD, Docker, configs, and documentation"
argument-hint: "[--stack 'description'] [--minimal]"
---

# Scaffold Kit

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Execute phases in order.** Do NOT skip ahead, reorder, or merge phases.
2. **Write output files.** Each phase MUST produce its output file in `.scaffold/` before the next phase begins. Read from prior phase files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails, STOP immediately. Present the error and ask the user how to proceed. Do NOT silently continue.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin or `general-purpose`. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.
6. **NEVER write project files without user approval.** Phase 2 presents the scaffold plan. The user MUST approve before any files are created in the working directory.

## Pre-flight Checks

### 1. Check for existing session

Check if `.scaffold/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display the current phase, and ask the user:

  ```
  Found an in-progress scaffold session:
  Current phase: [phase from state]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Check upstream context

Check for upstream input files in priority order:

- **Primary:** `.stack/tech-decisions.md` — If it exists, read it and extract the full recommended stack (language, backend, frontend, database, hosting, CI/CD). Set `has_tech_decisions` flag to `true`.
- **Secondary:** `.roadmap/roadmap.md` — If it exists, read it and extract project modules, milestones, and architecture notes. Set `has_roadmap` flag to `true`.
- **Tertiary:** `.spark/business-case.md` — If it exists, read it and extract product concept and constraints. Set `has_business_case` flag to `true`.

At minimum, the tech stack must be known. If `.stack/tech-decisions.md` does not exist and no `--stack` argument was provided, ask the user to either run `/stack` first or provide a stack description with `--stack`.

### 3. Initialize state

Create `.scaffold/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "stack": null,
    "minimal": false,
    "has_tech_decisions": false,
    "has_roadmap": false,
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

## Phase 0: Stack Analysis

Read all available upstream files and extract stack information.

If `--stack` was provided as an argument, use that as the stack description. Otherwise, extract from `.stack/tech-decisions.md`.

If `--minimal` was passed, set the scaffold mode to Minimal (fewer files, no CI/CD, no Docker, basic configs only).

**Output file:** `.scaffold/00-scaffold-spec.md`

```markdown
# Scaffold Spec

## Tech Stack

- **Language:** [Primary language]
- **Backend:** [Framework and version]
- **Frontend:** [Framework and version, or N/A]
- **Database:** [Database and provider]
- **Hosting:** [Provider and tier]
- **CI/CD:** [Provider]

## Project Modules

[List of modules/features from roadmap, or "Single module — monolith" if no roadmap]

## Scaffold Mode

[Full / Minimal]

- Full: Directory structure, CI/CD, Docker, configs, linting, docs
- Minimal: Directory structure, package config, .gitignore, README only

## Upstream Context

- Tech decisions: [Available / Not available]
- Roadmap: [Available / Not available]
- Business case: [Available / Not available]
```

Update `state.json`: set `current_phase` to 1, add phase 0 to `completed_phases`, set `flags.stack` to the primary language/framework.

---

## Phase 1: Parallel Generation

Read `.scaffold/00-scaffold-spec.md` to load the scaffold spec.

Launch ALL generator agents in parallel using multiple Agent tool calls in a single message. Each agent receives the scaffold spec.

### 1a. Project Structurer

```
Agent:
  subagent_type: "project-structurer"
  description: "Design directory structure following stack conventions"
  prompt: |
    Design the project directory structure for this project.

    ## Scaffold Spec
    [Insert full contents of .scaffold/00-scaffold-spec.md]

    Follow your analysis strategy. Use Glob to check for existing conventions if any files exist.
    Output in your defined format.
```

### 1b. CI/CD Generator

```
Agent:
  subagent_type: "cicd-generator"
  description: "Generate CI/CD pipeline, Dockerfile, and docker-compose for this project"
  prompt: |
    Generate CI/CD pipeline configuration, Dockerfile, and docker-compose for this project.

    ## Scaffold Spec
    [Insert full contents of .scaffold/00-scaffold-spec.md]

    Follow your analysis strategy. Generate complete, ready-to-use files.
    Output in your defined format.
```

### 1c. Config Generator

```
Agent:
  subagent_type: "config-generator"
  description: "Generate linting, formatting, pre-commit, and editor configs for this project"
  prompt: |
    Generate all developer tooling configuration files for this project.

    ## Scaffold Spec
    [Insert full contents of .scaffold/00-scaffold-spec.md]

    Follow your analysis strategy. Generate complete, ready-to-use config files.
    Output in your defined format.
```

### 1d. Docs Bootstrapper

```
Agent:
  subagent_type: "docs-bootstrapper"
  description: "Generate README, CONTRIBUTING, .env.example, and architecture docs for this project"
  prompt: |
    Generate all documentation files for this project.

    ## Scaffold Spec
    [Insert full contents of .scaffold/00-scaffold-spec.md]

    Follow your analysis strategy. Write practical, not ceremonial, documentation.
    Output in your defined format.
```

After ALL agents complete, save each agent's raw output to individual files:

- `.scaffold/01-structure.md`
- `.scaffold/01-cicd.md`
- `.scaffold/01-configs.md`
- `.scaffold/01-docs.md`

Update `state.json`: set `current_phase` to 2, add phase 1 to `completed_phases`.

---

## Phase 2: Assembly and User Approval

Read ALL `.scaffold/01-*.md` files.

Launch the scaffold assembler agent:

```
Agent:
  subagent_type: "scaffold-assembler"
  description: "Assemble unified scaffold plan from all generators and resolve conflicts"
  prompt: |
    You have received outputs from four specialist generators. Assemble them into a unified, consistent scaffold plan ready for user approval.

    ## Scaffold Spec
    [Insert contents of .scaffold/00-scaffold-spec.md]

    ## Directory Structure
    [Insert contents of .scaffold/01-structure.md]

    ## CI/CD Configuration
    [Insert contents of .scaffold/01-cicd.md]

    ## Developer Configs
    [Insert contents of .scaffold/01-configs.md]

    ## Documentation
    [Insert contents of .scaffold/01-docs.md]

    Follow your synthesis strategy. Produce the final scaffold plan in your defined output format.
```

Save the assembler's output to `.scaffold/scaffold-plan.md`.

### User Approval Gate

Present the scaffold plan to the user, listing all files that will be created:

```
Scaffold plan ready!

## Files to Create
[Numbered list of every file from the scaffold plan]

Total: [X] files across [Y] directories

Please review the plan in .scaffold/scaffold-plan.md

1. Approve — create all files in the working directory
2. Modify — tell me what to change, and I will regenerate
3. Cancel — discard the scaffold plan
```

**If the user approves:** Write all files from the scaffold plan to the working directory. Create directories first, then files in the order specified by the plan.

**If the user requests modifications:** Update the scaffold plan based on feedback and re-present for approval.

**If the user cancels:** Do not write any files. Keep the `.scaffold/` directory with the plan for reference.

Update `state.json`: set `status` to `"complete"`, `current_phase` to `"done"`, add phase 2 to `completed_phases`.

---

## Completion

Present the final summary:

```
Project scaffold complete!

## Files Created
[List of all files written to the working directory]

## Setup Commands
[Stack-specific setup commands, e.g.:]
- `npm install` or `pip install -r requirements.txt`
- `cp .env.example .env`
- `docker-compose up -d`

## Output
- Scaffold spec: .scaffold/00-scaffold-spec.md
- Directory structure: .scaffold/01-structure.md
- CI/CD configs: .scaffold/01-cicd.md
- Developer configs: .scaffold/01-configs.md
- Documentation: .scaffold/01-docs.md
- Scaffold plan: .scaffold/scaffold-plan.md

## Next Step
Run `/build` to start implementing features from your roadmap.
```
