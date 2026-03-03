---
name: stack-patterns
description: Decision frameworks for tech stack selection including build vs buy, monolith vs micro, managed vs self-hosted, and solo dev optimization strategies. Use when choosing technologies for a new project or evaluating architecture decisions.
---

# Stack Patterns

Decision frameworks for choosing the right technology stack quickly and pragmatically.

## When to Use This Skill

- Running the `/stack` command to evaluate tech stack options
- Deciding whether to build a component or buy/use a managed service
- Evaluating monolith vs. microservices for a new project
- Choosing between managed and self-hosted infrastructure as a solo developer

## The Solo Dev Stack Hierarchy

When choosing technologies as a solo developer, prioritize in this order:

| Priority | Dimension | Why It Matters Most |
|----------|-----------|-------------------|
| 1 | **Shipping speed** | Time to market is your only real advantage over funded teams |
| 2 | **Operational simplicity** | You are the entire ops team — every moving part is your on-call burden |
| 3 | **Cost** | Bootstrap budgets are real — free tiers and cheap scaling matter |
| 4 | **Ecosystem** | Libraries, plugins, and integrations save weeks of development |
| 5 | **Performance** | Good enough performance beats perfect performance you never ship |
| 6 | **Scalability** | Solve scaling problems when you have scaling problems, not before |

## Build vs Buy Decision Matrix

| Component | Build | Buy/Use Managed | Reasoning |
|-----------|-------|-----------------|-----------|
| **Core differentiator** | BUILD | - | This is your product's unique value — own it completely |
| **Commodity feature** | - | BUY | Auth, payments, email — solved problems with excellent services |
| **Data storage** | - | MANAGED | Never self-host databases as a solo dev — use managed Postgres, Supabase, PlanetScale |
| **Infrastructure** | - | PaaS | Use Vercel, Railway, Fly.io, Render — not raw AWS/GCP unless you must |
| **Monitoring** | - | FREE TIER | Sentry free tier, Grafana Cloud free tier — do not skip monitoring |

**The rule:** If it is not your core product, do not build it. Your time is the scarcest resource.

## Monolith vs Microservices (Solo Dev Edition)

**Always start with a monolith.** This is not a suggestion — it is a rule for solo developers.

| "Reason" to Go Micro | Why It Is Wrong for Solo Devs |
|----------------------|------------------------------|
| "It scales better" | A well-built monolith handles 100K+ users. You do not have that problem yet. |
| "Separation of concerns" | Use modules/packages inside the monolith. Same separation, zero network overhead. |
| "Independent deployments" | You are one person. You are not blocked by another team's deploy schedule. |
| "Technology diversity" | Pick one language and go deep. Polyglot stacks multiply your operational burden. |
| "Fault isolation" | A monolith with good error handling is more reliable than distributed services with network failures. |

**When to split (the only valid reasons):**
- You have actual performance data showing a specific component is a bottleneck
- A component needs fundamentally different scaling characteristics (e.g., video transcoding)
- You are handing off a component to a different team or developer

## Managed vs Self-Hosted

| Component | Recommendation | Why |
|-----------|---------------|-----|
| **Database** | Managed (Supabase, Neon, PlanetScale) | Backups, scaling, monitoring included — self-hosting Postgres is a full-time job |
| **Redis/Cache** | Managed (Upstash, Redis Cloud) | Free tiers are generous, zero maintenance |
| **Search** | Managed (Algolia, Meilisearch Cloud, Typesense Cloud) | Search infrastructure is complex — let someone else run it |
| **CI/CD** | Managed (GitHub Actions, GitLab CI) | Free tiers cover solo dev needs entirely |
| **Hosting** | PaaS (Vercel, Railway, Fly.io, Render) | Git-push deploys, auto-scaling, SSL, logs — all included |
| **Kubernetes** | NEVER for solo dev | The operational overhead of K8s will consume all your development time. Use a PaaS. |

## Technology Lifecycle Stages

Choose technologies at the right lifecycle stage for your risk tolerance:

| Stage | Examples | Solo Dev Verdict |
|-------|----------|-----------------|
| **Bleeding edge** | Pre-1.0 frameworks, experimental APIs | AVOID — you will hit undocumented bugs with no Stack Overflow answers |
| **Early adopter** | Recently stable, growing fast, some rough edges | RISKY — only adopt if you are comfortable reading source code to debug |
| **Growth** | 2-5 years old, large community, stable APIs, good docs | GOOD SWEET SPOT — modern enough to be pleasant, mature enough to be reliable |
| **Mature** | 5-15 years old, battle-tested, huge ecosystem, boring | SAFE BET — Rails, Django, Laravel, Express. Boring is a feature. |
| **Declining** | Shrinking community, fewer updates, talent leaving | AVOID — even if it still works, the ecosystem is contracting and help is harder to find |
