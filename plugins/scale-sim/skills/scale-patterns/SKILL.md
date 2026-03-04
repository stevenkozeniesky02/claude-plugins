---
name: scale-patterns
description: Scaling strategy frameworks, capacity planning heuristics, common bottleneck patterns, and performance optimization techniques for growing applications. Use when analyzing scaling readiness or planning capacity improvements.
---

# Scale Patterns

Frameworks for analyzing and improving application scalability.

## When to Use This Skill

- Running the `/simulate` command
- Planning for traffic growth
- Investigating performance degradation
- Designing systems for scale

## The Scaling Hierarchy

Fix bottlenecks in this order (cheapest and most effective first):

```
1. Code optimization (free)
   ├── Add missing indexes
   ├── Fix N+1 queries
   ├── Add LIMIT to queries
   └── Remove unnecessary computation

2. Configuration tuning (free)
   ├── Increase connection pool size
   ├── Tune cache TTLs
   ├── Adjust worker concurrency
   └── Set appropriate timeouts

3. Caching (low cost)
   ├── Application-level caching
   ├── CDN for static assets
   ├── Database query caching
   └── API response caching

4. Async processing (medium cost)
   ├── Background job queues
   ├── Event-driven architecture
   ├── Webhook processing
   └── Batch operations

5. Horizontal scaling (higher cost)
   ├── Stateless application servers
   ├── Read replicas
   ├── Load balancing
   └── Distributed caching

6. Architecture changes (highest cost)
   ├── Database sharding
   ├── Microservices decomposition
   ├── CQRS/Event sourcing
   └── Edge computing
```

## Common Bottleneck Patterns

### Database Bottlenecks

| Pattern | Symptom | Scale Impact | Fix |
|---------|---------|-------------|-----|
| N+1 queries | Linear query growth with data | O(N) -> O(N*M) | Eager loading, JOINs |
| Missing index | Slow queries on filtered columns | Full table scan | Add index |
| Unbounded query | `SELECT *` without LIMIT | OOM at scale | Add pagination |
| Write hotspot | Lock contention on popular rows | Serialized writes | Partition/shard |
| Fat rows | Large text/blob in frequently read rows | Memory/bandwidth | Normalize, lazy load |
| Connection exhaustion | "Too many connections" | Hard ceiling | Pool sizing, pgBouncer |

### Application Bottlenecks

| Pattern | Symptom | Scale Impact | Fix |
|---------|---------|-------------|-----|
| Synchronous I/O | Request blocks on external call | Thread starvation | Async/background job |
| In-memory state | Session/cache in process memory | Can't add servers | External store (Redis) |
| Global locks | Mutex/semaphore contention | Serialized execution | Lock-free or distributed lock |
| Large payload | Multi-MB responses | Bandwidth saturation | Pagination, compression |
| CPU-bound handler | Heavy computation in request | Thread saturation | Background job, cache result |

### Infrastructure Bottlenecks

| Pattern | Symptom | Scale Impact | Fix |
|---------|---------|-------------|-----|
| Single server | No redundancy | Hard ceiling | Load balancer + instances |
| No CDN | Static files from app server | Bandwidth waste | CDN (Cloudflare free tier) |
| No caching layer | Every request hits DB | DB overload | Redis/Memcached |
| Single-region | High latency for distant users | Poor global UX | Multi-region or edge |

## Capacity Planning Formulas

### Concurrent Users Estimation
```
Peak concurrent = Daily active users x 0.1  (10% rule)
Requests/sec = Peak concurrent x Requests per session / Avg session duration
```

### Database Connection Pool Sizing
```
Pool size = (Number of CPU cores x 2) + Number of disk spindles
For SSD: Pool size = CPU cores x 2 + 1
Rule of thumb: max_connections = num_app_instances x pool_size_per_instance
```

### Memory Estimation
```
Memory per instance = Base runtime + (Per-request memory x Max concurrent requests)
Total memory = Memory per instance x Number of instances
```

### Storage Growth
```
Monthly growth = New users/month x Storage per user + Log volume + Backup overhead
1-year projection = Current storage + (Monthly growth x 12) x 1.2  (20% buffer)
```

## Scale Tier Reference

| Tier | Users | Typical Architecture | Monthly Cost |
|------|-------|---------------------|-------------|
| Hobby | < 100 | Single server, SQLite/PG | $0-20 |
| Startup | 100-1K | Single server, managed DB | $20-100 |
| Growth | 1K-10K | Load balanced, Redis, CDN | $100-500 |
| Scale | 10K-100K | Multi-server, read replicas, queue | $500-5K |
| Enterprise | 100K+ | Microservices, sharding, multi-region | $5K+ |

## Horizontal Scaling Checklist

Before adding more servers, verify:

| Requirement | Check | Common Blocker |
|------------|-------|---------------|
| Stateless application | No in-memory sessions | Move sessions to Redis |
| External session store | Redis/DB sessions | Session in memory |
| Shared file storage | S3/GCS for uploads | Local filesystem |
| Database connections | Pool handles multiple instances | Pool too small |
| Background jobs | Distributed lock/queue | Cron on single server |
| Health check endpoint | /health returns 200 | No health check |
| Graceful shutdown | Handle SIGTERM | Drops connections |

## Performance Quick Reference

| Latency | Description | User Perception |
|---------|-------------|----------------|
| < 100ms | Instant | Feels immediate |
| 100-300ms | Fast | Noticeable but acceptable |
| 300ms-1s | Moderate | Feels slow |
| 1-3s | Slow | Frustrating |
| > 3s | Unacceptable | Users leave |

| Throughput | Description |
|-----------|-------------|
| 100 req/s | Single Node.js/Python server |
| 1K req/s | Optimized single server with caching |
| 10K req/s | Multiple servers with load balancer |
| 100K req/s | Distributed architecture with CDN |
