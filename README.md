# Solo Dev Toolbox

**Ship like a funded startup. Build alone.**

18 Claude Code plugins that give a solo developer the firepower of an entire product team — market research, architecture, TDD, scaling analysis, pitch decks, pricing strategy, landing pages, and legal docs. All from your terminal.

**89 specialized AI agents. 18 slash commands. Zero context switching.**

```bash
/spark "AI-powered code review tool"   # Validate the idea
/stack                                  # Pick the tech stack
/scaffold                               # Bootstrap the project
/build --milestone 1                    # Build with TDD
/preflight                              # Check production readiness
/landing --format both --tone bold      # Generate the landing page
/pricing --model freemium               # Design the pricing
/pitch --stage pre-seed                 # Create the pitch deck
/comply                                 # Generate legal docs
```

---

## Why Solo Dev Toolbox?

Building a product alone means wearing every hat — developer, architect, QA engineer, DevOps, product manager, marketer, and sometimes lawyer. Each of those roles has specialized knowledge that takes years to develop. You don't have years. You have a weekend and an idea.

**The problem isn't code.** AI can help you write code. The problem is everything *around* the code:

- Is this idea even worth building? Who are the competitors? What's the market size?
- What tech stack won't bite you at 10K users?
- Where are the security holes you don't know about?
- What does your infrastructure cost at scale?
- How do you write a pitch deck that doesn't get laughed out of the room?
- What should you charge? How do you design pricing tiers?
- What goes in a privacy policy when your app touches user data?

**Solo Dev Toolbox answers all of these.** Each plugin runs a team of specialized AI agents that analyze your actual codebase and produce real deliverables — not generic advice, but documents built from your specific code, your specific architecture, your specific data flows.

### What You Get

| Without the Toolbox | With the Toolbox |
|---------------------|-----------------|
| Google "how to write a pitch deck" and stare at templates | `/pitch` analyzes your codebase and business case, generates a stage-appropriate deck with market sizing and financials |
| Manually grep for security issues you know about | `/preflight` runs 50+ checks across security, ops, reliability, and docs with a GO/NO-GO verdict |
| Guess at pricing based on competitors you found | `/pricing` triangulates cost floor + market rate + value ceiling with specific tier recommendations |
| Copy a generic privacy policy from the internet | `/comply` scans your actual data flows and generates regulation-specific documents with gap analysis |
| Wonder if your app will survive a traffic spike | `/simulate` models your architecture at 10x/100x/1000x and tells you exactly what breaks first |
| Write tests for whatever feels important | `/coverage` risk-ranks every untested path and generates skeleton test files |

### The Solo Dev Advantage

Big teams have specialists. You have something better: **a pipeline that connects every stage of product development.** When you run `/pricing`, it already knows your infrastructure costs from `/estimate`. When you run `/pitch`, it pulls your market research from `/spark` and your tech decisions from `/stack`. When you run `/comply`, it scans the actual data handling in your codebase — not a questionnaire.

Each plugin builds on the last. No copy-pasting between tools. No context lost between conversations. One terminal, one codebase, one pipeline from idea to shipped product.

---

## Quick Start

### Option A: Interactive Pipeline with solodev (Recommended)

[**solodev**](https://github.com/stevenkozeniesky02/solodev) wraps all 18 plugins in an interactive TUI with persistent context — every plugin run builds on the last, so by step 15, Claude knows your entire product history.

```bash
npm install -g solodev
solodev
```

No API keys. No flags to remember. Just a visual pipeline that guides you from idea to shipped product.

### Option B: Direct Plugin Installation

Install the plugins directly into Claude Code for use as slash commands:

```bash
claude mcp add sk-plugins -- npx @anthropic-ai/claude-code-plugins \
  --url https://github.com/stevenkozeniesky02/claude-plugins

# Then use any command inside a Claude Code session
/spark "your idea here"
```

---

## Table of Contents

- [Plugin Overview](#plugin-overview)
- [Plugin Chain](#plugin-chain)
- [Detailed Plugin Reference](#detailed-plugin-reference)
  - [Idea to Architecture](#idea-to-architecture)
  - [Build and Improve](#build-and-improve)
  - [Test and Harden](#test-and-harden)
  - [Scale and Operate](#scale-and-operate)
  - [Go to Market](#go-to-market)
- [Architecture](#architecture)
- [Output Directories](#output-directories)
- [Plugin File Structure](#plugin-file-structure)
- [License](#license)

---

## Plugin Overview

| # | Plugin | Command | Agents | Category | One-Liner |
|---|--------|---------|:------:|----------|-----------|
| 1 | idea-spark | `/spark` | 5 | Business | Validate business ideas with market research and feasibility scoring |
| 2 | stack-pick | `/stack` | 5 | Development | Evaluate and recommend optimal tech stacks |
| 3 | scaffold-kit | `/scaffold` | 5 | Development | Bootstrap project skeletons with CI/CD, Docker, linting |
| 4 | codebase-roadmap | `/generate` | 7 | Development | Produce phased roadmaps from codebase domain analysis |
| 5 | build-engine | `/build` | 5 | Development | Implement milestones with TDD multi-agent coding |
| 6 | codebase-ideation | `/discover` | 7 | Development | Scan codebases for improvement ideas across 6+ categories |
| 7 | codebase-autopsy | `/autopsy` | 5 | Development | Deep tech debt analysis with risk-scored time bombs |
| 8 | onboard-gen | `/onboard` | 5 | Development | Generate contributor docs and architecture walkthroughs |
| 9 | test-gap | `/coverage` | 5 | Development | Find untested critical paths with risk-ranked coverage maps |
| 10 | dep-doctor | `/checkup` | 5 | Development | Dependency health audit — CVEs, freshness, licenses, bloat |
| 11 | launch-check | `/preflight` | 5 | DevOps | Production readiness audit with GO/NO-GO scorecard |
| 12 | cost-forecast | `/estimate` | 4 | DevOps | Estimate infra and API costs at different user scales |
| 13 | scale-sim | `/simulate` | 5 | DevOps | Identify scaling bottlenecks at 10x/100x/1000x |
| 14 | incident-planner | `/runbook` | 4 | DevOps | Generate runbooks and incident response playbooks |
| 15 | pitch-craft | `/pitch` | 5 | Business | Generate pitch decks, one-pagers, and investor FAQ |
| 16 | price-model | `/pricing` | 4 | Business | Design pricing strategies with competitive analysis |
| 17 | ship-page | `/landing` | 4 | Business | Generate landing page copy and optional responsive HTML |
| 18 | legal-kit | `/comply` | 4 | Business | Generate privacy policies, terms, and cookie policies |

---

## Plugin Chain

Plugins are designed to feed into each other. Upstream outputs are automatically detected — for example, `/pricing` reads cost data from `/estimate`, and `/pitch` pulls business context from `/spark`.

```
                        THE SOLO DEV PIPELINE

    VALIDATE          BUILD             HARDEN            SHIP
    ───────          ─────             ──────            ────
    /spark    ──→    /scaffold   ──→   /coverage   ──→   /pitch
       │                │                 │                 │
    /stack    ──→    /generate  ──→   /checkup    ──→   /pricing
       │                │                 │                 │
                     /build     ──→   /preflight  ──→   /landing
                        │                 │                 │
                     /discover  ──→   /estimate   ──→   /comply
                        │                 │
                     /autopsy   ──→   /simulate
                        │                 │
                     /onboard   ──→   /runbook
```

**Recommended first-time flow:**

```bash
/spark "your idea here"       # 1. Validate the idea
/stack                        # 2. Pick your tech stack
/scaffold                     # 3. Bootstrap the project
/generate                     # 4. Create a development roadmap
/build --milestone 1          # 5. Build milestone by milestone
/coverage                     # 6. Find and fill test gaps
/preflight                    # 7. Check production readiness
/landing --format both        # 8. Generate your landing page
/pricing --target b2c         # 9. Design your pricing
/pitch --stage pre-seed       # 10. Pitch to investors
/comply                       # 11. Generate legal docs
```

---

## Detailed Plugin Reference

### Idea to Architecture

#### `/spark` — idea-spark

Validate and refine a business idea through multi-agent market research, competitive analysis, revenue modeling, and feasibility assessment.

```bash
/spark "AI-powered code review tool for solo developers"
/spark "marketplace for freelance data scientists"
```

| Agent | Role |
|-------|------|
| market-researcher | TAM/SAM/SOM sizing, growth rates, industry trends via web search |
| competitor-scout | Competing products, features, pricing, positioning, strengths/weaknesses |
| revenue-modeler | Revenue models, pricing structures, unit economics, break-even analysis |
| feasibility-analyst | Technical complexity, timeline, risks within solo-dev constraints |
| spark-synthesizer | Merges all findings into a business case with go/no-go verdict |

**Output:** `.spark/business-case.md` — Complete business case with market analysis, competitive landscape, revenue projections, and feasibility assessment.

**Skill patterns:** Lean Canvas, competitive moat analysis, go/no-go decision frameworks.

---

#### `/stack` — stack-pick

Research and recommend optimal tech stacks by evaluating backend frameworks, frontend tools, infrastructure, and developer experience.

```bash
/stack --requirements "real-time collaboration, <1s latency" --budget low
/stack --requirements "e-commerce, SEO-critical, mobile-first"
```

| Agent | Role |
|-------|------|
| backend-researcher | Languages, frameworks, ORMs, performance benchmarks, hosting costs |
| frontend-researcher | Bundle sizes, component libraries, SSR/SSG, build tooling |
| infra-researcher | Hosting, databases, caching, queues, CI/CD pricing and free tiers |
| dx-analyst | Documentation quality, error messages, community support, solo-dev ergonomics |
| stack-synthesizer | 2-3 complete stack recommendations ranked by project fit |

**Output:** `.stack/recommendation.md` — Ranked stack recommendations with trade-off analysis, cost estimates, and migration paths.

**Skill patterns:** Build vs buy, monolith vs micro, managed vs self-hosted decision frameworks.

---

#### `/scaffold` — scaffold-kit

Bootstrap a project skeleton from tech stack decisions with directory structure, CI/CD pipelines, Docker configs, linting, and documentation.

```bash
/scaffold --stack "FastAPI + React + PostgreSQL"
/scaffold --minimal
```

| Agent | Role |
|-------|------|
| project-structurer | Directory layout following stack-specific conventions with module boundaries |
| cicd-generator | CI/CD pipelines, multi-stage Dockerfile, docker-compose for local dev |
| config-generator | Package manager, linting, formatting, pre-commit hooks, .gitignore |
| docs-bootstrapper | README with 5-minute setup, CONTRIBUTING guide, .env.example |
| scaffold-assembler | Merges all generators into a conflict-free scaffold ready for creation |

**Output:** `.scaffold/scaffold-plan.md` — Complete file listing with contents, ordered for creation. Approved by you before any files are written.

**Skill patterns:** Per-stack project structure conventions, CI/CD templates, config best practices.

---

#### `/generate` — codebase-roadmap

Analyze an existing codebase by domain (backend, frontend, infra, security) with market research to produce a phased roadmap with dependency tracking.

```bash
/generate --goal "add real-time collaboration" --horizon full
/generate --goal "prepare for 10K users" --format json
```

| Agent | Role |
|-------|------|
| project-scanner | Deep structural analysis — stack, maturity, test coverage, CI/CD state |
| backend-analyst | API design, data layer, business logic, integration milestones |
| frontend-analyst | Components, state management, routing, design system, accessibility |
| infra-analyst | Deployment, CI/CD, monitoring, containerization milestones |
| security-analyst | Auth, secrets, input validation, dependency vulns, compliance |
| market-research-analyst | Competitive landscape and market justification for prioritization |
| roadmap-synthesizer | Unified multi-phase roadmap with dependency graph and exit criteria |

**Output:** `.roadmap/roadmap.md` — Phased roadmap with dependency tracking, milestone ordering, exit criteria per phase, and market justification.

**Skill patterns:** Dependency graph construction, phase exit criteria, milestone prioritization.

---

### Build and Improve

#### `/build` — build-engine

Implement roadmap milestones feature-by-feature using TDD multi-agent coding with planning, test-first development, implementation, and code review.

```bash
/build --milestone 1
/build --resume
```

| Agent | Role |
|-------|------|
| milestone-planner | Breaks milestones into 15-45 minute tasks with exact file paths |
| test-writer | Writes failing tests BEFORE implementation (TDD red phase) |
| implementer | Writes minimal code to make tests pass (TDD green phase) |
| code-reviewer | Reviews for bugs, security, convention deviations — APPROVE or REQUEST CHANGES |
| build-orchestrator | Tracks multi-milestone progress, identifies blockers, generates status reports |

**Output:** `.build/` — Implementation progress, build metrics, and review reports. Code is written directly into your project.

**Skill patterns:** TDD red-green-refactor, commit conventions, milestone decomposition.

---

#### `/discover` — codebase-ideation

Scan your codebase for actionable improvement ideas across code quality, security, performance, docs, UX, and dynamically detected categories.

```bash
/discover --depth deep
/discover --categories "DevOps,Testing"
```

| Agent | Role |
|-------|------|
| code-analyst | Architectural improvements, tech debt, missing abstractions, dead code |
| security-analyst | Missing auth, unvalidated inputs, hardcoded secrets, OWASP gaps |
| performance-analyst | N+1 queries, missing caching, unoptimized loops, memory issues |
| docs-analyst | Undocumented modules, stale docs, missing API docs, onboarding gaps |
| ux-analyst | Accessibility gaps, missing loading states, responsive design issues |
| market-research-analyst | Competitor features and industry trends to inform priorities |
| idea-curator | Deduplicates, ranks by impact, enriches with market signals |

**Output:** `.ideation/ideas.md` — Ranked idea cards with impact scores, effort estimates, and market context.

**Skill patterns:** Idea card format, impact-effort prioritization, category detection heuristics.

---

#### `/autopsy` — codebase-autopsy

Deep tech debt analysis with risk-scored time bombs, complexity mapping, pattern archaeology, and prioritized refactoring recommendations.

```bash
/autopsy --scope src/api --severity critical
/autopsy
```

| Agent | Role |
|-------|------|
| debt-detector | TODOs, hardcoded values, copy-paste code, dead code, workarounds |
| complexity-analyst | Function length, nesting depth, cyclomatic complexity, coupling/cohesion |
| pattern-archaeologist | Deprecated APIs, legacy conventions, inconsistent styles, abandoned migrations |
| risk-scorer | Detonation risk by blast radius, trigger probability, time sensitivity |
| autopsy-synthesizer | Health verdict with prioritized fix plan and risk timeline |

**Output:** `.autopsy/report.md` — Tech debt inventory with risk scores, complexity heatmap, and prioritized refactoring plan.

**Skill patterns:** Tech debt classification, risk scoring methodology, refactoring prioritization.

---

#### `/onboard` — onboard-gen

Generate contributor documentation and architecture walkthroughs by scanning your codebase for structure, flows, setup requirements, and coding conventions.

```bash
/onboard --format guide
/onboard --scope src/core --format wiki
```

| Agent | Role |
|-------|------|
| architecture-documenter | Component mapping, layer diagram, data flow, ASCII architecture diagrams |
| flow-tracer | Traces key user flows from entry point to completion with file paths |
| setup-guide-writer | Prerequisites, dependency install, database setup, env vars, first-run verification |
| convention-extractor | Naming patterns, file organization, error handling patterns, style decisions |
| onboard-synthesizer | Polished, cohesive onboarding documentation suite |

**Output:** `.onboard/` — Architecture docs, flow traces, setup guide, and convention reference.

**Skill patterns:** Documentation structure templates, architecture diagram conventions, onboarding checklists.

---

### Test and Harden

#### `/coverage` — test-gap

Find untested critical paths and generate test strategies with risk-ranked coverage maps and optional skeleton test files.

```bash
/coverage --generate-skeletons
/coverage --scope src/auth
```

| Agent | Role |
|-------|------|
| coverage-mapper | Maps source modules to test files, flags shallow and orphan tests |
| risk-path-finder | Untested auth, payment, data mutation, error handling, security paths |
| test-strategist | Recommends test types per path — unit, integration, e2e, property-based |
| skeleton-writer | Generates test file skeletons matching your codebase's testing conventions |
| coverage-synthesizer | Prioritized coverage improvement plan with A-F grade |

**Output:** `.testgap/coverage-report.md` — Coverage matrix, risk-ranked gaps, test strategy, and optional skeleton files.

**Skill patterns:** Testing pyramid, risk-based prioritization (P0-P6), effort estimation.

---

#### `/checkup` — dep-doctor

Dependency health audit with CVE scanning, freshness checks, license auditing, and bloat detection.

```bash
/checkup --fix
/checkup --scope runtime
```

| Agent | Role |
|-------|------|
| vuln-scanner | CVEs via native audit tools, exploitability assessment |
| freshness-checker | Outdated deps, breaking changes, abandoned packages via web search |
| license-auditor | License compatibility, copyleft flags, unknown/missing licenses |
| bloat-detector | Unused deps, duplicate packages, heavy transitive trees |
| dep-synthesizer | Health grade A-F with prioritized upgrade/replace/remove actions |

**Output:** `.depdoctor/health-report.md` — Vulnerability report, freshness status, license audit, bloat analysis, and maintenance schedule.

**Skill patterns:** Version pinning strategies, license compatibility matrix, dependency anti-patterns.

---

#### `/preflight` — launch-check

Production readiness audit with 50+ checks across security, ops, reliability, and documentation producing a GO/NO-GO scorecard.

```bash
/preflight --strict
/preflight --category security
```

| Agent | Role |
|-------|------|
| security-checker | Auth, secrets, input validation, dependency vulns, CORS, injection risks |
| ops-checker | CI/CD, Docker, deployment strategy, health checks, monitoring |
| reliability-checker | Error handling, retries, circuit breakers, graceful degradation, logging |
| docs-checker | README, API docs, setup guide, architecture docs, changelog |
| preflight-synthesizer | GO / CONDITIONAL GO / NO-GO verdict with scorecard |

**Output:** `.preflight/scorecard.md` — Category scores, detailed findings, and prioritized fix list.

**Skill patterns:** Production readiness checklists, severity classification, industry benchmarks.

---

#### `/estimate` — cost-forecast

Estimate infrastructure and API costs at different user scales with cost cliff identification, optimization recommendations, and burn rate projections.

```bash
/estimate --scale 10K --budget low
/estimate --scale 100K
```

| Agent | Role |
|-------|------|
| infra-cost-analyst | Hosting, database, caching, storage, compute costs per tier |
| api-cost-analyst | LLM usage, payment processing, email, auth, external API costs per tier |
| scale-modeler | Total costs at 100/1K/10K/100K users, cost cliff identification, burn rate |
| cost-synthesizer | Comprehensive forecast with optimization recs and budget alerts |

**Output:** `.costforecast/report.md` — Cost breakdown by tier, cost cliffs, optimization opportunities, and runway projections.

**Skill patterns:** Cloud pricing models, free tier reference tables, unit economics frameworks.

---

### Scale and Operate

#### `/simulate` — scale-sim

Identify scaling bottlenecks before they hit production with bottleneck hunting, database profiling, queue analysis, and capacity modeling.

```bash
/simulate --baseline 1000
/simulate --scope src/api
```

| Agent | Role |
|-------|------|
| bottleneck-hunter | SPOFs, synchronous bottlenecks, resource contention, scaling limits |
| database-profiler | N+1 queries, missing indexes, unbounded queries, connection pool issues |
| queue-analyst | Queue throughput, backpressure, dead letters, sync-should-be-async |
| capacity-modeler | Behavior at 10x/100x/1000x with specific resource numbers and break points |
| scale-synthesizer | Scaling grade A-F, bottleneck chain, and scaling roadmap |

**Output:** `.scalesim/report.md` — Bottleneck inventory, database analysis, queue assessment, and capacity model with scaling roadmap.

**Skill patterns:** Scaling hierarchy (6 levels), capacity planning formulas, horizontal scaling checklist.

---

#### `/runbook` — incident-planner

Generate runbooks and incident response playbooks from architecture analysis with failure mode identification and severity classification.

```bash
/runbook --severity critical
/runbook --scope src/payments
```

| Agent | Role |
|-------|------|
| architecture-mapper | Services, dependencies, communication paths, data flows, SPOFs |
| failure-mode-analyst | Crash, timeout, corruption, capacity, config failures with SEV1-4 |
| runbook-writer | Step-by-step diagnosis, mitigation, recovery, and verification procedures |
| runbook-synthesizer | Master incident response plan with severity matrix and quick reference |

**Output:** `.runbook/` — Architecture map, failure mode catalog, per-scenario runbooks, and master incident response plan.

**Skill patterns:** Severity classification, runbook templates, post-mortem format, monitoring essentials.

---

### Go to Market

#### `/pitch` — pitch-craft

Generate investor pitch decks and one-pagers from business context and codebase analysis, tailored to funding stage.

```bash
/pitch --format all --stage pre-seed
/pitch --format deck --stage seed
```

| Agent | Role |
|-------|------|
| narrative-builder | Story arc — hook, problem, solution, traction, ask |
| market-sizer | TAM/SAM/SOM with bottom-up and top-down estimates via web research |
| financial-modeler | Revenue projections, unit economics, burn rate appropriate to stage |
| deck-writer | Slide-by-slide content with speaker notes following Sequoia/YC frameworks |
| pitch-synthesizer | Deck outline, one-pager, elevator pitches (30s/60s), investor FAQ (10 Qs) |

**Reads from:** `.spark/` (business case), `.stack/` (tech decisions), `.costforecast/` (cost data)

**Output:** `.pitch/` — Pitch deck outline, one-pager, elevator pitches, and investor FAQ.

**Skill patterns:** Sequoia/YC deck frameworks, Hero's Journey storytelling, stage-appropriate metrics.

---

#### `/pricing` — price-model

Design pricing strategies through competitive analysis, unit economics modeling, and value metric evaluation.

```bash
/pricing --model subscription --target b2b
/pricing --model freemium --target prosumer
```

| Agent | Role |
|-------|------|
| competitor-pricing-analyst | Competitor tiers, prices, feature gates via web research |
| unit-economics-modeler | COGS per user, margins, break-even, pricing floor, scale economics |
| value-metric-analyst | Optimal value metric (seats/usage/features/flat), tier structure, upgrade triggers |
| pricing-synthesizer | Price triangulation (cost floor + market + value ceiling), specific price points |

**Reads from:** `.costforecast/` (cost data), `.spark/` (business context)

**Output:** `.pricing/pricing-strategy.md` — Competitive landscape, unit economics, tier design, and implementation guide.

**Skill patterns:** Pricing model decision tree, psychological pricing, freemium conversion benchmarks.

---

#### `/landing` — ship-page

Generate landing page copy and optional production-ready HTML from codebase feature extraction, benefit mapping, and conversion-optimized copywriting.

```bash
/landing --format both --tone bold
/landing --format copy --tone professional
```

| Agent | Role |
|-------|------|
| feature-extractor | Scans routes, UI components, integrations for user-facing features |
| benefit-mapper | Transforms features into benefits using the "So what?" method |
| copywriter | Hero, feature blocks, social proof, FAQ, CTAs in the specified tone |
| page-assembler | Polished landing page doc + optional self-contained responsive HTML |

**Reads from:** `.spark/` (product context), `.pricing/` (pricing tiers), `.pitch/` (narrative)

**Output:** `.shippage/landing-page.md` and optionally `.shippage/index.html` — Complete landing page content with SEO metadata.

**Skill patterns:** PAS/AIDA/BAB copywriting frameworks, above-the-fold rules, CTA optimization.

---

#### `/comply` — legal-kit

Generate compliance documentation by scanning your codebase for data handling patterns. Produces privacy policies, terms of service, and cookie policies with GDPR/CCPA gap analysis.

> **Disclaimer:** Output is a starting point — not legal advice. Always have a qualified attorney review generated documents.

```bash
/comply --regulation gdpr
/comply --scope privacy --regulation both
```

| Agent | Role |
|-------|------|
| data-flow-mapper | Personal data collection, storage, processing, third-party sharing patterns |
| regulation-matcher | GDPR/CCPA requirement mapping, consent gaps, cross-border transfer issues |
| doc-drafter | Privacy policy, terms of service, cookie policy drafts with placeholders |
| compliance-synthesizer | Final package with compliance grade A-F, gap analysis, and action plan |

**Output:** `.legalkit/compliance-package.md` — Draft documents, compliance gaps, action plan, placeholder summary, and maintenance schedule.

**Skill patterns:** GDPR Article 13/14 structure, CCPA requirements, cookie consent rules, legal basis decision tree.

---

## Architecture

Every plugin follows the same three-phase pipeline:

```
Phase 0: Survey / Context Gathering
│
│   Sequential. Scans the codebase or asks the user for context.
│   Produces a baseline document that all specialists receive.
│
├──→ Phase 1: Parallel Specialists
│    │
│    │   3-7 domain experts run simultaneously.
│    │   Each receives Phase 0 output + their domain expertise.
│    │   Each produces an independent analysis.
│    │
│    ├── Specialist A  ──→  analysis-a.md
│    ├── Specialist B  ──→  analysis-b.md
│    ├── Specialist C  ──→  analysis-c.md
│    └── Specialist D  ──→  analysis-d.md
│
└──→ Phase 2: Synthesis
     │
     │   Sequential. One synthesizer reads ALL specialist outputs.
     │   Merges, deduplicates, resolves conflicts, and produces
     │   the final deliverable.
     │
     └── Synthesizer  ──→  final-deliverable.md
```

### Model Strategy

- **Specialists** use `model: inherit` — fast, focused, domain-specific analysis
- **Synthesizers** use `model: opus` — deep reasoning for merging complex, potentially conflicting analyses

### Tool Assignments

| Agent Type | Tools | Rationale |
|-----------|-------|-----------|
| Codebase scanners | `Read, Glob, Grep, Bash` | Need to explore files and run commands |
| Read-only analysts | `Read, Glob, Grep` | Analyze without side effects |
| Web researchers | `Read, Glob, Grep, WebSearch, WebFetch` | Need external data |
| Synthesizers | `Read, Glob, Grep` | Read specialist outputs, no execution |

### State Management

Every plugin tracks execution state in its output directory:

```json
{
  "status": "in_progress",
  "flags": { "scope": "all" },
  "current_phase": 1,
  "completed_phases": [0],
  "files_created": ["00-survey.md"],
  "started_at": "2026-03-03T10:00:00Z",
  "last_updated": "2026-03-03T10:05:00Z"
}
```

If a command is interrupted, re-running it detects the existing session and offers to resume or start fresh.

---

## Output Directories

Each plugin writes results to a dot-directory in your project root:

| Plugin | Output Directory | Key File |
|--------|-----------------|----------|
| idea-spark | `.spark/` | `business-case.md` |
| stack-pick | `.stack/` | `recommendation.md` |
| scaffold-kit | `.scaffold/` | `scaffold-plan.md` |
| codebase-roadmap | `.roadmap/` | `roadmap.md` |
| build-engine | `.build/` | `status.md` |
| codebase-ideation | `.ideation/` | `ideas.md` |
| codebase-autopsy | `.autopsy/` | `report.md` |
| onboard-gen | `.onboard/` | `guide.md` |
| test-gap | `.testgap/` | `coverage-report.md` |
| dep-doctor | `.depdoctor/` | `health-report.md` |
| launch-check | `.preflight/` | `scorecard.md` |
| cost-forecast | `.costforecast/` | `report.md` |
| scale-sim | `.scalesim/` | `report.md` |
| incident-planner | `.runbook/` | `incident-plan.md` |
| pitch-craft | `.pitch/` | `deck-outline.md` |
| price-model | `.pricing/` | `pricing-strategy.md` |
| ship-page | `.shippage/` | `landing-page.md` |
| legal-kit | `.legalkit/` | `compliance-package.md` |

Add these to your `.gitignore` if you don't want generated output in version control:

```gitignore
.spark/
.stack/
.scaffold/
.roadmap/
.build/
.ideation/
.autopsy/
.onboard/
.testgap/
.depdoctor/
.preflight/
.costforecast/
.scalesim/
.runbook/
.pitch/
.pricing/
.shippage/
.legalkit/
```

---

## Plugin File Structure

Every plugin follows a consistent layout:

```
plugins/{name}/
├── .claude-plugin/
│   └── plugin.json          # Name, description, version, category
├── commands/
│   └── {command}.md         # Slash command — orchestrates the pipeline
├── agents/
│   ├── specialist-1.md      # Domain specialist (model: inherit)
│   ├── specialist-2.md      # Domain specialist (model: inherit)
│   ├── specialist-3.md      # Domain specialist (model: inherit)
│   └── synthesizer.md       # Result synthesizer (model: opus)
└── skills/
    └── {skill-name}/
        └── SKILL.md         # Domain knowledge, frameworks, patterns
```

**Agent files** contain YAML frontmatter (`name`, `description`, `model`, `tools`) followed by a persona, expert purpose, analysis/synthesis strategy, output format template, and rules.

**Command files** contain YAML frontmatter (`description`, `argument-hint`) followed by 5 behavioral rules, pre-flight checks, and the three-phase pipeline definition.

**Skill files** contain domain-specific knowledge — frameworks, checklists, decision trees, and reference tables — that agents and commands can reference.

---

## Contributing

Found a bug or want to add a plugin? Open an issue or PR.

Plugin conventions:
- One command per plugin
- Specialists use `model: inherit`, synthesizers use `model: opus`
- All output goes to a dot-directory (`.pluginname/`)
- State tracking via `state.json` for resumable sessions
- Skills contain domain knowledge, not orchestration logic

---

## License

MIT

## Author

Steven Kozeniesky

## Links

- [solodev](https://github.com/stevenkozeniesky02/solodev) — Interactive CLI that orchestrates all 18 plugins with persistent context
- [Claude Code](https://claude.ai/code) — The AI engine underneath
