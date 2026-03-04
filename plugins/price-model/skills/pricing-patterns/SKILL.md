---
name: pricing-patterns
description: Pricing strategy frameworks, freemium conversion patterns, usage-based pricing models, price anchoring techniques, and tier design principles. Use when developing or reviewing pricing strategy.
---

# Pricing Patterns

Frameworks for developing effective pricing strategies.

## When to Use This Skill

- Running the `/pricing` command
- Designing pricing for a new product
- Optimizing existing pricing
- Analyzing competitor pricing

## Pricing Model Decision Tree

```
Is your product collaborative (value increases with more users)?
  YES → Per-seat pricing (Slack, Notion, GitHub)
  NO → Does usage vary significantly between customers?
    YES → Usage-based pricing (Twilio, AWS, OpenAI)
    NO → Is there a clear "power user" vs "casual user" split?
      YES → Feature-based tiers (most SaaS)
      NO → Flat-rate pricing (Basecamp)
```

## Pricing Models

| Model | Best For | Pros | Cons |
|-------|---------|------|------|
| Per seat | Collaboration tools | Predictable revenue, scales with org | Discourages adoption |
| Usage-based | APIs, infrastructure | Aligns with value | Revenue unpredictable |
| Feature-based | SaaS products | Clear upgrade path | Feature gate friction |
| Flat rate | Simple tools | Easy to understand | Leaves money on table |
| Freemium | Consumer/prosumer | Viral growth | Conversion challenge |
| Transaction | Marketplaces, fintech | Zero-risk for customer | Revenue tied to volume |
| Hybrid | Complex products | Best of multiple models | Complexity |

## Tier Design Principles

### The 3-Tier Structure

```
                Enterprise (Custom)
               /  High-touch sales, custom features
              /
    Pro ($X/mo)
   /  Power users, teams, advanced features
  /
Starter ($Y/mo)
  Individual users, core features

Free (optional)
  Hook, limited functionality
```

### What to Gate by Tier

| Gate Type | Examples | User Reaction |
|-----------|---------|---------------|
| Usage limits | 100 API calls/mo free, 10K paid | Fair -- "I pay for what I use" |
| Feature access | Advanced analytics in Pro | Acceptable -- "I pay for power" |
| Support level | Community vs. priority vs. dedicated | Expected -- "I pay for help" |
| Integrations | Basic in Starter, all in Pro | Acceptable for B2B |
| Team features | Solo free, team paid | Natural for collaboration tools |

### What NOT to Gate

| Never Gate | Why |
|-----------|-----|
| Security features | Ethical and legal liability |
| Data export | Creates vendor lock-in perception |
| Bug fixes | Punishes customers for your bugs |
| Basic API access | Kills developer ecosystem |

## Psychological Pricing Tactics

| Tactic | Example | Effect |
|--------|---------|--------|
| Charm pricing | $29 not $30 | 8-10% higher conversion |
| Price anchoring | Show Enterprise first | Makes Pro feel reasonable |
| Decoy pricing | 3 tiers where middle is best value | Steers to target tier |
| Annual discount | 20% off annual billing | Increases retention, cash flow |
| Social proof | "Most popular" badge | Reduces decision anxiety |

## Freemium Conversion Benchmarks

| Product Type | Free-to-Paid Rate | Benchmark |
|-------------|-------------------|-----------|
| Developer tools | 2-5% | Good if > 3% |
| B2B SaaS | 3-7% | Good if > 5% |
| Consumer apps | 1-4% | Good if > 2% |
| Productivity tools | 2-5% | Good if > 3% |

## Value-Based Pricing Formula

```
Your Price = (Customer's Current Cost of Problem) x (0.1 to 0.3)

Example:
  Problem costs customer $1,000/month in wasted time
  Your price: $100-300/month (saves them 70-90%)
  Customer perception: Great deal
  Your margin: High (if costs are low)
```

## Common Pricing Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Pricing too low | Signals low value, hard to raise | Start higher, discount down |
| Too many tiers | Decision paralysis | 3-4 tiers maximum |
| Complex pricing | Customers can't calculate cost | One value metric, simple math |
| Free tier too generous | No reason to upgrade | Gate the "aha moment" |
| No annual option | High churn, poor cash flow | 15-20% annual discount |
| Pricing page hidden | Signals expensive/opaque | Show pricing prominently |
| Enterprise = Free + more | No upgrade path | Each tier adds clear value |

## Price Change Communication

When raising prices:

1. **Grandfather existing customers** for 6-12 months
2. **Communicate the value added** since the last price
3. **Give 30+ days notice** before the change takes effect
4. **Offer annual lock-in** at the old price
5. **Never apologize** for charging for value -- frame as investment in the product
