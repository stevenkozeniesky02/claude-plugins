# solodev CLI Design Document

**Date:** 2026-03-03
**Status:** Approved

## Overview

solodev is an interactive TUI and direct CLI that orchestrates the 18 Solo Dev Toolbox plugins through a guided pipeline experience with persistent context across every step. It wraps Claude Code as a subprocess, maintaining a single session per project so that context accumulates — by the time you run `/comply`, Claude knows every decision from `/spark` onward.

**Install:** `npm install -g solodev`
**Interactive:** `solodev`
**Direct:** `solodev spark "my idea"`

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                   solodev TUI                    │
│          (Ink + React terminal components)        │
│                                                   │
│  Screens: Auth → Project Select → Dashboard → Run │
├─────────────────────────────────────────────────┤
│               Orchestration Layer                │
│  ┌───────────┐  ┌───────────┐  ┌─────────────┐ │
│  │  Project   │  │  Pipeline │  │   Session    │ │
│  │  Manager   │  │  Tracker  │  │   Store      │ │
│  └───────────┘  └───────────┘  └─────────────┘ │
├─────────────────────────────────────────────────┤
│              Claude Code Subprocess              │
│                                                   │
│  claude -p "prompt"                               │
│    --resume SESSION_ID                            │
│    --plugin-dir ./plugins                         │
│    --output-format stream-json                    │
│    --permission-mode <from project config>        │
├─────────────────────────────────────────────────┤
│              Bundled Plugins (18)                 │
│   spark, stack, scaffold, generate, build, ...   │
└─────────────────────────────────────────────────┘
```

### Orchestration Modules

- **Project Manager** — CRUD for projects stored in `~/.config/solodev/projects.json`. Each project has a name, path, permission level, and session ID.
- **Pipeline Tracker** — Scans for `.spark/`, `.stack/`, `.preflight/`, etc. to determine what's been run. Calculates "what's next" and highlights it on the dashboard.
- **Session Store** — Saves the Claude Code session ID per project. Every plugin run resumes the same session, so context accumulates. If the user starts a new project, a fresh session is created.

### Why Subprocess Over Agent SDK

Claude Code's `-p` flag with `--resume SESSION_ID` provides full conversation persistence across calls. Combined with `--output-format stream-json` for streaming and `--plugin-dir` for plugin loading, the subprocess approach gives us everything we need while letting users authenticate with their existing Claude subscription (Pro/Max) instead of requiring API keys.

The Agent SDK requires API keys and pay-per-use billing. It remains a potential future enhancement for CI/CD and headless use cases.

---

## Screen Flow

```
Auth Check → Project Select → Dashboard → Run View
                  ↑               │            │
                  └───────────────┘            │
                        ↑                      │
                        └──────────────────────┘
```

### Screen 1: Auth Check

Runs `claude auth status --output-format json` on startup.

- If authenticated: shows email + plan, auto-advances after 1 second
- If not authenticated: prompts to log in, runs `claude auth login` (opens browser)

```
┌─────────────────────────────────────────────────┐
│                                                   │
│   ███████╗ ██████╗ ██╗      ██████╗              │
│   ██╔════╝██╔═══██╗██║     ██╔═══██╗             │
│   ███████╗██║   ██║██║     ██║   ██║             │
│   ╚════██║██║   ██║██║     ██║   ██║             │
│   ███████║╚██████╔╝███████╗╚██████╔╝             │
│   ╚══════╝ ╚═════╝ ╚══════╝ ╚═════╝  DEV        │
│                                                   │
│   Checking Claude connection...                   │
│                                                   │
│   ✅ steven@gmail.com (Max plan)                  │
│                                                   │
│   Press Enter to continue                         │
│                                                   │
└─────────────────────────────────────────────────┘
```

### Screen 2: Project Select

Lists registered projects from `projects.json`, sorted by `lastOpenedAt`. Shows pipeline progress count.

```
┌─────────────────────────────────────────────────┐
│  SOLODEV                              v1.0.0     │
├─────────────────────────────────────────────────┤
│                                                   │
│  Your Projects:                                   │
│                                                   │
│  > my-saas-app       ~/projects/saas    6 steps   │
│    osint-engine      ~/osint-engine     2 steps   │
│    side-project      ~/code/sidething   new       │
│                                                   │
│  ─────────────────────────────────────────────   │
│                                                   │
│  [n] New project from idea                        │
│  [o] Open existing directory                      │
│  [Enter] Select project                           │
│  [d] Remove from list                             │
│  [q] Quit                                         │
│                                                   │
└─────────────────────────────────────────────────┘
```

- `[n]` prompts for idea description, creates directory, starts with `/spark`
- `[o]` prompts for path, registers existing codebase, auto-detects pipeline state
- "6 steps" = count of completed dot-directories found
- "new" = registered but no plugins run yet

### Screen 3: Dashboard

Home base. Pipeline visualization with smart "next step" suggestion.

```
┌─────────────────────────────────────────────────┐
│  my-saas-app                ~/projects/saas      │
├─────────────────────────────────────────────────┤
│                                                   │
│  Pipeline:                                        │
│                                                   │
│  ✅  1. Idea Validation       /spark     .spark/  │
│  ✅  2. Tech Stack             /stack     .stack/  │
│  ✅  3. Scaffold               /scaffold  .scaffold│
│  ✅  4. Roadmap                /generate  .roadmap/│
│  🔨  5. Build                  /build     .build/  │
│  ⬚   6. Test Coverage          /coverage           │
│  ⬚   7. Dependencies           /checkup            │
│  ⬚   8. Production Readiness   /preflight          │
│  ⬚   9. Cost Forecast          /estimate           │
│  ⬚  10. Scale Analysis         /simulate           │
│  ⬚  11. Incident Runbooks      /runbook            │
│  ⬚  12. Landing Page           /landing            │
│  ⬚  13. Pricing Strategy       /pricing            │
│  ⬚  14. Pitch Deck             /pitch              │
│  ⬚  15. Legal & Compliance     /comply             │
│                                                   │
│  → Continue building? (milestone 2)               │
│                                                   │
│  [Enter] Run next  [↑↓] Select step  [v] View    │
│  [s] Settings  [p] Projects  [q] Quit             │
│                                                   │
└─────────────────────────────────────────────────┘
```

- State detection: scans for dot-directories + reads `state.json` status field
- 🔨 = `state.json` exists with `status: "in_progress"`
- ✅ = `state.json` exists with `status: "complete"`
- ⬚ = no dot-directory found
- Smart suggestion: not strictly sequential. If code exists but no tests, suggests `/coverage`
- `[v]` on a completed step opens the output file
- `[↑↓]` allows jumping to any step — pipeline is not locked to order
- Utility plugins (`/discover`, `/autopsy`, `/onboard`) accessible via `[m] More tools`

### Screen 4: Run View

Shows real-time execution progress with collapsible live output stream.

```
┌─────────────────────────────────────────────────┐
│  Running: /coverage                    ◐ 1m 23s  │
├─────────────────────────────────────────────────┤
│                                                   │
│  ✅ Phase 0: Codebase survey              12s     │
│  🔄 Phase 1: Running specialists...      1m 11s  │
│     ├── coverage-mapper        ✅ done            │
│     ├── risk-path-finder       🔄 running...      │
│     ├── test-strategist        ⏳ waiting          │
│     └── skeleton-writer        ⏳ waiting          │
│  ⬚  Phase 2: Synthesis                            │
│                                                   │
│  ─────────────────────────────────────────────   │
│  [Space] Expand live output  [c] Cancel  [q] Back │
│                                                   │
└─────────────────────────────────────────────────┘
```

Press `[Space]` to expand:

```
│  ─────────── Live Output ─────────────────────   │
│  > Scanning src/api/ for untested endpoints...    │
│  > Found 23 source files, 8 test files            │
│  > Coverage ratio: 34% (8/23 modules)             │
│  > Analyzing risk paths in auth middleware...      │
│                                                   │
│  [Space] Collapse  [c] Cancel  [q] Back           │
```

When complete, shows results summary and returns to dashboard with step marked ✅.

---

## Data Model

### Global Config: `~/.config/solodev/config.json`

```json
{
  "version": 1,
  "pluginDir": null,
  "defaultPermissionMode": "default",
  "editor": "$EDITOR"
}
```

- `pluginDir`: null = use bundled plugins. Set to a path for custom/updated plugins.
- `defaultPermissionMode`: applied to new projects. Overridable per-project.

### Project Registry: `~/.config/solodev/projects.json`

```json
{
  "projects": [
    {
      "id": "a1b2c3d4",
      "name": "my-saas-app",
      "path": "/Users/steven/projects/saas",
      "sessionId": "550e8400-e29b-41d4-a716-446655440000",
      "permissionMode": "acceptEdits",
      "createdAt": "2026-03-03T10:00:00Z",
      "lastOpenedAt": "2026-03-03T15:30:00Z"
    }
  ]
}
```

- `sessionId`: the Claude Code session that persists across all plugin runs for this project
- `permissionMode`: per-project trust level — `"default"`, `"acceptEdits"`, or `"bypassPermissions"`

### Pipeline State

Not stored by solodev. Detected by scanning the project directory:

```
For each plugin in pipeline order:
  1. Check if .{plugin}/ directory exists
  2. If yes, read state.json → status field
     - "complete" → ✅
     - "in_progress" → 🔨
  3. If no directory → ⬚ (not started)
```

Plugins are the source of truth. No duplicate state.

---

## Session Lifecycle

### First plugin run (new project)

```
1. solodev generates UUID → sessionId
2. Saves to projects.json
3. Runs: claude -p "/spark \"idea\"" --session-id UUID --plugin-dir ... --output-format stream-json
4. Claude Code stores session with full conversation history
```

### Subsequent plugin runs

```
1. solodev reads sessionId from projects.json
2. Runs: claude -p "/stack" --resume SESSION_ID --plugin-dir ... --output-format stream-json
3. Claude resumes with full context from all prior runs
4. /stack recommendations informed by everything from /spark
```

### Context accumulation

```
Run 1:  /spark    → Claude knows the idea
Run 2:  /stack    → Claude knows idea + stack decision
Run 3:  /scaffold → Claude knows idea + stack + project structure
...
Run 15: /comply   → Claude has the ENTIRE product history in context
```

### Edge cases

- **Session too long:** Claude Code auto-compresses older messages at context limits. Handled automatically.
- **Corrupted/expired session:** solodev creates a fresh session, warns user. Pipeline state preserved via dot-directories.
- **Manual reset:** `solodev reset` creates fresh session. `solodev reset --hard` also deletes dot-directories.

---

## CLI Modes

### Interactive mode

```bash
solodev              # Opens TUI: auth → project select → dashboard
```

### Direct commands

```bash
solodev spark "my idea"                  # Run /spark on current directory
solodev coverage --generate-skeletons    # Run /coverage with flags
solodev status                           # Print pipeline status
solodev projects                         # List registered projects
solodev open ~/path/to/project           # Register + open project in TUI
solodev reset                            # Fresh session, keep outputs
solodev reset --hard                     # Fresh session + delete outputs
solodev settings                         # Open global config in $EDITOR
solodev update-plugins                   # Pull latest plugin versions
solodev version                          # Show version
```

Direct commands skip the TUI. They auto-register the current directory, look up the session ID, run the plugin, and stream output to stdout. Context persists across direct calls too.

---

## Package Structure

```
solodev/
├── package.json
├── tsconfig.json
├── src/
│   ├── cli.ts                   # Entry point — arg parsing, route to TUI or direct
│   ├── tui/
│   │   ├── App.tsx              # Root Ink component — screen router
│   │   ├── screens/
│   │   │   ├── AuthScreen.tsx
│   │   │   ├── ProjectScreen.tsx
│   │   │   ├── DashboardScreen.tsx
│   │   │   └── RunScreen.tsx
│   │   └── components/
│   │       ├── PipelineList.tsx  # ✅🔨⬚ pipeline display
│   │       ├── LiveStream.tsx   # Collapsible streaming output
│   │       ├── Spinner.tsx      # Phase progress indicator
│   │       └── Header.tsx       # Top bar
│   ├── core/
│   │   ├── claude.ts            # Subprocess wrapper — spawn, stream, resume
│   │   ├── session.ts           # Session ID management per project
│   │   ├── pipeline.ts          # Dot-directory scanning + state detection
│   │   ├── projects.ts          # Project CRUD
│   │   └── config.ts            # Global config read/write
│   ├── commands/
│   │   ├── spark.ts             # Direct mode: solodev spark
│   │   ├── status.ts            # Direct mode: solodev status
│   │   ├── reset.ts             # Direct mode: solodev reset
│   │   └── index.ts             # Command registry
│   └── plugins/                 # Bundled (copied at build time)
│       ├── idea-spark/
│       ├── stack-pick/
│       └── ... (all 18)
├── scripts/
│   └── bundle-plugins.ts        # Copies plugins from source repo at build
└── dist/
```

### Dependencies

| Package | Purpose |
|---------|---------|
| `ink` | React for CLI — terminal components |
| `ink-select-input` | Arrow-key selection lists |
| `ink-spinner` | Loading spinners |
| `ink-text-input` | Text input fields |
| `commander` | CLI arg parsing for direct mode |
| `xdg-basedir` | Cross-platform config paths (~/.config/) |
| `uuid` | Generate session IDs |

### Build & Publish

```bash
npm run build          # tsc + bundle plugins
npm publish            # Ships to npm as solodev
```

### User Install

```bash
npm install -g solodev
solodev
```

Requires Claude Code to be installed and authenticated (`claude login`).

---

## Tech Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | TypeScript | Same ecosystem as Claude Code and plugins |
| TUI framework | Ink (React for CLI) | Component model, composable, well-maintained |
| CLI parser | Commander | Standard, lightweight, handles subcommands |
| Config location | `~/.config/solodev/` | XDG-compliant, works on Linux + macOS |
| State storage | JSON files | No database needed, human-readable, trivial to debug |
| Pipeline state | Dot-directory scanning | Single source of truth, no sync issues |
| Context persistence | Claude Code `--resume` | Full conversation history, no API key required |
| Streaming | `--output-format stream-json` | Real-time progress for the collapsible live view |
| Plugin delivery | Bundled in npm package | One install, zero config |
| Plugin updates | `solodev update-plugins` | Pull from GitHub, overwrite bundled copy |
