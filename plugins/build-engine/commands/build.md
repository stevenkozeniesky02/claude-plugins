---
description: "Implement roadmap milestones feature-by-feature using TDD and multi-agent coding with user approval gates at each milestone"
argument-hint: "[--milestone N] [--resume]"
---

# Build Engine

## CRITICAL BEHAVIORAL RULES

You MUST follow these rules exactly. Violating any of them is a failure.

1. **Never start a milestone without user approval.** Present the plan, wait for go-ahead. Do NOT begin implementation until the user explicitly approves.
2. **Write output files.** Track all progress in `.build/` files. Read from prior files -- do NOT rely on context window memory.
3. **Halt on failure.** If any agent fails or tests don't pass, STOP immediately. Present the error and ask the user how to proceed. Do NOT silently continue.
4. **Use only local agents.** All `subagent_type` references use agents bundled with this plugin. No cross-plugin dependencies.
5. **Never enter plan mode autonomously.** Do NOT use EnterPlanMode. This command IS the plan -- execute it.
6. **Tests before implementation.** Always write tests first, verify they fail, then implement. Never write implementation code before tests exist.
7. **Commit after each milestone.** Never leave uncommitted work between milestones.

## Pre-flight Checks

### 1. Check for existing session

Check if `.build/state.json` exists:

- If it exists and `status` is `"in_progress"`: Read it, display current milestone progress, and ask the user:

  ```
  Found an in-progress build session:
  Current milestone: [milestone from state]
  Milestones completed: [X/Y]

  1. Resume from where we left off
  2. Start fresh (archives existing session)
  ```

- If it exists and `status` is `"complete"`: Ask whether to archive and start fresh.

### 2. Check upstream context

Check for the roadmap file:

- **Primary:** `.roadmap/roadmap.md` — If it exists, read it and extract all milestones, features, and dependencies. Set `has_roadmap` flag to `true`.

If `.roadmap/roadmap.md` does not exist:

```
No roadmap found at .roadmap/roadmap.md

1. Run /generate to create a roadmap from your codebase
2. Describe your milestones manually and I'll create a roadmap
```

### 3. Initialize state

Create `.build/` directory and `state.json`:

```json
{
  "status": "in_progress",
  "flags": {
    "milestone": null,
    "resume": false,
    "has_roadmap": false
  },
  "current_milestone": 0,
  "milestones_completed": [],
  "started_at": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP"
}
```

---

## Phase 0: Milestone Selection

Read `.roadmap/roadmap.md` and extract all milestones.

Present the milestone list to the user:

```
## Roadmap Milestones

1. [Milestone 1 name] — [brief description]
2. [Milestone 2 name] — [brief description]
3. [Milestone 3 name] — [brief description]
...

Which milestone would you like to start with? (default: 1)
```

If `--milestone N` was provided as an argument, select milestone N directly without prompting.

**Output file:** `.build/00-current-milestone.md`

```markdown
# Current Milestone

## Milestone [N]: [Name]

**Goal:** [Goal from roadmap]

**Features:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

**Dependencies:**
- [Any prerequisites or prior milestones that must be complete]
```

Update `state.json`: set `current_milestone` to N, set `flags.milestone` to N.

---

## Phase 1: Milestone Planning

Read `.build/00-current-milestone.md` to load the selected milestone.

Launch the milestone planner agent:

```
Agent:
  subagent_type: "milestone-planner"
  description: "Break milestone into concrete implementation tasks"
  prompt: |
    Break this milestone into concrete, ordered implementation tasks.

    ## Current Milestone
    [Insert full contents of .build/00-current-milestone.md]

    Survey the existing codebase to understand current state, patterns, and conventions.
    Follow your analysis strategy. Output in your defined format.
```

Save the planner's output to `.build/01-milestone-plan.md`.

### User Approval Gate

Present the milestone plan to the user:

```
## Milestone Plan: [Name]

[Summary of tasks from the plan]

Total tasks: [X]
Estimated scope: [X tasks x 15-45 min each]

1. Approve — begin TDD build loop
2. Modify — tell me what to change, and I will replan
3. Skip — move to the next milestone
```

**If the user approves:** Proceed to Phase 2.

**If the user requests modifications:** Update the plan based on feedback and re-present for approval.

**If the user skips:** Mark milestone as skipped, return to Phase 0 for next milestone selection.

Update `state.json`: add phase 1 to completed state.

---

## Phase 2: Build Loop (Per Task)

Read `.build/01-milestone-plan.md` to load the task list.

For each task in the plan, execute the following steps:

### Step A: Write Tests (Red Phase)

Launch the test-writer agent:

```
Agent:
  subagent_type: "test-writer"
  description: "Write failing tests for task [N]: [task name]"
  prompt: |
    Write tests for this task. Tests MUST fail initially — this is the TDD red phase.

    ## Task
    [Insert task details from .build/01-milestone-plan.md]

    ## Milestone Context
    [Insert relevant context from .build/00-current-milestone.md]

    Read the existing codebase to understand test conventions, frameworks, and patterns.
    Follow your analysis strategy. Write the test files and report results.
```

After the test-writer completes, verify the tests fail by running them:

- If tests **fail** (expected): Proceed to Step B.
- If tests **pass** without implementation: The tests are wrong — they don't test the new feature. Ask the test-writer to rewrite them.
- If tests have **syntax errors**: Ask the test-writer to fix them.

### Step B: Implement (Green Phase)

Launch the implementer agent:

```
Agent:
  subagent_type: "implementer"
  description: "Implement task [N]: [task name] to make tests pass"
  prompt: |
    Write the minimum code to make all failing tests pass. This is the TDD green phase.

    ## Task
    [Insert task details from .build/01-milestone-plan.md]

    ## Test Files
    [List the test files created in Step A]

    Read the failing tests to understand exactly what behavior is expected.
    Read the existing codebase for patterns and conventions.
    Follow your analysis strategy. Write the implementation and report results.
```

After the implementer completes, run the tests:

- If tests **pass**: Proceed to Step C.
- If tests **fail**: Give the implementer the failure output and ask it to fix. Iterate up to 3 times. If still failing after 3 attempts, STOP and present the error to the user.

### Step C: Code Review

Launch the code-reviewer agent:

```
Agent:
  subagent_type: "code-reviewer"
  description: "Review implementation of task [N]: [task name]"
  prompt: |
    Review the implementation for this task. Focus on bugs, security, and convention violations.

    ## Task
    [Insert task details from .build/01-milestone-plan.md]

    ## Files Changed
    [List all test and implementation files created or modified]

    Read the test files and implementation files.
    Follow your analysis strategy. Provide your verdict.
```

Process the review verdict:

- **APPROVE:** Log the task as complete, move to the next task.
- **REQUEST CHANGES:** Send the critical issues back to the implementer agent (return to Step B with review feedback). If the reviewer requests changes more than 2 times on the same task, STOP and present the issues to the user.

Save task progress to `.build/progress.md` after each task completes.

---

## Phase 3: Milestone Completion

After all tasks in the milestone are complete:

1. **Run full test suite** — Execute all project tests, not just the ones added in this milestone. If any tests fail, STOP and present failures.

2. **Update progress** — Save completion status to `.build/progress.md`:

```markdown
# Build Progress

## Milestone [N]: [Name] — COMPLETE

### Tasks Completed
- [x] Task 1: [name] — [files changed]
- [x] Task 2: [name] — [files changed]
- [x] Task 3: [name] — [files changed]

### Test Summary
- Tests added: [X]
- Tests passing: [X]
- Total project tests: [X]

### Files Changed
- Created: [list]
- Modified: [list]
```

3. **Commit** — Stage all changes and commit:

```
git add -A && git commit -m "feat: complete milestone [N] - [milestone name]"
```

4. **Present completion summary** to the user:

```
Milestone [N] complete: [Name]

## Summary
- Tasks completed: [X/X]
- Tests added: [X]
- Files created: [X]
- Files modified: [X]

## What's Next
1. Continue to milestone [N+1]: [next milestone name]
2. Stop here (progress saved in .build/)
```

**If the user continues:** Return to Phase 0 to select the next milestone. Add milestone N to `milestones_completed` in `state.json`.

**If the user stops:** Save state and exit. Progress is preserved for `--resume`.

**If all milestones are done:** Set `state.json` status to `"complete"`.

---

## Completion

When all milestones are complete, present the full progress report:

```
Build complete! All milestones implemented.

## Progress Report
- Milestones completed: [X/X]
- Total tasks: [X]
- Total tests added: [X]
- Total files created: [X]
- Total files modified: [X]

## Milestones
1. [Milestone 1] — COMPLETE
2. [Milestone 2] — COMPLETE
3. [Milestone 3] — COMPLETE

## Output
- Build state: .build/state.json
- Progress log: .build/progress.md
- Milestone plan: .build/01-milestone-plan.md

All work has been committed to the repository.
```
