---
name: legal-patterns
description: GDPR/CCPA compliance requirements, privacy policy structure, terms of service essentials, cookie consent rules, and compliance checklist frameworks. Use when generating compliance documentation.
---

# Legal Compliance Patterns

Frameworks for generating compliance documentation from codebase analysis.

## When to Use This Skill

- Running the `/comply` command
- Drafting privacy policies or terms of service
- Assessing GDPR or CCPA compliance
- Auditing data handling practices

## GDPR Key Requirements

| Requirement | Article | What It Means |
|------------|---------|---------------|
| Lawful basis | Art. 6 | Must have legal basis for each data processing activity |
| Consent | Art. 7 | Must be freely given, specific, informed, unambiguous |
| Transparency | Art. 13-14 | Must inform users what data you collect and why |
| Data minimization | Art. 5(1)(c) | Only collect data you actually need |
| Purpose limitation | Art. 5(1)(b) | Only use data for stated purposes |
| Storage limitation | Art. 5(1)(e) | Don't keep data longer than necessary |
| Right of access | Art. 15 | Users can request copy of their data |
| Right to erasure | Art. 17 | Users can request deletion ("right to be forgotten") |
| Right to portability | Art. 20 | Users can export data in machine-readable format |
| Breach notification | Art. 33-34 | Must notify authority within 72 hours of breach |
| DPA required | Art. 28 | Need Data Processing Agreements with all processors |
| DPIA for high risk | Art. 35 | Data Protection Impact Assessment for risky processing |

## CCPA Key Requirements

| Requirement | Section | What It Means |
|------------|---------|---------------|
| Right to know | 1798.100 | Consumers can request what PI you collect |
| Right to delete | 1798.105 | Consumers can request deletion of PI |
| Right to opt-out | 1798.120 | "Do Not Sell My Personal Information" link required |
| Right to non-discrimination | 1798.125 | Can't penalize users who exercise rights |
| Notice at collection | 1798.100(b) | Must disclose categories of PI at point of collection |
| Financial incentives | 1798.125(b) | Must disclose value exchange for data collection |
| Service provider contracts | 1798.140(v) | Contracts must restrict how providers use data |

## CCPA Applicability Thresholds

CCPA applies if the business meets ANY of:
- Annual gross revenue > $25 million
- Buys/sells/shares PI of 100,000+ consumers/households
- Derives 50%+ of annual revenue from selling consumers' PI

## Legal Basis Decision Tree

```
Is data processing necessary for the contract?
  └── YES → Legal basis: Contract performance (Art. 6(1)(b))
  └── NO ↓

Is there a legal obligation to process this data?
  └── YES → Legal basis: Legal obligation (Art. 6(1)(c))
  └── NO ↓

Is there a legitimate business interest?
  └── YES → Does it override user's rights?
        └── NO → Legal basis: Legitimate interest (Art. 6(1)(f))
        └── YES ↓
  └── NO ↓

Has the user given explicit consent?
  └── YES → Legal basis: Consent (Art. 6(1)(a))
  └── NO → No legal basis — DO NOT process
```

## Privacy Policy Required Sections

| Section | GDPR Required | CCPA Required | Content |
|---------|:---:|:---:|---------|
| Identity and contact | Yes | Yes | Who you are, how to reach you |
| DPO contact | Yes (if applicable) | No | Data Protection Officer details |
| Data collected | Yes | Yes | What personal data you collect |
| Purposes | Yes | Yes | Why you collect each data type |
| Legal basis | Yes | No | GDPR legal basis per purpose |
| Recipients/third parties | Yes | Yes | Who you share data with |
| International transfers | Yes | No | Cross-border data transfers |
| Retention periods | Yes | No | How long you keep data |
| User rights | Yes | Yes | What rights users have |
| Consent withdrawal | Yes | No | How to withdraw consent |
| Right to complain | Yes | No | How to file complaints |
| Automated decisions | Yes (if applicable) | No | Profiling and automated decisions |
| Cookie information | Yes | No | How cookies are used |
| Do Not Sell link | No | Yes | Opt-out of data selling |
| Financial incentives | No | Yes (if applicable) | Value exchange disclosure |
| Categories sold/disclosed | No | Yes | 12-month lookback |

## Cookie Consent Rules

### GDPR (EU/EEA)

| Cookie Type | Consent Required? | Example |
|-------------|:-:|---------|
| Strictly necessary | No | Session cookies, CSRF tokens, load balancing |
| Functional | Yes | Language preference, theme, saved settings |
| Analytics | Yes | Google Analytics, Mixpanel, Hotjar |
| Marketing | Yes | Ad trackers, retargeting pixels, social media |

**Rules:**
- Consent must be opt-IN (no pre-checked boxes)
- Must be able to accept/reject each category separately
- Must work without non-essential cookies if user rejects
- Must provide easy withdrawal mechanism
- Must store proof of consent

### CCPA (California)

- No opt-in consent required for cookies per se
- "Do Not Sell" opt-out required if cookies share data with third parties
- "Sharing" under CPRA includes targeted advertising

## Terms of Service Essential Clauses

| Clause | Purpose | Key Points |
|--------|---------|------------|
| Acceptance | How users agree | Click-wrap preferred over browse-wrap |
| Service description | What you provide | Match actual capabilities |
| User accounts | Registration rules | Age requirements, accuracy, security |
| Acceptable use | What's prohibited | Abuse, illegal activity, reverse engineering |
| Intellectual property | Ownership | Your IP, user content licensing |
| User content | Rights to user data | License grant, responsibility |
| Payment terms | If applicable | Pricing, refunds, auto-renewal |
| Termination | When/how accounts end | For cause, convenience, data after |
| Disclaimer | Warranties | "As is" language |
| Limitation of liability | Cap on damages | Monetary cap, exclusions |
| Indemnification | User responsibility | User holds you harmless |
| Governing law | Jurisdiction | Choice of law, dispute resolution |
| Changes | How terms update | Notice period, material changes |
| Severability | Invalid clauses | Rest of agreement survives |
| Entire agreement | Completeness | Supersedes prior agreements |

## Compliance Checklist Framework

### Pre-Launch (Critical)

- [ ] Privacy policy published and accessible from every page
- [ ] Terms of service published with acceptance mechanism
- [ ] Cookie consent banner implemented (if serving EU users)
- [ ] "Do Not Sell" link visible (if CCPA applies)
- [ ] Data Processing Agreements signed with all third-party processors
- [ ] SSL/TLS encryption on all pages handling personal data
- [ ] Password hashing implemented (bcrypt/argon2, NOT MD5/SHA1)

### Post-Launch (30 Days)

- [ ] Data subject request process documented and tested
- [ ] Account deletion flow implemented and tested
- [ ] Data export functionality available
- [ ] Breach notification procedure documented
- [ ] Records of Processing Activities (ROPA) created
- [ ] Employee data handling training completed

### Ongoing (Quarterly)

- [ ] Privacy policy reviewed for accuracy
- [ ] New third-party integrations assessed for compliance
- [ ] Cookie audit performed (new cookies cataloged)
- [ ] Data retention review (delete expired data)
- [ ] Access logs reviewed for unauthorized access

## Common Compliance Mistakes

| Mistake | Why It's a Problem | Fix |
|---------|-------------------|-----|
| Generic privacy policy | Doesn't reflect actual practices | Generate from codebase analysis |
| Missing third parties | Incomplete disclosure | Audit all API calls and SDKs |
| No cookie consent | GDPR violation, fines up to 4% revenue | Implement consent banner |
| Pre-checked consent | Invalid under GDPR | Switch to opt-in |
| No deletion endpoint | Violates right to erasure | Implement account deletion |
| PII in server logs | Uncontrolled data processing | Sanitize logs |
| Indefinite retention | Violates storage limitation | Define and enforce TTLs |
| No breach plan | 72-hour notification deadline will be missed | Document response procedure |
