---
name: runbook-patterns
description: Incident response frameworks, severity classification systems, runbook writing templates, and post-mortem best practices for operational readiness. Use when generating runbooks or planning incident response.
---

# Runbook Patterns

Frameworks for building effective incident response capabilities.

## When to Use This Skill

- Running the `/runbook` command
- Writing operational runbooks manually
- Setting up an incident response process
- Planning monitoring and alerting

## Severity Classification Framework

| Level | Name | User Impact | Data Impact | Response |
|-------|------|-----------|-------------|----------|
| SEV1 | Critical | Complete outage for all users | Data loss or corruption possible | All hands, immediate response |
| SEV2 | High | Major feature unavailable | No data loss, degraded experience | On-call responds within 30 min |
| SEV3 | Medium | Minor feature degraded | No data impact | Fix during business hours |
| SEV4 | Low | Cosmetic or edge case | None | Schedule for next sprint |

### Auto-Classification Rules

| Symptom | Auto-Severity |
|---------|--------------|
| API returning 500 to all users | SEV1 |
| Database unreachable | SEV1 |
| Authentication broken | SEV1 |
| Payment processing failing | SEV1 |
| Single endpoint returning errors | SEV2 |
| Background job queue backed up | SEV2 |
| Search feature degraded | SEV3 |
| Slow response times (< 2x normal) | SEV3 |
| UI rendering issue in one browser | SEV4 |

## Runbook Structure Template

Every runbook should follow this structure:

```
1. TITLE + METADATA
   - Severity, impact, estimated resolution time
   - Last verified date

2. SYMPTOMS
   - What the operator will see/hear about
   - Specific error messages or alert names

3. DIAGNOSIS
   - Step-by-step verification commands
   - Expected output for each step
   - Decision tree: "If X, go to step Y"

4. MITIGATION (Stop the Bleeding)
   - Immediate actions to reduce user impact
   - May be a workaround, not a fix

5. RECOVERY (Fix Root Cause)
   - Steps to fully resolve the issue
   - Specific commands with expected output

6. VERIFICATION
   - How to confirm the fix worked
   - What "normal" looks like

7. POST-INCIDENT
   - Checklist of follow-up actions
   - When to schedule post-mortem
```

## Common Failure Mode Categories

### Infrastructure Failures

| Failure | Detection | Typical Mitigation |
|---------|-----------|-------------------|
| Database connection exhaustion | Connection pool errors in logs | Increase pool size, kill idle connections |
| Disk space full | Disk utilization alert | Clear logs, expand volume |
| Memory exhaustion (OOM) | OOM killer in system logs | Restart service, investigate leak |
| Certificate expiry | TLS handshake failures | Renew certificate |
| DNS resolution failure | Connection timeout to known hosts | Check DNS config, use IP fallback |

### Application Failures

| Failure | Detection | Typical Mitigation |
|---------|-----------|-------------------|
| Unhandled exception crash loop | Service restart frequency | Fix exception, add handler |
| Deadlock | Requests hanging, no errors | Kill blocked transactions |
| Memory leak | Gradually increasing memory usage | Restart service (temporary) |
| Configuration error | Service won't start | Check env vars, revert config |
| Dependency version mismatch | Import errors on startup | Pin versions, rebuild |

### External Dependency Failures

| Failure | Detection | Typical Mitigation |
|---------|-----------|-------------------|
| Third-party API down | HTTP errors to external service | Activate fallback/cache |
| Rate limited by external API | 429 responses | Implement backoff, check quotas |
| Payment provider outage | Payment failures | Queue payments for retry |
| Email service down | Delivery failures | Queue emails, switch provider |

## Post-Mortem Template

```markdown
# Post-Mortem: [Incident Title]

**Date:** [YYYY-MM-DD]
**Duration:** [start time] to [end time] ([total duration])
**Severity:** [SEV level]
**Author:** [name]

## Summary
[2-3 sentence summary of what happened]

## Impact
- Users affected: [N or percentage]
- Revenue impact: [if applicable]
- Data impact: [any data loss or corruption]

## Timeline
| Time | Event |
|------|-------|
| HH:MM | [First symptom detected] |
| HH:MM | [Incident declared] |
| HH:MM | [Root cause identified] |
| HH:MM | [Mitigation applied] |
| HH:MM | [Fully resolved] |

## Root Cause
[Detailed explanation of what went wrong and why]

## What Went Well
- [Thing that helped during response]

## What Went Wrong
- [Thing that made response harder]

## Action Items
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| [Preventive fix] | [name] | [date] | Open |
| [Detection improvement] | [name] | [date] | Open |
| [Runbook update] | [name] | [date] | Open |
```

## Monitoring Essentials for Solo Devs

| What to Monitor | Tool Options | Alert When |
|----------------|-------------|-----------|
| Uptime | UptimeRobot, Pingdom (free tier) | Site down for > 1 min |
| Error rate | Sentry (free tier), LogRocket | Error rate > 1% |
| Response time | Built-in APM, Vercel Analytics | P95 > 2s |
| Database connections | Connection pool metrics | > 80% pool utilization |
| Disk usage | Cloud provider alerts | > 85% disk full |
| Certificate expiry | cert-manager, manual check | < 14 days to expiry |
| Dependency CVEs | Dependabot, Snyk | Any critical/high CVE |
