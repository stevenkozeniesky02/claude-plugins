---
name: cost-patterns
description: Cloud pricing models, cost optimization strategies, free tier reference tables, and unit economics frameworks for infrastructure cost forecasting. Use when estimating or optimizing project costs.
---

# Cost Patterns

Frameworks and reference data for estimating and optimizing infrastructure costs.

## When to Use This Skill

- Running the `/estimate` command
- Manually estimating infrastructure costs
- Comparing hosting or service providers
- Optimizing existing infrastructure costs

## Free Tier Reference (Common Services)

### Hosting / Compute

| Provider | Free Tier | First Paid Tier | Cost Cliff |
|----------|-----------|----------------|-----------|
| Vercel | 100 GB bandwidth, 100K function invocations | $20/mo Pro | Functions, bandwidth overages |
| Railway | $5 credit/mo | $5/mo + usage | Runs out fast with always-on services |
| Fly.io | 3 shared VMs, 160 GB bandwidth | ~$2/mo per extra VM | CPU/memory scaling |
| Render | 750 hours free instance (spins down) | $7/mo always-on | Cold starts in free tier |
| Cloudflare Workers | 100K requests/day | $5/mo (10M requests) | Very generous free tier |

### Databases

| Provider | Free Tier | First Paid Tier | Cost Cliff |
|----------|-----------|----------------|-----------|
| Supabase | 500 MB, 2 GB bandwidth | $25/mo Pro | Storage and bandwidth |
| Neon | 0.5 GB storage, 191 compute hours | $19/mo | Compute hours |
| PlanetScale | 1 GB storage, 1B row reads | $29/mo Scaler | Row reads at scale |
| MongoDB Atlas | 512 MB shared | $57/mo dedicated | Shared to dedicated jump |
| Turso | 9 GB storage, 500 databases | $29/mo Scaler | Storage and databases |

### Auth

| Provider | Free Tier | First Paid Tier | Cost Cliff |
|----------|-----------|----------------|-----------|
| Supabase Auth | 50K MAU | Included in Pro ($25/mo) | Generous free tier |
| Clerk | 10K MAU | $25/mo Pro | MAU-based scaling |
| Auth0 | 7,500 MAU | $23/mo Essential | MAU-based scaling |

### Email

| Provider | Free Tier | First Paid Tier | Cost Cliff |
|----------|-----------|----------------|-----------|
| Resend | 3K emails/mo | $20/mo (50K emails) | Per-email overages |
| SendGrid | 100 emails/day | $20/mo (50K emails) | Daily limit in free tier |
| AWS SES | 62K emails/mo (from EC2) | $0.10/1K emails | Very cheap at scale |

### LLM APIs

| Provider | Free/Credit Tier | Pricing | Cost Cliff |
|----------|-----------------|---------|-----------|
| OpenAI (GPT-4o) | $5 free credit | ~$2.50/1M input, $10/1M output | Per-token, scales with usage |
| Anthropic (Claude) | Pay-as-you-go | ~$3/1M input, $15/1M output | Per-token, scales with usage |
| Google (Gemini) | Free tier (15 RPM) | ~$1.25/1M input, $5/1M output | Rate limits in free tier |

*Note: LLM pricing changes frequently. Always verify current rates.*

## Cost Estimation Heuristics

### Usage Multipliers

| Metric | Conservative | Moderate | Aggressive |
|--------|-------------|----------|-----------|
| API calls per user per day | 5 | 15 | 50 |
| Pages/screens per session | 3 | 8 | 20 |
| Sessions per user per month | 10 | 20 | 30 |
| Storage per user | 10 MB | 50 MB | 500 MB |
| Emails per user per month | 2 | 5 | 15 |
| LLM tokens per interaction | 500 | 1,500 | 5,000 |

### Cost Scaling Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| **Linear** | Cost grows proportionally with users | Storage, bandwidth |
| **Step function** | Free until threshold, then jumps | Database tier upgrades |
| **Sublinear** | Cost grows slower than users | CDN (caching benefits) |
| **Superlinear** | Cost grows faster than users | LLM costs (more users = more tokens = more $) |

## Cost Optimization Strategies

| Strategy | Effort | Typical Savings | When to Use |
|----------|--------|----------------|------------|
| Response caching (Redis/CDN) | Medium | 20-60% on API costs | Repeated identical requests |
| LLM prompt optimization | Low | 20-40% on LLM costs | Long prompts, verbose responses |
| Smaller LLM for simple tasks | Low | 50-80% on LLM costs | Classification, extraction, formatting |
| Database query optimization | Medium | 10-30% on DB costs | High read volumes, missing indexes |
| Image/asset CDN | Low | 30-50% on bandwidth | Image-heavy applications |
| Serverless over always-on | Low | 50-90% at low scale | < 1K users with bursty traffic |
| Reserved instances | Low | 30-50% on compute | Predictable, steady workloads |
| Batch processing | Medium | 20-40% on API costs | Non-real-time operations |

## Solo Dev Budget Benchmarks

| Stage | Monthly Budget | What You Can Run |
|-------|---------------|-----------------|
| Side project | $0 (free tiers) | 100-500 users typically |
| Early startup | $50-$100/mo | 500-2K users |
| Growing | $200-$500/mo | 2K-10K users |
| Scaling | $1K-$5K/mo | 10K-50K users |
| Established | $5K+/mo | 50K+ users |
