# Solo Dev Toolbox -- Plugin Suite Design

**Date:** 2026-03-03
**Status:** Approved
**Scope:** 16 new plugins + 2 existing = 18 total

## Vision

The ultimate plugin suite for solo developers and small startups. Takes someone from "I have an idea" to "I have a deployed product" through an orchestrated pipeline of specialized plugins -- each working standalone but also feeding context forward to the next.

## Architecture Principles

All plugins follow the same proven pattern established by codebase-ideation and codebase-roadmap:

- **Phase-driven workflows:** Phase 0 (scan) -> Phase 1 (parallel specialists) -> Phase 2 (synthesis)
- **State persistence:** Each plugin writes to `.{plugin}/state.json` for resumable sessions
- **Output layering:** `00-*.md` (foundational scan) -> `01-*.md` (specialist outputs) -> final report
- **Context chaining:** Each plugin reads upstream plugin outputs if they exist (e.g., stack-pick reads `.spark/business-case.md`)
- **Standalone operation:** Every plugin works independently -- the pipeline is a recommendation, not a requirement
- **User approval gates:** No destructive action without explicit user consent

## Plugin Categories

### Category 1: Core Pipeline (5 plugins)

Sequential chain that takes a user from idea to running code. Each produces output the next consumes. Users can enter at any point.

```
idea-spark -> stack-pick -> codebase-roadmap -> scaffold-kit -> build-engine
```

---

### 1. idea-spark

- **Command:** `/spark`
- **Purpose:** Validate and refine a business idea through conversational research
- **Input:** User describes their idea in plain English
- **Output directory:** `.spark/`
- **Output:** `business-case.md` (problem, solution, market, competitors, revenue model, risks, go/no-go)
- **Triggers next:** Suggests `/stack` when business case is approved

**Process:**
- Conversational back-and-forth to understand the idea
- Market research agents investigate competitors, TAM/SAM/SOM, revenue models
- Feasibility analysis considering solo dev constraints
- Iterates with user until consensus on the business case

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `market-researcher` | TAM/SAM/SOM sizing, market trends, timing analysis | Read, Glob, Grep, WebSearch, WebFetch |
| `competitor-scout` | Finds and analyzes competing products, pricing, positioning | Read, Glob, Grep, WebSearch, WebFetch |
| `revenue-modeler` | Proposes revenue models, unit economics, pricing structures | Read, Glob, Grep |
| `feasibility-analyst` | Technical feasibility, solo dev constraints, timeline reality check | Read, Glob, Grep, WebSearch |
| `spark-synthesizer` | Combines all research into coherent business case | Read, Glob, Grep |

**Skill:** `spark-patterns` -- frameworks for idea validation (Lean Canvas, Jobs-to-be-Done, competitive moat analysis)

---

### 2. stack-pick

- **Command:** `/stack`
- **Purpose:** Research and recommend the optimal tech stack
- **Input:** Reads `.spark/business-case.md` if exists; otherwise asks user about their project
- **Output directory:** `.stack/`
- **Output:** `tech-decisions.md` (chosen stack with rationale, rejected alternatives, cost estimates, learning curve)
- **Triggers next:** Suggests `/generate` (codebase-roadmap)

**Process:**
- Reads business case to understand requirements (scale, budget, domain)
- Web-researches current frameworks, databases, hosting options
- Evaluates against solo dev constraints (simplicity, cost, ecosystem, hiring pool)
- Presents 2-3 complete stack options with trade-offs
- User picks or asks for more research

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `backend-researcher` | Evaluates backend frameworks, languages, ORMs | Read, Glob, Grep, WebSearch, WebFetch |
| `frontend-researcher` | Evaluates frontend frameworks, UI libraries, build tools | Read, Glob, Grep, WebSearch, WebFetch |
| `infra-researcher` | Evaluates hosting, databases, queues, CI/CD options | Read, Glob, Grep, WebSearch, WebFetch |
| `dx-analyst` | Developer experience assessment -- learning curve, docs quality, community, solo dev fit | Read, Glob, Grep, WebSearch, WebFetch |
| `stack-synthesizer` | Combines research into ranked stack recommendations | Read, Glob, Grep |

**Skill:** `stack-patterns` -- decision frameworks for tech stack selection (build vs buy, monolith vs micro, managed vs self-hosted)

---

### 3. codebase-roadmap (EXISTING -- enhanced)

- **Command:** `/generate`
- **Enhancement:** Reads `.spark/business-case.md` and `.stack/tech-decisions.md` if they exist to enrich roadmap with business context and stack-specific milestones
- **Output:** `.roadmap/roadmap.md` (unchanged format)

---

### 4. scaffold-kit

- **Command:** `/scaffold`
- **Purpose:** Bootstrap the project skeleton from all upstream decisions
- **Input:** Reads `.stack/tech-decisions.md` and `.roadmap/roadmap.md`
- **Output directory:** Actual project files in the working directory
- **Output:** Project structure, package configs, CI/CD, Dockerfile, docker-compose, README, .env.example, pre-commit hooks, test scaffolding
- **Triggers next:** Suggests `/build`

**Process:**
- Reads tech stack decisions and roadmap
- Generates project structure following stack conventions
- Creates CI/CD pipeline matching the chosen stack
- Generates Dockerfile and docker-compose if applicable
- Creates README with setup instructions
- Sets up linting, formatting, pre-commit hooks
- Creates test directory structure with example tests
- User reviews and approves before files are written

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `project-structurer` | Designs directory layout, module organization, entry points | Read, Glob, Grep, Bash |
| `cicd-generator` | Creates CI/CD pipeline, Dockerfile, docker-compose | Read, Glob, Grep, Bash |
| `config-generator` | Creates package configs, linting, formatting, pre-commit | Read, Glob, Grep, Bash |
| `docs-bootstrapper` | Creates README, CONTRIBUTING, .env.example, architecture doc | Read, Glob, Grep |
| `scaffold-assembler` | Orchestrates file generation, resolves conflicts | Read, Glob, Grep |

**Skill:** `scaffold-patterns` -- project structure conventions per stack, CI/CD templates, config best practices

---

### 5. build-engine

- **Command:** `/build`
- **Purpose:** Implements the roadmap feature-by-feature using TDD and multi-agent coding
- **Input:** Reads `.roadmap/roadmap.md`, works through milestones sequentially
- **Output directory:** `.build/`
- **Output:** Working code committed after each milestone; progress tracked in `.build/progress.md`

**Process:**
- Reads roadmap and presents milestones to user
- For each milestone: plan -> write tests -> implement -> verify -> commit
- User approves each milestone before coding starts
- Parallel agents handle independent sub-tasks within a milestone
- Commits at clean points with descriptive messages
- Never starts next milestone without user go-ahead

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `milestone-planner` | Breaks milestone into implementation tasks, identifies files to create/modify | Read, Glob, Grep, Bash |
| `test-writer` | Writes tests first (TDD), defines acceptance criteria | Read, Glob, Grep, Bash, Write, Edit |
| `implementer` | Writes the actual feature code to pass the tests | Read, Glob, Grep, Bash, Write, Edit |
| `code-reviewer` | Reviews implementation for bugs, security, quality | Read, Glob, Grep, Bash |
| `build-orchestrator` | Coordinates the build loop, tracks progress, manages commits | Read, Glob, Grep |

**Skill:** `build-patterns` -- TDD workflow, commit conventions, milestone decomposition strategies

**Key guardrails:**
- Never starts a milestone without user approval
- Always runs tests before committing
- Commits after each completed milestone
- Tracks progress in `.build/progress.md`
- Can resume mid-milestone if interrupted

---

### Category 2: Code & Architecture Utilities (4 plugins)

Work on any existing codebase. No pipeline dependency.

---

### 6. codebase-ideation (EXISTING)

- **Command:** `/discover`
- **No changes**

---

### 7. codebase-autopsy

- **Command:** `/autopsy`
- **Purpose:** Deep tech debt analysis with risk-scored "time bombs"
- **Output directory:** `.autopsy/`
- **Output:** `report.md` (ranked tech debt inventory with time-bomb scores, fix order, effort estimates)

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `debt-detector` | Finds shortcuts, workarounds, hardcoded values, TODO/FIXME/HACK comments | Read, Glob, Grep, Bash |
| `complexity-analyst` | Measures cyclomatic complexity, coupling, cohesion, function length | Read, Glob, Grep, Bash |
| `pattern-archaeologist` | Finds outdated patterns, deprecated API usage, legacy conventions | Read, Glob, Grep, Bash |
| `risk-scorer` | Assigns "detonation risk" scores based on blast radius and probability | Read, Glob, Grep |
| `autopsy-synthesizer` | Combines findings into prioritized report | Read, Glob, Grep |

**Skill:** `autopsy-patterns` -- tech debt classification, risk scoring frameworks, refactoring prioritization

---

### 8. launch-check

- **Command:** `/preflight`
- **Purpose:** Production readiness audit with GO/NO-GO verdict
- **Output directory:** `.preflight/`
- **Output:** `report.md` (scorecard with verdict, blocking issues, warnings, prioritized fix list)

**Checks 50+ criteria across:**
- Security (auth, secrets, headers, CORS, input validation)
- Performance (N+1 queries, missing indexes, caching, bundle size)
- Observability (logging, health checks, error tracking, metrics)
- Reliability (error handling, retries, circuit breakers, graceful degradation)
- Configuration (env vars, secrets management, feature flags)
- Documentation (README, API docs, setup guide, architecture)
- Deployment (CI/CD, Dockerfile, rollback capability, health probes)
- Compliance (privacy policy, data handling, cookie consent)

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `security-checker` | Auth, secrets, headers, CORS, input validation, dependency CVEs | Read, Glob, Grep, Bash |
| `ops-checker` | CI/CD, Docker, health probes, deployment strategy, env config | Read, Glob, Grep, Bash |
| `reliability-checker` | Error handling, retries, circuit breakers, graceful degradation, logging | Read, Glob, Grep, Bash |
| `docs-checker` | README, API docs, setup guide, architecture docs, CHANGELOG | Read, Glob, Grep, Bash |
| `preflight-synthesizer` | Aggregates into scorecard with GO/NO-GO verdict | Read, Glob, Grep |

**Skill:** `preflight-patterns` -- production readiness checklists, severity classification, industry benchmarks

---

### 9. test-gap

- **Command:** `/coverage`
- **Purpose:** Find untested critical paths and generate test strategies
- **Output directory:** `.testgap/`
- **Output:** `report.md` (coverage map, risk-ranked untested paths, test strategies) + optional skeleton test files

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `coverage-mapper` | Maps existing test coverage against source modules | Read, Glob, Grep, Bash |
| `risk-path-finder` | Identifies high-risk untested paths (auth, payments, data mutations, error handlers) | Read, Glob, Grep, Bash |
| `test-strategist` | Recommends test types per path (unit, integration, e2e, property-based) | Read, Glob, Grep |
| `skeleton-writer` | Generates skeleton test files with test names and setup | Read, Glob, Grep |
| `coverage-synthesizer` | Produces prioritized coverage improvement plan | Read, Glob, Grep |

**Skill:** `testgap-patterns` -- test prioritization frameworks, coverage vs confidence, testing pyramid strategies

---

### 10. dep-doctor

- **Command:** `/checkup`
- **Purpose:** Dependency health audit with actionable recommendations
- **Output directory:** `.depdoctor/`
- **Output:** `report.md` (dependency health scorecard, upgrade/replace/remove recommendations)

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `vuln-scanner` | Checks for known CVEs in all dependencies | Read, Glob, Grep, Bash |
| `freshness-checker` | Identifies outdated deps, checks latest versions, breaking changes | Read, Glob, Grep, Bash, WebSearch |
| `license-auditor` | Checks license compatibility, flags copyleft in proprietary projects | Read, Glob, Grep, Bash |
| `bloat-detector` | Finds unused deps, transitive bloat, duplicate packages | Read, Glob, Grep, Bash |
| `dep-synthesizer` | Combines into health scorecard with recommendations | Read, Glob, Grep |

**Skill:** `dep-patterns` -- dependency management best practices, version pinning strategies, license compatibility matrix

---

### Category 3: DevOps & Operations Utilities (3 plugins)

---

### 11. cost-forecast

- **Command:** `/estimate`
- **Purpose:** Estimate infrastructure and API costs at different user scales
- **Output directory:** `.costforecast/`
- **Output:** `report.md` (cost breakdown per tier, cost cliffs, optimization recommendations, burn rate)

**Agents (4):**

| Agent | Role | Tools |
|-------|------|-------|
| `infra-cost-analyst` | Analyzes hosting, database, queue, storage costs | Read, Glob, Grep, Bash, WebSearch, WebFetch |
| `api-cost-analyst` | Analyzes third-party API costs, LLM usage, external service fees | Read, Glob, Grep, Bash, WebSearch, WebFetch |
| `scale-modeler` | Models costs at 100, 1K, 10K, 100K user tiers; identifies cost cliffs | Read, Glob, Grep |
| `cost-synthesizer` | Produces cost report with optimization recommendations | Read, Glob, Grep |

**Skill:** `cost-patterns` -- cloud pricing models, cost optimization strategies, free tier limits reference

---

### 12. incident-planner

- **Command:** `/runbook`
- **Purpose:** Generate runbooks and incident response playbooks from architecture
- **Output directory:** `.runbooks/`
- **Output:** Individual runbooks per failure scenario + master incident response plan

**Agents (4):**

| Agent | Role | Tools |
|-------|------|-------|
| `architecture-mapper` | Maps all services, dependencies, communication paths, data flows | Read, Glob, Grep, Bash |
| `failure-mode-analyst` | Identifies failure modes per component (crash, timeout, data corruption, capacity) | Read, Glob, Grep, Bash |
| `runbook-writer` | Generates step-by-step runbooks for each failure scenario | Read, Glob, Grep |
| `runbook-synthesizer` | Creates master incident response plan and severity classification | Read, Glob, Grep |

**Skill:** `runbook-patterns` -- incident response frameworks, severity classification, postmortem templates

---

### 13. scale-sim

- **Command:** `/simulate`
- **Purpose:** Identify scaling bottlenecks before they hit production
- **Output directory:** `.scalesim/`
- **Output:** `report.md` (bottleneck map with scaling ceilings per component, fixes, priority order)

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `bottleneck-hunter` | Identifies single points of failure, synchronous bottlenecks, resource contention | Read, Glob, Grep, Bash |
| `database-profiler` | Analyzes schemas for N+1 queries, missing indexes, unbounded queries, connection limits | Read, Glob, Grep, Bash |
| `queue-analyst` | Analyzes message queue configs for throughput limits, backpressure, dead letters | Read, Glob, Grep, Bash |
| `capacity-modeler` | Models what breaks at 10x, 100x, 1000x with specific numbers | Read, Glob, Grep |
| `scale-synthesizer` | Produces scaling report with prioritized recommendations | Read, Glob, Grep |

**Skill:** `scale-patterns` -- scaling strategy frameworks, capacity planning heuristics, common bottleneck patterns

---

### Category 4: Business & Go-to-Market Utilities (5 plugins)

---

### 14. pitch-craft

- **Command:** `/pitch`
- **Purpose:** Generate investor-ready pitch materials
- **Input:** Reads `.spark/business-case.md` if exists; scans codebase for traction metrics
- **Output directory:** `.pitch/`
- **Output:** Pitch deck outline, one-pager, elevator pitch, investor FAQ

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `narrative-builder` | Crafts the story arc: problem -> solution -> traction -> ask | Read, Glob, Grep |
| `market-sizer` | TAM/SAM/SOM with bottom-up and top-down estimates | Read, Glob, Grep, WebSearch, WebFetch |
| `financial-modeler` | Revenue projections, unit economics, burn rate, runway | Read, Glob, Grep |
| `deck-writer` | Writes slide-by-slide content for pitch deck | Read, Glob, Grep |
| `pitch-synthesizer` | Produces all pitch formats (deck, one-pager, elevator pitch) | Read, Glob, Grep |

**Skill:** `pitch-patterns` -- pitch deck frameworks (Sequoia, YC), storytelling patterns, investor objection handling

---

### 15. price-model

- **Command:** `/pricing`
- **Purpose:** Recommend pricing strategy based on costs, competitors, and value
- **Input:** Reads `.costforecast/report.md` if exists; researches competitor pricing
- **Output directory:** `.pricing/`
- **Output:** `strategy.md` (recommended tiers, price points, feature gates, margin analysis)

**Agents (4):**

| Agent | Role | Tools |
|-------|------|-------|
| `competitor-pricing-analyst` | Researches competitor pricing, packaging, positioning | Read, Glob, Grep, WebSearch, WebFetch |
| `unit-economics-modeler` | Calculates COGS per user, contribution margin, break-even | Read, Glob, Grep |
| `value-metric-analyst` | Identifies the right value metric (seats, API calls, records, etc.) | Read, Glob, Grep |
| `pricing-synthesizer` | Produces tier recommendations with margin analysis | Read, Glob, Grep |

**Skill:** `pricing-patterns` -- pricing strategy frameworks, freemium vs usage-based, price anchoring, tier design

---

### 16. ship-page

- **Command:** `/landing`
- **Purpose:** Generate landing page copy and marketing content from your codebase
- **Input:** Scans codebase for features; reads business case and pricing if available
- **Output directory:** `.shippage/`
- **Output:** Copy document + optional HTML landing page

**Agents (4):**

| Agent | Role | Tools |
|-------|------|-------|
| `feature-extractor` | Scans codebase to identify all user-facing features | Read, Glob, Grep, Bash |
| `benefit-mapper` | Translates features into user benefits and value propositions | Read, Glob, Grep |
| `copywriter` | Writes hero copy, feature sections, FAQ, CTA, comparison tables | Read, Glob, Grep |
| `page-assembler` | Produces final copy doc and optional HTML | Read, Glob, Grep |

**Skill:** `shippage-patterns` -- landing page conversion patterns, copywriting frameworks (PAS, AIDA), above-the-fold best practices

---

### 17. legal-kit

- **Command:** `/comply`
- **Purpose:** Generate legal documents and compliance checklists based on actual app behavior
- **Input:** Scans codebase for data collection, auth, payments, cookies, third-party integrations
- **Output directory:** `.legalkit/`
- **Output:** Privacy policy, terms of service, cookie policy, compliance checklist, GDPR/CCPA gap analysis

**Agents (4):**

| Agent | Role | Tools |
|-------|------|-------|
| `data-flow-mapper` | Identifies all data collection, storage, sharing, and processing | Read, Glob, Grep, Bash |
| `regulation-matcher` | Determines which regulations apply based on data types and jurisdictions | Read, Glob, Grep, WebSearch |
| `doc-drafter` | Drafts privacy policy, ToS, cookie policy from templates | Read, Glob, Grep |
| `compliance-synthesizer` | Produces compliance checklist and gap analysis | Read, Glob, Grep |

**Skill:** `legal-patterns` -- GDPR/CCPA requirements, privacy policy templates, compliance checklist frameworks

**Disclaimer:** Output is a starting point, not legal advice. Users should have an attorney review.

---

### 18. onboard-gen

- **Command:** `/onboard`
- **Purpose:** Generate contributor documentation and architecture walkthroughs
- **Input:** Scans entire codebase and existing docs
- **Output directory:** `.onboard/`
- **Output:** Architecture guide, setup guide, contribution guide, key flow walkthroughs

**Agents (5):**

| Agent | Role | Tools |
|-------|------|-------|
| `architecture-documenter` | Maps system architecture, components, data flow, tech stack | Read, Glob, Grep, Bash |
| `flow-tracer` | Traces key user flows through the code (auth, core feature, data pipeline) | Read, Glob, Grep, Bash |
| `setup-guide-writer` | Documents setup steps, env vars, dependencies, first-run instructions | Read, Glob, Grep, Bash |
| `convention-extractor` | Documents coding conventions, patterns, naming, file organization | Read, Glob, Grep, Bash |
| `onboard-synthesizer` | Combines into polished onboarding documentation suite | Read, Glob, Grep |

**Skill:** `onboard-patterns` -- documentation structures, architecture diagram conventions, onboarding checklist frameworks

---

## Pipeline Context Chain

```
.spark/business-case.md
    |
    v
.stack/tech-decisions.md  (reads .spark/)
    |
    v
.roadmap/roadmap.md  (reads .spark/ + .stack/)
    |
    v
scaffold-kit  (reads .stack/ + .roadmap/)
    |
    v
.build/progress.md  (reads .roadmap/)
```

Utility plugins can optionally read upstream outputs:
- `pitch-craft` reads `.spark/business-case.md`
- `price-model` reads `.costforecast/report.md`
- `ship-page` reads `.spark/`, `.pricing/`
- `cost-forecast` reads `.stack/tech-decisions.md`

---

## Agent Count Summary

| Plugin | Agents | Category |
|--------|--------|----------|
| idea-spark | 5 | Pipeline |
| stack-pick | 5 | Pipeline |
| codebase-roadmap | 7 | Pipeline (existing) |
| scaffold-kit | 5 | Pipeline |
| build-engine | 5 | Pipeline |
| codebase-ideation | 7 | Code (existing) |
| codebase-autopsy | 5 | Code |
| launch-check | 5 | Code |
| test-gap | 5 | Code |
| dep-doctor | 5 | Code |
| cost-forecast | 4 | DevOps |
| incident-planner | 4 | DevOps |
| scale-sim | 5 | DevOps |
| pitch-craft | 5 | Business |
| price-model | 4 | Business |
| ship-page | 4 | Business |
| legal-kit | 4 | Business |
| onboard-gen | 5 | Business |
| **Total** | **87** | |

---

## Implementation Order (Recommended)

Build in waves, starting with the pipeline core:

**Wave 1 -- Pipeline core:**
1. idea-spark
2. stack-pick
3. scaffold-kit
4. build-engine

**Wave 2 -- Highest-impact utilities:**
5. launch-check
6. codebase-autopsy
7. cost-forecast
8. onboard-gen

**Wave 3 -- Remaining code/ops utilities:**
9. test-gap
10. dep-doctor
11. incident-planner
12. scale-sim

**Wave 4 -- Business utilities:**
13. pitch-craft
14. price-model
15. ship-page
16. legal-kit
