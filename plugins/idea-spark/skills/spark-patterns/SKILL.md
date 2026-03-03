---
name: spark-patterns
description: Frameworks and techniques for validating business ideas, including Lean Canvas, competitive moat analysis, and go/no-go decision criteria. Use when evaluating startup ideas or refining business cases.
---

# Spark Patterns

Frameworks and techniques for validating business ideas quickly and rigorously.

## When to Use This Skill

- Running the `/idea-spark:spark` command to validate a new business idea
- Filling out a Lean Canvas for quick idea screening
- Evaluating competitive moats for a solo developer product
- Making a go/no-go decision on whether to build something

## Lean Canvas (Quick Validation)

Use this framework to quickly test whether an idea has all the essential components:

| Box | Question | Red Flag If Missing |
|-----|----------|-------------------|
| **Problem** | What are the top 3 problems you're solving? | No clear pain point |
| **Customer Segments** | Who has this problem? Be specific. | "Everyone" is not a segment |
| **Unique Value Proposition** | What makes you different and worth buying? | Can't articulate in one sentence |
| **Solution** | What are you building to solve the problem? | Solution looking for a problem |
| **Channels** | How will customers find you? | No organic discovery path |
| **Revenue Streams** | How will you make money? | No willingness to pay |
| **Cost Structure** | What are your fixed and variable costs? | Costs exceed realistic revenue |
| **Key Metrics** | What numbers define success? | No measurable outcomes |
| **Unfair Advantage** | What can't be easily copied or bought? | Nothing defensible |

## Competitive Moat Analysis

Moat types ranked by strength for solo developers:

| Moat Type | Solo Dev Strength | How to Build It |
|-----------|------------------|-----------------|
| Network effects | Strong | Build community features, user-generated content, integrations |
| Switching costs | Strong | Deep workflow integration, data lock-in, learning curve investment |
| Niche expertise | Strong | Domain-specific knowledge baked into product, proprietary datasets |
| Brand/Trust | Medium | Consistent content, open source contributions, community presence |
| Speed/Iteration | Medium | Ship faster than funded competitors, tight feedback loops |
| Scale economies | Weak | Hard for solo devs — requires volume that takes time to build |
| Patents/IP | Weak | Expensive, slow, and hard to enforce as a solo dev |

## Go/No-Go Decision Criteria

| Criterion | GO | CAUTION | NO-GO |
|-----------|----|---------|-------|
| **Market size (SOM)** | >$10M addressable | $1M-$10M addressable | <$1M addressable |
| **Competition** | Clear gap or underserved niche | Crowded but differentiable | Dominated by well-funded incumbents |
| **Feasibility** | MVP in <8 weeks solo | MVP in 8-16 weeks solo | MVP >16 weeks or needs team |
| **LTV:CAC** | >5:1 projected | 3:1-5:1 projected | <3:1 projected |
| **Founder fit** | Deep domain expertise | Adjacent experience | No relevant experience |

## Common Idea Anti-Patterns

Recognize these patterns early to avoid wasted effort:

1. **Solution looking for a problem** — You built something cool but can't find anyone who needs it. Fix: Start with customer interviews, not code.
2. **Boiling the ocean** — The idea requires solving 5 hard problems simultaneously. Fix: Find the smallest version that delivers value and start there.
3. **Feature not product** — The idea is a feature that an existing platform will inevitably add. Fix: Build on top of platforms instead of competing with them, or find a niche they won't serve.
4. **Vitamin not painkiller** — Nice to have, but nobody's hair is on fire. Fix: Find the customers for whom this IS a painkiller, or pivot to their actual pain.
5. **Timing mismatch** — The market isn't ready (too early) or the window has closed (too late). Fix: Validate timing signals — look for recent changes in technology, regulation, or behavior that create the opening.
