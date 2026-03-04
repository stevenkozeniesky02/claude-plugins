# solodev CLI Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build an interactive TUI + direct CLI that orchestrates the 18 Solo Dev Toolbox plugins through a guided pipeline with persistent Claude Code session context.

**Architecture:** TypeScript + Ink (React for CLI) TUI with Commander for direct commands. Orchestrates Claude Code as a subprocess with `--resume` for session persistence. Bundled plugins, XDG config, collapsible live streaming.

**Tech Stack:** TypeScript, Ink 6.x, React 18, Commander, fullscreen-ink, xdg-basedir, uuid, Node.js 18+

**Design Doc:** `docs/plans/2026-03-03-solodev-cli-design.md`

---

## Important Notes for the Implementer

- Ink is **pure ESM**. The package.json must have `"type": "module"`.
- With `"module": "NodeNext"` in tsconfig, all local imports must use `.js` extensions: `import { foo } from './foo.js'`
- All bare text in JSX must be wrapped in `<Text>`. Bare strings cause runtime errors.
- Use `fullscreen-ink` for the alternate screen buffer (TUI behaves like vim/htop).
- `useInput` competes with `TextInput` — only one can be active at a time. Use `isActive` / `focus` props to manage this.
- Source goes in `source/` (Ink convention), compiles to `dist/`.

---

### Task 1: Project Scaffold

**Files:**
- Create: `solodev/package.json`
- Create: `solodev/tsconfig.json`
- Create: `solodev/.gitignore`

**Step 1: Create the project directory**

```bash
mkdir -p /Users/steven/solodev
cd /Users/steven/solodev
git init
```

**Step 2: Create package.json**

```json
{
  "name": "solodev",
  "version": "0.1.0",
  "description": "Interactive CLI for the Solo Dev Toolbox — 18 plugins, one pipeline, persistent context",
  "type": "module",
  "bin": {
    "solodev": "./dist/cli.js"
  },
  "main": "./dist/cli.js",
  "files": [
    "dist",
    "plugins"
  ],
  "engines": {
    "node": ">=18.0.0"
  },
  "scripts": {
    "build": "tsc && chmod +x dist/cli.js",
    "dev": "tsc --watch",
    "start": "node dist/cli.js",
    "test": "node --experimental-vm-modules node_modules/.bin/jest",
    "prepublishOnly": "npm run build"
  },
  "keywords": ["claude", "cli", "tui", "solo-developer", "ai", "toolbox"],
  "author": "Steven Kozeniesky",
  "license": "MIT",
  "dependencies": {
    "ink": "^6.8.0",
    "react": "^18.3.1",
    "ink-select-input": "^6.2.0",
    "ink-text-input": "^6.0.0",
    "ink-spinner": "^5.0.0",
    "fullscreen-ink": "^2.0.0",
    "commander": "^13.0.0",
    "xdg-basedir": "^5.1.0",
    "uuid": "^11.0.0"
  },
  "devDependencies": {
    "typescript": "^5.8.0",
    "@types/react": "^18.3.0",
    "@types/node": "^22.0.0",
    "@types/uuid": "^10.0.0",
    "jest": "^29.7.0",
    "@jest/globals": "^29.7.0",
    "ts-jest": "^29.2.0"
  }
}
```

**Step 3: Create tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "jsx": "react",
    "esModuleInterop": true,
    "outDir": "dist",
    "rootDir": "source",
    "strict": true,
    "sourceMap": true,
    "declaration": true,
    "skipLibCheck": true
  },
  "include": ["source"]
}
```

**Step 4: Create .gitignore**

```
node_modules/
dist/
*.tsbuildinfo
```

**Step 5: Install dependencies**

```bash
cd /Users/steven/solodev
npm install
```

**Step 6: Commit**

```bash
git add package.json tsconfig.json .gitignore package-lock.json
git commit -m "feat: scaffold solodev project with dependencies"
```

---

### Task 2: Core — Config Module

**Files:**
- Create: `solodev/source/core/config.ts`
- Create: `solodev/source/core/__tests__/config.test.ts`

**Step 1: Write the failing test**

```typescript
// source/core/__tests__/config.test.ts
import { describe, it, expect, beforeEach, afterEach } from '@jest/globals';
import fs from 'fs';
import path from 'path';
import os from 'os';

// We'll test getConfigDir, loadConfig, saveConfig
describe('config', () => {
  let tmpDir: string;

  beforeEach(() => {
    tmpDir = fs.mkdtempSync(path.join(os.tmpdir(), 'solodev-test-'));
    process.env['XDG_CONFIG_HOME'] = tmpDir;
  });

  afterEach(() => {
    fs.rmSync(tmpDir, { recursive: true, force: true });
    delete process.env['XDG_CONFIG_HOME'];
  });

  it('should return config dir under XDG_CONFIG_HOME', async () => {
    const { getConfigDir } = await import('../config.js');
    const dir = getConfigDir();
    expect(dir).toBe(path.join(tmpDir, 'solodev'));
  });

  it('should return default config when no file exists', async () => {
    const { loadConfig } = await import('../config.js');
    const config = loadConfig();
    expect(config).toEqual({
      version: 1,
      pluginDir: null,
      defaultPermissionMode: 'default',
      editor: null,
    });
  });

  it('should save and load config', async () => {
    const { saveConfig, loadConfig, getConfigDir } = await import('../config.js');
    fs.mkdirSync(getConfigDir(), { recursive: true });
    saveConfig({ version: 1, pluginDir: '/custom', defaultPermissionMode: 'acceptEdits', editor: 'vim' });
    const loaded = loadConfig();
    expect(loaded.pluginDir).toBe('/custom');
    expect(loaded.defaultPermissionMode).toBe('acceptEdits');
  });
});
```

**Step 2: Run test to verify it fails**

```bash
cd /Users/steven/solodev
npm test -- --testPathPattern=config
```
Expected: FAIL — module not found

**Step 3: Write minimal implementation**

```typescript
// source/core/config.ts
import fs from 'fs';
import path from 'path';
import os from 'os';

export interface SolodevConfig {
  version: number;
  pluginDir: string | null;
  defaultPermissionMode: 'default' | 'acceptEdits' | 'bypassPermissions';
  editor: string | null;
}

const DEFAULT_CONFIG: SolodevConfig = {
  version: 1,
  pluginDir: null,
  defaultPermissionMode: 'default',
  editor: null,
};

export function getConfigDir(): string {
  const xdgConfig = process.env['XDG_CONFIG_HOME'] || path.join(os.homedir(), '.config');
  return path.join(xdgConfig, 'solodev');
}

function getConfigPath(): string {
  return path.join(getConfigDir(), 'config.json');
}

export function loadConfig(): SolodevConfig {
  const configPath = getConfigPath();
  if (!fs.existsSync(configPath)) {
    return { ...DEFAULT_CONFIG };
  }
  const raw = fs.readFileSync(configPath, 'utf-8');
  return { ...DEFAULT_CONFIG, ...JSON.parse(raw) };
}

export function saveConfig(config: SolodevConfig): void {
  const configPath = getConfigPath();
  fs.mkdirSync(path.dirname(configPath), { recursive: true });
  fs.writeFileSync(configPath, JSON.stringify(config, null, 2) + '\n');
}
```

**Step 4: Run test to verify it passes**

```bash
npm test -- --testPathPattern=config
```
Expected: PASS

**Step 5: Commit**

```bash
git add source/core/config.ts source/core/__tests__/config.test.ts
git commit -m "feat: add config module with XDG-compliant paths"
```

---

### Task 3: Core — Projects Module

**Files:**
- Create: `solodev/source/core/projects.ts`
- Create: `solodev/source/core/__tests__/projects.test.ts`

**Step 1: Write the failing test**

```typescript
// source/core/__tests__/projects.test.ts
import { describe, it, expect, beforeEach, afterEach } from '@jest/globals';
import fs from 'fs';
import path from 'path';
import os from 'os';

describe('projects', () => {
  let tmpDir: string;

  beforeEach(() => {
    tmpDir = fs.mkdtempSync(path.join(os.tmpdir(), 'solodev-test-'));
    process.env['XDG_CONFIG_HOME'] = tmpDir;
  });

  afterEach(() => {
    fs.rmSync(tmpDir, { recursive: true, force: true });
    delete process.env['XDG_CONFIG_HOME'];
  });

  it('should return empty list when no projects file exists', async () => {
    const { listProjects } = await import('../projects.js');
    const projects = listProjects();
    expect(projects).toEqual([]);
  });

  it('should add and retrieve a project', async () => {
    const { addProject, listProjects, getConfigDir } = await import('../projects.js');
    fs.mkdirSync(path.join(tmpDir, 'solodev'), { recursive: true });
    const project = addProject('test-app', '/tmp/test-app');
    expect(project.name).toBe('test-app');
    expect(project.path).toBe('/tmp/test-app');
    expect(project.sessionId).toBeTruthy();
    expect(project.permissionMode).toBe('default');
    const all = listProjects();
    expect(all).toHaveLength(1);
    expect(all[0].name).toBe('test-app');
  });

  it('should get a project by id', async () => {
    const { addProject, getProject } = await import('../projects.js');
    fs.mkdirSync(path.join(tmpDir, 'solodev'), { recursive: true });
    const added = addProject('test-app', '/tmp/test-app');
    const found = getProject(added.id);
    expect(found).toBeTruthy();
    expect(found!.name).toBe('test-app');
  });

  it('should remove a project', async () => {
    const { addProject, removeProject, listProjects } = await import('../projects.js');
    fs.mkdirSync(path.join(tmpDir, 'solodev'), { recursive: true });
    const added = addProject('test-app', '/tmp/test-app');
    removeProject(added.id);
    expect(listProjects()).toHaveLength(0);
  });

  it('should update lastOpenedAt', async () => {
    const { addProject, touchProject, getProject } = await import('../projects.js');
    fs.mkdirSync(path.join(tmpDir, 'solodev'), { recursive: true });
    const added = addProject('test-app', '/tmp/test-app');
    const before = added.lastOpenedAt;
    // Small delay to ensure timestamp differs
    await new Promise(r => setTimeout(r, 10));
    touchProject(added.id);
    const updated = getProject(added.id);
    expect(updated!.lastOpenedAt >= before).toBe(true);
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npm test -- --testPathPattern=projects
```
Expected: FAIL

**Step 3: Write minimal implementation**

```typescript
// source/core/projects.ts
import fs from 'fs';
import path from 'path';
import { v4 as uuidv4 } from 'uuid';
import { getConfigDir } from './config.js';

export interface Project {
  id: string;
  name: string;
  path: string;
  sessionId: string;
  permissionMode: 'default' | 'acceptEdits' | 'bypassPermissions';
  createdAt: string;
  lastOpenedAt: string;
}

interface ProjectStore {
  projects: Project[];
}

function getProjectsPath(): string {
  return path.join(getConfigDir(), 'projects.json');
}

function loadStore(): ProjectStore {
  const filePath = getProjectsPath();
  if (!fs.existsSync(filePath)) {
    return { projects: [] };
  }
  return JSON.parse(fs.readFileSync(filePath, 'utf-8'));
}

function saveStore(store: ProjectStore): void {
  const filePath = getProjectsPath();
  fs.mkdirSync(path.dirname(filePath), { recursive: true });
  fs.writeFileSync(filePath, JSON.stringify(store, null, 2) + '\n');
}

export function listProjects(): Project[] {
  return loadStore().projects.sort(
    (a, b) => new Date(b.lastOpenedAt).getTime() - new Date(a.lastOpenedAt).getTime()
  );
}

export function getProject(id: string): Project | undefined {
  return loadStore().projects.find(p => p.id === id);
}

export function getProjectByPath(projectPath: string): Project | undefined {
  return loadStore().projects.find(p => p.path === projectPath);
}

export function addProject(name: string, projectPath: string): Project {
  const store = loadStore();
  const now = new Date().toISOString();
  const project: Project = {
    id: uuidv4().slice(0, 8),
    name,
    path: projectPath,
    sessionId: uuidv4(),
    permissionMode: 'default',
    createdAt: now,
    lastOpenedAt: now,
  };
  store.projects.push(project);
  saveStore(store);
  return project;
}

export function removeProject(id: string): void {
  const store = loadStore();
  store.projects = store.projects.filter(p => p.id !== id);
  saveStore(store);
}

export function touchProject(id: string): void {
  const store = loadStore();
  const project = store.projects.find(p => p.id === id);
  if (project) {
    project.lastOpenedAt = new Date().toISOString();
    saveStore(store);
  }
}

export function updateProjectSession(id: string, sessionId: string): void {
  const store = loadStore();
  const project = store.projects.find(p => p.id === id);
  if (project) {
    project.sessionId = sessionId;
    saveStore(store);
  }
}

export { getConfigDir };
```

**Step 4: Run test to verify it passes**

```bash
npm test -- --testPathPattern=projects
```
Expected: PASS

**Step 5: Commit**

```bash
git add source/core/projects.ts source/core/__tests__/projects.test.ts
git commit -m "feat: add projects module with CRUD and session tracking"
```

---

### Task 4: Core — Pipeline Tracker

**Files:**
- Create: `solodev/source/core/pipeline.ts`
- Create: `solodev/source/core/__tests__/pipeline.test.ts`

**Step 1: Write the failing test**

```typescript
// source/core/__tests__/pipeline.test.ts
import { describe, it, expect, beforeEach, afterEach } from '@jest/globals';
import fs from 'fs';
import path from 'path';
import os from 'os';

describe('pipeline', () => {
  let tmpDir: string;

  beforeEach(() => {
    tmpDir = fs.mkdtempSync(path.join(os.tmpdir(), 'solodev-test-'));
  });

  afterEach(() => {
    fs.rmSync(tmpDir, { recursive: true, force: true });
  });

  it('should return all steps as not_started for empty directory', async () => {
    const { scanPipeline } = await import('../pipeline.js');
    const steps = scanPipeline(tmpDir);
    expect(steps.length).toBeGreaterThan(0);
    expect(steps.every(s => s.status === 'not_started')).toBe(true);
  });

  it('should detect a completed step', async () => {
    const { scanPipeline } = await import('../pipeline.js');
    const sparkDir = path.join(tmpDir, '.spark');
    fs.mkdirSync(sparkDir);
    fs.writeFileSync(path.join(sparkDir, 'state.json'), JSON.stringify({ status: 'complete' }));
    const steps = scanPipeline(tmpDir);
    const spark = steps.find(s => s.command === '/spark');
    expect(spark!.status).toBe('complete');
  });

  it('should detect an in-progress step', async () => {
    const { scanPipeline } = await import('../pipeline.js');
    const buildDir = path.join(tmpDir, '.build');
    fs.mkdirSync(buildDir);
    fs.writeFileSync(path.join(buildDir, 'state.json'), JSON.stringify({ status: 'in_progress' }));
    const steps = scanPipeline(tmpDir);
    const build = steps.find(s => s.command === '/build');
    expect(build!.status).toBe('in_progress');
  });

  it('should detect directory without state.json as complete', async () => {
    const { scanPipeline } = await import('../pipeline.js');
    const sparkDir = path.join(tmpDir, '.spark');
    fs.mkdirSync(sparkDir);
    // No state.json — directory exists, assume complete
    const steps = scanPipeline(tmpDir);
    const spark = steps.find(s => s.command === '/spark');
    expect(spark!.status).toBe('complete');
  });

  it('should suggest next step', async () => {
    const { scanPipeline, suggestNext } = await import('../pipeline.js');
    const steps = scanPipeline(tmpDir);
    const next = suggestNext(steps);
    expect(next).toBeTruthy();
    expect(next!.command).toBe('/spark'); // first step when nothing done
  });

  it('should count completed steps', async () => {
    const { scanPipeline, countCompleted } = await import('../pipeline.js');
    fs.mkdirSync(path.join(tmpDir, '.spark'));
    fs.mkdirSync(path.join(tmpDir, '.stack'));
    const steps = scanPipeline(tmpDir);
    expect(countCompleted(steps)).toBe(2);
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npm test -- --testPathPattern=pipeline
```
Expected: FAIL

**Step 3: Write minimal implementation**

```typescript
// source/core/pipeline.ts

export interface PipelineStep {
  index: number;
  name: string;
  command: string;
  dotDir: string;
  pluginName: string;
  status: 'not_started' | 'in_progress' | 'complete';
  category: 'validate' | 'build' | 'harden' | 'ship';
}

const PIPELINE_STEPS: Omit<PipelineStep, 'status'>[] = [
  { index: 1,  name: 'Idea Validation',      command: '/spark',     dotDir: '.spark',       pluginName: 'idea-spark',       category: 'validate' },
  { index: 2,  name: 'Tech Stack',           command: '/stack',     dotDir: '.stack',       pluginName: 'stack-pick',       category: 'validate' },
  { index: 3,  name: 'Scaffold',             command: '/scaffold',  dotDir: '.scaffold',    pluginName: 'scaffold-kit',     category: 'validate' },
  { index: 4,  name: 'Roadmap',              command: '/generate',  dotDir: '.roadmap',     pluginName: 'codebase-roadmap', category: 'build' },
  { index: 5,  name: 'Build',                command: '/build',     dotDir: '.build',       pluginName: 'build-engine',     category: 'build' },
  { index: 6,  name: 'Test Coverage',        command: '/coverage',  dotDir: '.testgap',     pluginName: 'test-gap',         category: 'harden' },
  { index: 7,  name: 'Dependencies',         command: '/checkup',   dotDir: '.depdoctor',   pluginName: 'dep-doctor',       category: 'harden' },
  { index: 8,  name: 'Production Readiness', command: '/preflight', dotDir: '.preflight',   pluginName: 'launch-check',     category: 'harden' },
  { index: 9,  name: 'Cost Forecast',        command: '/estimate',  dotDir: '.costforecast', pluginName: 'cost-forecast',   category: 'harden' },
  { index: 10, name: 'Scale Analysis',       command: '/simulate',  dotDir: '.scalesim',    pluginName: 'scale-sim',        category: 'harden' },
  { index: 11, name: 'Incident Runbooks',    command: '/runbook',   dotDir: '.runbook',     pluginName: 'incident-planner', category: 'harden' },
  { index: 12, name: 'Landing Page',         command: '/landing',   dotDir: '.shippage',    pluginName: 'ship-page',        category: 'ship' },
  { index: 13, name: 'Pricing Strategy',     command: '/pricing',   dotDir: '.pricing',     pluginName: 'price-model',      category: 'ship' },
  { index: 14, name: 'Pitch Deck',           command: '/pitch',     dotDir: '.pitch',       pluginName: 'pitch-craft',      category: 'ship' },
  { index: 15, name: 'Legal & Compliance',   command: '/comply',    dotDir: '.legalkit',    pluginName: 'legal-kit',        category: 'ship' },
];

import fs from 'fs';
import path from 'path';

function detectStatus(projectPath: string, dotDir: string): PipelineStep['status'] {
  const dirPath = path.join(projectPath, dotDir);
  if (!fs.existsSync(dirPath)) return 'not_started';

  const statePath = path.join(dirPath, 'state.json');
  if (!fs.existsSync(statePath)) return 'complete'; // dir exists but no state = assume done

  try {
    const state = JSON.parse(fs.readFileSync(statePath, 'utf-8'));
    if (state.status === 'in_progress') return 'in_progress';
    return 'complete';
  } catch {
    return 'complete'; // malformed state = assume done
  }
}

export function scanPipeline(projectPath: string): PipelineStep[] {
  return PIPELINE_STEPS.map(step => ({
    ...step,
    status: detectStatus(projectPath, step.dotDir),
  }));
}

export function suggestNext(steps: PipelineStep[]): PipelineStep | null {
  // First: any in-progress step
  const inProgress = steps.find(s => s.status === 'in_progress');
  if (inProgress) return inProgress;

  // Then: first not-started step
  const notStarted = steps.find(s => s.status === 'not_started');
  return notStarted || null;
}

export function countCompleted(steps: PipelineStep[]): number {
  return steps.filter(s => s.status === 'complete').length;
}

export { PIPELINE_STEPS };
```

**Step 4: Run test to verify it passes**

```bash
npm test -- --testPathPattern=pipeline
```
Expected: PASS

**Step 5: Commit**

```bash
git add source/core/pipeline.ts source/core/__tests__/pipeline.test.ts
git commit -m "feat: add pipeline tracker with dot-directory scanning"
```

---

### Task 5: Core — Claude Subprocess Wrapper

**Files:**
- Create: `solodev/source/core/claude.ts`
- Create: `solodev/source/core/__tests__/claude.test.ts`

**Step 1: Write the failing test**

```typescript
// source/core/__tests__/claude.test.ts
import { describe, it, expect } from '@jest/globals';

describe('claude', () => {
  it('should build correct args for a new session', async () => {
    const { buildClaudeArgs } = await import('../claude.js');
    const args = buildClaudeArgs({
      prompt: '/spark "AI tool"',
      sessionId: '550e8400-e29b-41d4-a716-446655440000',
      isNewSession: true,
      pluginDir: '/path/to/plugins',
      permissionMode: 'acceptEdits',
    });
    expect(args).toContain('-p');
    expect(args).toContain('/spark "AI tool"');
    expect(args).toContain('--session-id');
    expect(args).toContain('550e8400-e29b-41d4-a716-446655440000');
    expect(args).toContain('--plugin-dir');
    expect(args).toContain('/path/to/plugins');
    expect(args).toContain('--output-format');
    expect(args).toContain('stream-json');
    expect(args).toContain('--permission-mode');
    expect(args).toContain('acceptEdits');
    expect(args).not.toContain('--resume');
  });

  it('should build correct args for resuming a session', async () => {
    const { buildClaudeArgs } = await import('../claude.js');
    const args = buildClaudeArgs({
      prompt: '/stack',
      sessionId: '550e8400-e29b-41d4-a716-446655440000',
      isNewSession: false,
      pluginDir: '/path/to/plugins',
      permissionMode: 'default',
    });
    expect(args).toContain('--resume');
    expect(args).toContain('550e8400-e29b-41d4-a716-446655440000');
    expect(args).not.toContain('--session-id');
  });

  it('should check auth status', async () => {
    const { checkAuth } = await import('../claude.js');
    // This calls the real claude CLI — will pass if claude is installed
    const status = await checkAuth();
    expect(status).toHaveProperty('loggedIn');
    expect(status).toHaveProperty('email');
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npm test -- --testPathPattern=claude
```
Expected: FAIL

**Step 3: Write minimal implementation**

```typescript
// source/core/claude.ts
import { spawn, execSync } from 'child_process';
import { EventEmitter } from 'events';

export interface ClaudeRunOptions {
  prompt: string;
  sessionId: string;
  isNewSession: boolean;
  pluginDir: string;
  permissionMode: 'default' | 'acceptEdits' | 'bypassPermissions';
}

export interface AuthStatus {
  loggedIn: boolean;
  email: string | null;
  subscriptionType: string | null;
}

export interface ClaudeStreamEvent {
  type: 'text' | 'tool_use' | 'result' | 'error' | 'system';
  content: string;
  raw?: unknown;
}

export function buildClaudeArgs(options: ClaudeRunOptions): string[] {
  const args: string[] = ['-p', options.prompt];

  if (options.isNewSession) {
    args.push('--session-id', options.sessionId);
  } else {
    args.push('--resume', options.sessionId);
  }

  args.push('--plugin-dir', options.pluginDir);
  args.push('--output-format', 'stream-json');
  args.push('--permission-mode', options.permissionMode);

  return args;
}

export async function checkAuth(): Promise<AuthStatus> {
  try {
    const output = execSync('claude auth status --output-format json', {
      encoding: 'utf-8',
      timeout: 10000,
      env: { ...process.env, CLAUDECODE: '' },
    });
    const parsed = JSON.parse(output.trim());
    return {
      loggedIn: parsed.loggedIn ?? false,
      email: parsed.email ?? null,
      subscriptionType: parsed.subscriptionType ?? null,
    };
  } catch {
    return { loggedIn: false, email: null, subscriptionType: null };
  }
}

export function runClaude(options: ClaudeRunOptions): {
  events: EventEmitter;
  kill: () => void;
} {
  const args = buildClaudeArgs(options);
  const emitter = new EventEmitter();

  const child = spawn('claude', args, {
    env: { ...process.env, CLAUDECODE: '' },
    shell: false,
  });

  let buffer = '';

  child.stdout.setEncoding('utf8');
  child.stdout.on('data', (chunk: string) => {
    buffer += chunk;
    const lines = buffer.split('\n');
    buffer = lines.pop() || '';

    for (const line of lines) {
      if (!line.trim()) continue;
      try {
        const parsed = JSON.parse(line);
        emitter.emit('message', parsed);
      } catch {
        emitter.emit('message', { type: 'text', content: line });
      }
    }
  });

  child.stderr.setEncoding('utf8');
  child.stderr.on('data', (chunk: string) => {
    emitter.emit('message', { type: 'error', content: chunk });
  });

  child.on('close', (code) => {
    if (buffer.trim()) {
      emitter.emit('message', { type: 'text', content: buffer });
    }
    emitter.emit('done', code);
  });

  child.on('error', (err) => {
    emitter.emit('message', { type: 'error', content: err.message });
    emitter.emit('done', 1);
  });

  return {
    events: emitter,
    kill: () => child.kill(),
  };
}
```

**Step 4: Run test to verify it passes**

```bash
npm test -- --testPathPattern=claude
```
Expected: PASS (the auth test needs claude installed — skip in CI if needed)

**Step 5: Commit**

```bash
git add source/core/claude.ts source/core/__tests__/claude.test.ts
git commit -m "feat: add Claude subprocess wrapper with streaming and auth"
```

---

### Task 6: Core — Plugin Bundler

**Files:**
- Create: `solodev/scripts/bundle-plugins.ts`
- Create: `solodev/source/core/plugins.ts`

**Step 1: Create the bundler script**

```typescript
// scripts/bundle-plugins.ts
import fs from 'fs';
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const SOURCE_DIR = path.resolve(__dirname, '../../claude-plugins/plugins');
const TARGET_DIR = path.resolve(__dirname, '../plugins');

if (!fs.existsSync(SOURCE_DIR)) {
  console.error(`Source plugins not found at ${SOURCE_DIR}`);
  process.exit(1);
}

// Clean target
if (fs.existsSync(TARGET_DIR)) {
  fs.rmSync(TARGET_DIR, { recursive: true });
}

// Copy all plugins
fs.cpSync(SOURCE_DIR, TARGET_DIR, { recursive: true });

const count = fs.readdirSync(TARGET_DIR).filter(f =>
  fs.statSync(path.join(TARGET_DIR, f)).isDirectory()
).length;

console.log(`Bundled ${count} plugins to ${TARGET_DIR}`);
```

**Step 2: Create the plugins resolver**

```typescript
// source/core/plugins.ts
import path from 'path';
import fs from 'fs';
import { fileURLToPath } from 'url';
import { loadConfig } from './config.js';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export function getPluginDir(): string {
  const config = loadConfig();

  // User-configured plugin dir takes priority
  if (config.pluginDir && fs.existsSync(config.pluginDir)) {
    return config.pluginDir;
  }

  // Bundled plugins (relative to dist/core/plugins.js → ../../plugins)
  const bundled = path.resolve(__dirname, '../../plugins');
  if (fs.existsSync(bundled)) {
    return bundled;
  }

  throw new Error('No plugins found. Run solodev update-plugins or set pluginDir in config.');
}

export function listAvailablePlugins(): string[] {
  const dir = getPluginDir();
  return fs.readdirSync(dir).filter(f =>
    fs.statSync(path.join(dir, f)).isDirectory() &&
    fs.existsSync(path.join(dir, f, '.claude-plugin', 'plugin.json'))
  );
}
```

**Step 3: Add bundle script to package.json**

Add to the `scripts` section:
```json
"bundle-plugins": "node --loader ts-node/esm scripts/bundle-plugins.ts",
"build": "tsc && chmod +x dist/cli.js"
```

**Step 4: Commit**

```bash
git add scripts/bundle-plugins.ts source/core/plugins.ts
git commit -m "feat: add plugin bundler and resolver"
```

---

### Task 7: TUI — Entry Point and App Shell

**Files:**
- Create: `solodev/source/cli.tsx`
- Create: `solodev/source/tui/App.tsx`

**Step 1: Create the entry point**

```tsx
#!/usr/bin/env node
// source/cli.tsx
import React from 'react';
import { render } from 'ink';
import { program } from 'commander';
import { App } from './tui/App.js';

const pkg = { version: '0.1.0' };

program
  .name('solodev')
  .version(pkg.version)
  .description('Solo Dev Toolbox — 18 plugins, one pipeline, persistent context')
  .argument('[command]', 'Plugin command to run directly (e.g., spark, coverage)')
  .argument('[args...]', 'Arguments for the command')
  .option('--project <path>', 'Project directory (defaults to cwd)')
  .action((command?: string, args?: string[], options?: { project?: string }) => {
    if (command) {
      // Direct mode — TODO: implement in Task 11
      console.log(`Direct mode: /${command} ${(args || []).join(' ')}`);
      process.exit(0);
    }

    // Interactive TUI mode
    const { waitUntilExit } = render(<App />);
    waitUntilExit().then(() => process.exit(0));
  });

program.parse();
```

**Step 2: Create the App shell with screen router**

```tsx
// source/tui/App.tsx
import React, { useState } from 'react';
import { Box, Text, useApp, useInput } from 'ink';

export type Screen = 'auth' | 'projects' | 'dashboard' | 'run';

export interface AppState {
  projectId: string | null;
  projectPath: string | null;
}

export const App = () => {
  const { exit } = useApp();
  const [screen, setScreen] = useState<Screen>('auth');
  const [appState, setAppState] = useState<AppState>({
    projectId: null,
    projectPath: null,
  });

  // Global quit handler
  useInput((input, key) => {
    if (input === 'q' && screen !== 'run') {
      if (screen === 'auth' || screen === 'projects') {
        exit();
      } else {
        setScreen('projects');
      }
    }
  });

  return (
    <Box flexDirection="column" height={process.stdout.rows}>
      {screen === 'auth' && (
        <Text>Auth screen — coming in Task 8</Text>
      )}
      {screen === 'projects' && (
        <Text>Projects screen — coming in Task 9</Text>
      )}
      {screen === 'dashboard' && (
        <Text>Dashboard screen — coming in Task 10</Text>
      )}
      {screen === 'run' && (
        <Text>Run screen — coming in Task 12</Text>
      )}
    </Box>
  );
};
```

**Step 3: Build and test the shell**

```bash
npm run build
node dist/cli.js --help
node dist/cli.js spark "test"
```
Expected: Help text prints. Direct mode prints placeholder. TUI mode shows auth placeholder.

**Step 4: Commit**

```bash
git add source/cli.tsx source/tui/App.tsx
git commit -m "feat: add CLI entry point and TUI app shell with screen router"
```

---

### Task 8: TUI — Auth Screen

**Files:**
- Create: `solodev/source/tui/screens/AuthScreen.tsx`
- Modify: `solodev/source/tui/App.tsx` — import and render AuthScreen

**Step 1: Create AuthScreen**

```tsx
// source/tui/screens/AuthScreen.tsx
import React, { useState, useEffect } from 'react';
import { Box, Text, useInput } from 'ink';
import Spinner from 'ink-spinner';
import { checkAuth } from '../../core/claude.js';

interface Props {
  onComplete: () => void;
}

type AuthState = 'checking' | 'authenticated' | 'not_authenticated' | 'error';

export const AuthScreen = ({ onComplete }: Props) => {
  const [state, setState] = useState<AuthState>('checking');
  const [email, setEmail] = useState('');
  const [plan, setPlan] = useState('');
  const [error, setError] = useState('');

  useEffect(() => {
    checkAuth().then(status => {
      if (status.loggedIn) {
        setEmail(status.email || 'unknown');
        setPlan(status.subscriptionType || 'unknown');
        setState('authenticated');
        // Auto-advance after 1.5s
        setTimeout(onComplete, 1500);
      } else {
        setState('not_authenticated');
      }
    }).catch(err => {
      setError(err.message);
      setState('error');
    });
  }, []);

  useInput((input, key) => {
    if (state === 'authenticated' && (key.return || input === ' ')) {
      onComplete();
    }
    if (state === 'not_authenticated' && key.return) {
      setState('checking');
      // Trigger login
      import('child_process').then(({ execSync }) => {
        try {
          execSync('claude auth login', { stdio: 'inherit' });
          // Re-check after login
          checkAuth().then(status => {
            if (status.loggedIn) {
              setEmail(status.email || 'unknown');
              setPlan(status.subscriptionType || 'unknown');
              setState('authenticated');
              setTimeout(onComplete, 1500);
            } else {
              setState('not_authenticated');
            }
          });
        } catch {
          setState('error');
          setError('Login failed');
        }
      });
    }
  });

  return (
    <Box flexDirection="column" alignItems="center" justifyContent="center" height={process.stdout.rows}>
      <Text bold color="cyan">
        {`███████╗ ██████╗ ██╗      ██████╗ `}
      </Text>
      <Text bold color="cyan">
        {`██╔════╝██╔═══██╗██║     ██╔═══██╗`}
      </Text>
      <Text bold color="cyan">
        {`███████╗██║   ██║██║     ██║   ██║`}
      </Text>
      <Text bold color="cyan">
        {`╚════██║██║   ██║██║     ██║   ██║`}
      </Text>
      <Text bold color="cyan">
        {`███████║╚██████╔╝███████╗╚██████╔╝`}
      </Text>
      <Text bold color="cyan">
        {`╚══════╝ ╚═════╝ ╚══════╝ ╚═════╝  DEV`}
      </Text>
      <Text> </Text>

      {state === 'checking' && (
        <Text>
          <Text color="yellow"><Spinner type="dots" /></Text>
          {' Checking Claude connection...'}
        </Text>
      )}

      {state === 'authenticated' && (
        <Box flexDirection="column" alignItems="center">
          <Text color="green">{'✅'} {email} ({plan} plan)</Text>
          <Text> </Text>
          <Text dimColor>Press Enter to continue</Text>
        </Box>
      )}

      {state === 'not_authenticated' && (
        <Box flexDirection="column" alignItems="center">
          <Text color="yellow">{'⚠'}  Not connected to Claude</Text>
          <Text> </Text>
          <Text dimColor>[Enter] Log in (opens browser)   [q] Quit</Text>
        </Box>
      )}

      {state === 'error' && (
        <Box flexDirection="column" alignItems="center">
          <Text color="red">{'✗'} Error: {error}</Text>
          <Text> </Text>
          <Text dimColor>Is Claude Code installed? Run: npm install -g @anthropic-ai/claude-code</Text>
        </Box>
      )}
    </Box>
  );
};
```

**Step 2: Update App.tsx to render AuthScreen**

Replace the auth placeholder in `App.tsx`:

```tsx
// Add import at top
import { AuthScreen } from './screens/AuthScreen.js';

// Replace the auth case in the return
{screen === 'auth' && (
  <AuthScreen onComplete={() => setScreen('projects')} />
)}
```

**Step 3: Build and test**

```bash
npm run build
node dist/cli.js
```
Expected: Shows SOLODEV logo, checks auth, shows email/plan, auto-advances.

**Step 4: Commit**

```bash
git add source/tui/screens/AuthScreen.tsx source/tui/App.tsx
git commit -m "feat: add auth screen with Claude connection check"
```

---

### Task 9: TUI — Project Select Screen

**Files:**
- Create: `solodev/source/tui/screens/ProjectScreen.tsx`
- Modify: `solodev/source/tui/App.tsx` — import and render ProjectScreen

**Step 1: Create ProjectScreen**

```tsx
// source/tui/screens/ProjectScreen.tsx
import React, { useState } from 'react';
import { Box, Text, useInput } from 'ink';
import SelectInput from 'ink-select-input';
import TextInput from 'ink-text-input';
import { listProjects, addProject, removeProject, type Project } from '../../core/projects.js';
import { countCompleted, scanPipeline } from '../../core/pipeline.js';

interface Props {
  onSelect: (projectId: string, projectPath: string) => void;
}

type Mode = 'list' | 'new_name' | 'new_path' | 'open_path';

export const ProjectScreen = ({ onSelect }: Props) => {
  const [mode, setMode] = useState<Mode>('list');
  const [newName, setNewName] = useState('');
  const [newPath, setNewPath] = useState('');
  const projects = listProjects();

  useInput((input) => {
    if (mode === 'list') {
      if (input === 'n') setMode('new_name');
      if (input === 'o') setMode('open_path');
    }
  }, { isActive: mode === 'list' });

  const items = projects.map(p => {
    const steps = scanPipeline(p.path);
    const completed = countCompleted(steps);
    const label = completed > 0 ? `${p.name}  ${completed} steps` : `${p.name}  new`;
    return { label, value: p.id, key: p.id };
  });

  const handleSelect = (item: { value: string }) => {
    const project = projects.find(p => p.id === item.value);
    if (project) {
      onSelect(project.id, project.path);
    }
  };

  const handleNewName = (name: string) => {
    setNewName(name);
    setMode('new_path');
  };

  const handleNewPath = (projectPath: string) => {
    const resolved = projectPath.startsWith('~')
      ? projectPath.replace('~', process.env['HOME'] || '')
      : projectPath;
    const project = addProject(newName, resolved);
    onSelect(project.id, project.path);
  };

  const handleOpenPath = (projectPath: string) => {
    const resolved = projectPath.startsWith('~')
      ? projectPath.replace('~', process.env['HOME'] || '')
      : projectPath;
    const project = addProject(
      resolved.split('/').pop() || 'project',
      resolved
    );
    onSelect(project.id, project.path);
  };

  return (
    <Box flexDirection="column" padding={1}>
      <Box marginBottom={1}>
        <Text bold color="cyan">SOLODEV</Text>
        <Text>                              v0.1.0</Text>
      </Box>

      {mode === 'list' && (
        <>
          {projects.length > 0 ? (
            <>
              <Text bold>Your Projects:</Text>
              <Text> </Text>
              <SelectInput items={items} onSelect={handleSelect} />
            </>
          ) : (
            <Text dimColor>No projects yet. Create one to get started.</Text>
          )}
          <Text> </Text>
          <Text dimColor>[n] New project   [o] Open existing directory   [q] Quit</Text>
        </>
      )}

      {mode === 'new_name' && (
        <Box flexDirection="column">
          <Text bold>New Project</Text>
          <Text> </Text>
          <Box>
            <Text color="cyan">Project name: </Text>
            <TextInput value={newName} onChange={setNewName} onSubmit={handleNewName} placeholder="my-saas-app" />
          </Box>
        </Box>
      )}

      {mode === 'new_path' && (
        <Box flexDirection="column">
          <Text bold>New Project: {newName}</Text>
          <Text> </Text>
          <Box>
            <Text color="cyan">Directory: </Text>
            <TextInput value={newPath} onChange={setNewPath} onSubmit={handleNewPath} placeholder="~/projects/my-app" />
          </Box>
        </Box>
      )}

      {mode === 'open_path' && (
        <Box flexDirection="column">
          <Text bold>Open Existing Project</Text>
          <Text> </Text>
          <Box>
            <Text color="cyan">Path: </Text>
            <TextInput value={newPath} onChange={setNewPath} onSubmit={handleOpenPath} placeholder="~/existing-project" />
          </Box>
        </Box>
      )}
    </Box>
  );
};
```

**Step 2: Update App.tsx**

```tsx
// Add import
import { ProjectScreen } from './screens/ProjectScreen.js';

// Replace projects case
{screen === 'projects' && (
  <ProjectScreen onSelect={(id, path) => {
    setAppState({ projectId: id, projectPath: path });
    setScreen('dashboard');
  }} />
)}
```

**Step 3: Build and test**

```bash
npm run build && node dist/cli.js
```
Expected: After auth, shows project list or empty state. Can create new project.

**Step 4: Commit**

```bash
git add source/tui/screens/ProjectScreen.tsx source/tui/App.tsx
git commit -m "feat: add project select screen with new/open project flows"
```

---

### Task 10: TUI — Dashboard Screen

**Files:**
- Create: `solodev/source/tui/screens/DashboardScreen.tsx`
- Create: `solodev/source/tui/components/PipelineList.tsx`
- Modify: `solodev/source/tui/App.tsx` — import and render DashboardScreen

**Step 1: Create PipelineList component**

```tsx
// source/tui/components/PipelineList.tsx
import React from 'react';
import { Box, Text } from 'ink';
import type { PipelineStep } from '../../core/pipeline.js';

interface Props {
  steps: PipelineStep[];
  selectedIndex: number;
}

function statusIcon(status: PipelineStep['status']): string {
  switch (status) {
    case 'complete': return '✅';
    case 'in_progress': return '🔨';
    case 'not_started': return '⬚ ';
  }
}

export const PipelineList = ({ steps, selectedIndex }: Props) => {
  return (
    <Box flexDirection="column">
      {steps.map((step, i) => {
        const isSelected = i === selectedIndex;
        const icon = statusIcon(step.status);
        const indexStr = step.index.toString().padStart(2, ' ');
        const nameStr = step.name.padEnd(22);
        const cmdStr = step.command.padEnd(12);
        const dirStr = step.status !== 'not_started' ? step.dotDir : '';

        return (
          <Box key={step.command}>
            <Text color={isSelected ? 'cyan' : undefined} bold={isSelected}>
              {isSelected ? '>' : ' '} {icon} {indexStr}. {nameStr} {cmdStr} {dirStr}
            </Text>
          </Box>
        );
      })}
    </Box>
  );
};
```

**Step 2: Create DashboardScreen**

```tsx
// source/tui/screens/DashboardScreen.tsx
import React, { useState, useMemo } from 'react';
import { Box, Text, useInput } from 'ink';
import { scanPipeline, suggestNext, type PipelineStep } from '../../core/pipeline.js';
import { getProject } from '../../core/projects.js';
import { PipelineList } from '../components/PipelineList.js';

interface Props {
  projectId: string;
  projectPath: string;
  onRun: (step: PipelineStep) => void;
  onBack: () => void;
}

export const DashboardScreen = ({ projectId, projectPath, onRun, onBack }: Props) => {
  const project = getProject(projectId);
  const steps = useMemo(() => scanPipeline(projectPath), [projectPath]);
  const suggested = useMemo(() => suggestNext(steps), [steps]);
  const suggestedIndex = suggested ? steps.findIndex(s => s.command === suggested.command) : 0;
  const [selectedIndex, setSelectedIndex] = useState(suggestedIndex);

  useInput((input, key) => {
    if (key.upArrow) {
      setSelectedIndex(i => Math.max(0, i - 1));
    }
    if (key.downArrow) {
      setSelectedIndex(i => Math.min(steps.length - 1, i + 1));
    }
    if (key.return) {
      onRun(steps[selectedIndex]);
    }
    if (input === 'v' && steps[selectedIndex].status !== 'not_started') {
      // Open output — TODO: implement viewer
    }
    if (input === 'p') {
      onBack();
    }
  });

  const selectedStep = steps[selectedIndex];
  const suggestion = suggested
    ? suggested.status === 'in_progress'
      ? `Continue ${suggested.name.toLowerCase()}?`
      : `Run ${suggested.name.toLowerCase()}?`
    : 'All steps complete!';

  return (
    <Box flexDirection="column" padding={1}>
      <Box marginBottom={1} justifyContent="space-between">
        <Text bold color="cyan">{project?.name || 'Project'}</Text>
        <Text dimColor>{projectPath}</Text>
      </Box>

      <Text bold>Pipeline:</Text>
      <Text> </Text>

      <PipelineList steps={steps} selectedIndex={selectedIndex} />

      <Text> </Text>
      <Text color="yellow">{'→'} {suggestion}</Text>
      <Text> </Text>
      <Text dimColor>[Enter] Run  [↑↓] Select  [v] View output  [p] Projects  [q] Quit</Text>
    </Box>
  );
};
```

**Step 3: Update App.tsx**

```tsx
// Add imports
import { DashboardScreen } from './screens/DashboardScreen.js';
import type { PipelineStep } from '../core/pipeline.js';

// Add run state
const [runStep, setRunStep] = useState<PipelineStep | null>(null);

// Replace dashboard case
{screen === 'dashboard' && appState.projectId && appState.projectPath && (
  <DashboardScreen
    projectId={appState.projectId}
    projectPath={appState.projectPath}
    onRun={(step) => {
      setRunStep(step);
      setScreen('run');
    }}
    onBack={() => setScreen('projects')}
  />
)}
```

**Step 4: Build and test**

```bash
npm run build && node dist/cli.js
```
Expected: Dashboard shows pipeline with status icons. Arrow keys navigate. Enter would trigger run.

**Step 5: Commit**

```bash
git add source/tui/screens/DashboardScreen.tsx source/tui/components/PipelineList.tsx source/tui/App.tsx
git commit -m "feat: add dashboard screen with pipeline visualization"
```

---

### Task 11: TUI — Run Screen with Collapsible Stream

**Files:**
- Create: `solodev/source/tui/screens/RunScreen.tsx`
- Create: `solodev/source/tui/components/LiveStream.tsx`
- Modify: `solodev/source/tui/App.tsx` — import and render RunScreen

**Step 1: Create LiveStream component**

```tsx
// source/tui/components/LiveStream.tsx
import React from 'react';
import { Box, Text } from 'ink';

interface Props {
  lines: string[];
  maxLines: number;
  expanded: boolean;
}

export const LiveStream = ({ lines, maxLines, expanded }: Props) => {
  if (!expanded) return null;

  const visible = lines.slice(-maxLines);

  return (
    <Box flexDirection="column" borderStyle="single" borderColor="gray" paddingX={1}>
      <Text bold dimColor>Live Output</Text>
      {visible.map((line, i) => (
        <Text key={i} dimColor>{line}</Text>
      ))}
      {lines.length === 0 && <Text dimColor>Waiting for output...</Text>}
    </Box>
  );
};
```

**Step 2: Create RunScreen**

```tsx
// source/tui/screens/RunScreen.tsx
import React, { useState, useEffect, useCallback } from 'react';
import { Box, Text, useInput } from 'ink';
import Spinner from 'ink-spinner';
import { runClaude, type ClaudeRunOptions } from '../../core/claude.js';
import { getProject, touchProject } from '../../core/projects.js';
import { getPluginDir } from '../../core/plugins.js';
import { LiveStream } from '../components/LiveStream.js';
import type { PipelineStep } from '../../core/pipeline.js';

interface Props {
  projectId: string;
  step: PipelineStep;
  onComplete: () => void;
  onCancel: () => void;
}

type Phase = 'running' | 'complete' | 'error';

export const RunScreen = ({ projectId, step, onComplete, onCancel }: Props) => {
  const [phase, setPhase] = useState<Phase>('running');
  const [lines, setLines] = useState<string[]>([]);
  const [expanded, setExpanded] = useState(false);
  const [elapsed, setElapsed] = useState(0);
  const [exitCode, setExitCode] = useState<number | null>(null);
  const [killFn, setKillFn] = useState<(() => void) | null>(null);

  useEffect(() => {
    const project = getProject(projectId);
    if (!project) {
      setPhase('error');
      setLines(['Project not found']);
      return;
    }

    const pluginDir = getPluginDir();
    const isNew = !project.sessionId;

    const options: ClaudeRunOptions = {
      prompt: step.command,
      sessionId: project.sessionId,
      isNewSession: isNew,
      pluginDir,
      permissionMode: project.permissionMode,
    };

    const { events, kill } = runClaude(options);
    setKillFn(() => kill);

    events.on('message', (msg: { type: string; content?: string }) => {
      if (msg.content) {
        setLines(prev => [...prev, msg.content!]);
      }
    });

    events.on('done', (code: number) => {
      setExitCode(code);
      setPhase(code === 0 ? 'complete' : 'error');
      touchProject(projectId);
    });

    return () => {
      kill();
    };
  }, [projectId, step.command]);

  // Elapsed timer
  useEffect(() => {
    if (phase !== 'running') return;
    const interval = setInterval(() => setElapsed(e => e + 1), 1000);
    return () => clearInterval(interval);
  }, [phase]);

  useInput((input, key) => {
    if (input === ' ') setExpanded(e => !e);
    if (input === 'c' && phase === 'running') {
      killFn?.();
      setPhase('error');
      setLines(prev => [...prev, 'Cancelled by user']);
    }
    if ((key.return || input === 'q') && phase !== 'running') {
      onComplete();
    }
  });

  const formatTime = (s: number) => {
    const m = Math.floor(s / 60);
    const sec = s % 60;
    return m > 0 ? `${m}m ${sec}s` : `${sec}s`;
  };

  return (
    <Box flexDirection="column" padding={1}>
      <Box justifyContent="space-between" marginBottom={1}>
        <Text bold>
          Running: <Text color="cyan">{step.command}</Text>
        </Text>
        <Text>
          {phase === 'running' && (
            <Text color="yellow"><Spinner type="dots" /> {formatTime(elapsed)}</Text>
          )}
          {phase === 'complete' && (
            <Text color="green">{'✅'} Done in {formatTime(elapsed)}</Text>
          )}
          {phase === 'error' && (
            <Text color="red">{'✗'} Failed after {formatTime(elapsed)}</Text>
          )}
        </Text>
      </Box>

      <LiveStream lines={lines} maxLines={expanded ? 20 : 0} expanded={expanded} />

      <Text> </Text>
      {phase === 'running' && (
        <Text dimColor>[Space] {expanded ? 'Collapse' : 'Expand'} output   [c] Cancel</Text>
      )}
      {phase !== 'running' && (
        <Text dimColor>[Enter] Back to dashboard   [Space] {expanded ? 'Collapse' : 'Expand'} output</Text>
      )}
    </Box>
  );
};
```

**Step 3: Update App.tsx**

```tsx
// Add import
import { RunScreen } from './screens/RunScreen.js';

// Replace run case
{screen === 'run' && appState.projectId && runStep && (
  <RunScreen
    projectId={appState.projectId}
    step={runStep}
    onComplete={() => {
      setRunStep(null);
      setScreen('dashboard');
    }}
    onCancel={() => {
      setRunStep(null);
      setScreen('dashboard');
    }}
  />
)}
```

**Step 4: Build and test**

```bash
npm run build && node dist/cli.js
```
Expected: Selecting a step from dashboard launches run screen with spinner. Space toggles output.

**Step 5: Commit**

```bash
git add source/tui/screens/RunScreen.tsx source/tui/components/LiveStream.tsx source/tui/App.tsx
git commit -m "feat: add run screen with collapsible live stream"
```

---

### Task 12: Direct Command Mode

**Files:**
- Create: `solodev/source/commands/run.ts`
- Create: `solodev/source/commands/status.ts`
- Create: `solodev/source/commands/reset.ts`
- Modify: `solodev/source/cli.tsx` — wire up direct commands

**Step 1: Create the run command**

```typescript
// source/commands/run.ts
import { getProjectByPath, addProject, touchProject } from '../core/projects.js';
import { getPluginDir } from '../core/plugins.js';
import { runClaude } from '../core/claude.js';

export async function runDirect(command: string, args: string[], projectPath: string): Promise<void> {
  let project = getProjectByPath(projectPath);
  if (!project) {
    const name = projectPath.split('/').pop() || 'project';
    project = addProject(name, projectPath);
  }

  const prompt = `/${command} ${args.join(' ')}`.trim();
  const pluginDir = getPluginDir();

  const { events, kill } = runClaude({
    prompt,
    sessionId: project.sessionId,
    isNewSession: false,
    pluginDir,
    permissionMode: project.permissionMode,
  });

  return new Promise((resolve, reject) => {
    events.on('message', (msg: { type: string; content?: string }) => {
      if (msg.content) {
        process.stdout.write(msg.content + '\n');
      }
    });

    events.on('done', (code: number) => {
      touchProject(project!.id);
      if (code === 0) resolve();
      else reject(new Error(`Claude exited with code ${code}`));
    });

    process.on('SIGINT', () => {
      kill();
      process.exit(130);
    });
  });
}
```

**Step 2: Create the status command**

```typescript
// source/commands/status.ts
import { scanPipeline, suggestNext, countCompleted } from '../core/pipeline.js';
import { getProjectByPath } from '../core/projects.js';

export function showStatus(projectPath: string): void {
  const project = getProjectByPath(projectPath);
  const steps = scanPipeline(projectPath);
  const completed = countCompleted(steps);
  const suggested = suggestNext(steps);

  console.log(`\nProject: ${project?.name || projectPath}`);
  console.log(`Progress: ${completed}/${steps.length} steps\n`);

  for (const step of steps) {
    const icon = step.status === 'complete' ? '✅' :
                 step.status === 'in_progress' ? '🔨' : '  ';
    const idx = step.index.toString().padStart(2);
    console.log(`  ${icon} ${idx}. ${step.name.padEnd(22)} ${step.command}`);
  }

  if (suggested) {
    console.log(`\nNext: ${suggested.command} (${suggested.name})`);
  } else {
    console.log('\nAll steps complete!');
  }
  console.log('');
}
```

**Step 3: Create the reset command**

```typescript
// source/commands/reset.ts
import fs from 'fs';
import path from 'path';
import { v4 as uuidv4 } from 'uuid';
import { getProjectByPath, updateProjectSession } from '../core/projects.js';
import { scanPipeline } from '../core/pipeline.js';

export function resetProject(projectPath: string, hard: boolean): void {
  const project = getProjectByPath(projectPath);
  if (!project) {
    console.log('No solodev project found for this directory.');
    return;
  }

  // Reset session
  const newSessionId = uuidv4();
  updateProjectSession(project.id, newSessionId);
  console.log(`Session reset for ${project.name}`);

  if (hard) {
    const steps = scanPipeline(projectPath);
    for (const step of steps) {
      const dirPath = path.join(projectPath, step.dotDir);
      if (fs.existsSync(dirPath)) {
        fs.rmSync(dirPath, { recursive: true, force: true });
        console.log(`  Removed ${step.dotDir}/`);
      }
    }
    console.log('All plugin outputs removed.');
  }
}
```

**Step 4: Wire up in cli.tsx**

Update the action handler in `cli.tsx`:

```tsx
.action(async (command?: string, args?: string[], options?: { project?: string }) => {
  const projectPath = options?.project || process.cwd();

  if (command === 'status') {
    const { showStatus } = await import('./commands/status.js');
    showStatus(projectPath);
    return;
  }

  if (command === 'reset') {
    const { resetProject } = await import('./commands/reset.js');
    resetProject(projectPath, args?.includes('--hard') || false);
    return;
  }

  if (command) {
    const { runDirect } = await import('./commands/run.js');
    try {
      await runDirect(command, args || [], projectPath);
    } catch (err: any) {
      console.error(err.message);
      process.exit(1);
    }
    return;
  }

  // Interactive TUI mode
  const { waitUntilExit } = render(<App />);
  await waitUntilExit();
});
```

**Step 5: Build and test**

```bash
npm run build
node dist/cli.js status
node dist/cli.js reset
```

**Step 6: Commit**

```bash
git add source/commands/run.ts source/commands/status.ts source/commands/reset.ts source/cli.tsx
git commit -m "feat: add direct command mode — status, reset, and plugin execution"
```

---

### Task 13: Plugin Bundling and Build Pipeline

**Files:**
- Modify: `solodev/package.json` — add bundle-plugins to build
- Modify: `solodev/scripts/bundle-plugins.ts` — refine for npm publish

**Step 1: Update build script in package.json**

```json
"scripts": {
  "build": "npm run bundle-plugins && tsc && chmod +x dist/cli.js",
  "bundle-plugins": "node --loader ts-node/esm scripts/bundle-plugins.ts",
  "dev": "tsc --watch",
  "start": "node dist/cli.js",
  "test": "node --experimental-vm-modules node_modules/.bin/jest",
  "prepublishOnly": "npm run build"
}
```

Also add `ts-node` to devDependencies:
```json
"ts-node": "^10.9.0"
```

**Step 2: Run full build**

```bash
cd /Users/steven/solodev
npm install
npm run build
```
Expected: Plugins copied to `plugins/`, TypeScript compiled to `dist/`, cli.js is executable.

**Step 3: Test global install locally**

```bash
npm link
solodev --version
solodev status
solodev
```
Expected: All three modes work.

**Step 4: Commit**

```bash
git add package.json scripts/bundle-plugins.ts
git commit -m "feat: integrate plugin bundling into build pipeline"
```

---

### Task 14: Jest Configuration

**Files:**
- Create: `solodev/jest.config.js`

**Step 1: Create jest config for ESM + TypeScript**

```javascript
// jest.config.js
export default {
  preset: 'ts-jest/presets/default-esm',
  extensionsToTreatAsEsm: ['.ts', '.tsx'],
  moduleNameMapper: {
    '^(\\.{1,2}/.*)\\.js$': '$1',
  },
  transform: {
    '^.+\\.tsx?$': [
      'ts-jest',
      {
        useESM: true,
        tsconfig: 'tsconfig.json',
      },
    ],
  },
  testMatch: ['**/__tests__/**/*.test.ts'],
};
```

**Step 2: Run all tests**

```bash
npm test
```
Expected: All tests from Tasks 2-5 pass.

**Step 3: Commit**

```bash
git add jest.config.js
git commit -m "feat: add Jest config for ESM + TypeScript"
```

---

### Task 15: Final Integration Test and README

**Files:**
- Create: `solodev/README.md`

**Step 1: Write README**

```markdown
# solodev

Interactive CLI for the Solo Dev Toolbox — 18 plugins, one pipeline, persistent context.

## Install

```bash
npm install -g solodev
```

Requires [Claude Code](https://claude.ai/code) installed and authenticated.

## Usage

```bash
# Interactive TUI
solodev

# Direct commands
solodev spark "AI-powered code review tool"
solodev status
solodev coverage --generate-skeletons
solodev reset
```

## How It Works

solodev wraps Claude Code with a persistent session per project. Every plugin run resumes the same conversation, so context accumulates across your entire pipeline.

See the [Solo Dev Toolbox](https://github.com/stevenkozeniesky02/claude-plugins) for the full plugin suite.
```

**Step 2: Full integration test**

```bash
npm run build
npm link
solodev --version
solodev --help
solodev status
solodev
```
Test the full flow: auth → create project → dashboard → run a step → back to dashboard.

**Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add README"
```

**Step 4: Push**

```bash
git remote add origin <your-repo-url>
git push -u origin main
```

---

## Task Summary

| Task | Description | Dependencies |
|------|-------------|-------------|
| 1 | Project scaffold (package.json, tsconfig, deps) | None |
| 2 | Core: config module | Task 1 |
| 3 | Core: projects module | Task 2 |
| 4 | Core: pipeline tracker | Task 1 |
| 5 | Core: Claude subprocess wrapper | Task 1 |
| 6 | Core: plugin bundler + resolver | Task 1 |
| 7 | TUI: entry point + app shell | Tasks 1-6 |
| 8 | TUI: auth screen | Task 7 |
| 9 | TUI: project select screen | Tasks 7, 3 |
| 10 | TUI: dashboard screen | Tasks 7, 3, 4 |
| 11 | TUI: run screen with live stream | Tasks 7, 5, 6 |
| 12 | Direct command mode | Tasks 3, 4, 5, 6 |
| 13 | Plugin bundling build pipeline | Task 6 |
| 14 | Jest configuration | Task 1 |
| 15 | Integration test + README | All |
