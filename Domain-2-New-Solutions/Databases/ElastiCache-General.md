# Amazon ElastiCache — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). ElastiCache is the in-memory caching layer. Know Redis vs Memcached, caching patterns, and when to use DAX instead.

---

## 1. Redis vs Memcached

| Feature | Redis | Memcached |
|---|---|---|
| Data structures | Strings, hashes, lists, sets, sorted sets, bitmaps, streams | Strings only |
| Persistence | Yes (RDB snapshots, AOF) | No |
| Replication | Yes (primary + replicas) | No |
| Multi-AZ | Yes (automatic failover) | No |
| Clustering | Yes (Redis Cluster — sharding) | Yes (multi-node) |
| Pub/Sub | Yes | No |
| Lua scripting | Yes | No |
| Geospatial | Yes | No |
| Use case | Sessions, leaderboards, pub/sub, complex data, HA | Simple caching, horizontal scaling, multi-threaded |

**Choose Redis** for almost all production use cases — it supports HA, persistence, and complex data structures.
**Choose Memcached** only when you need simple, multi-threaded caching with no persistence requirement.

---

## 2. ElastiCache for Redis Architecture

### Cluster Mode Disabled (Replication Group)

```
Primary Node (read/write)
├── Replica 1 (read-only, AZ-b)
└── Replica 2 (read-only, AZ-c)
```

- Single shard, up to 5 replicas
- Automatic failover promotes a replica to primary
- All data fits in one shard

### Cluster Mode Enabled (Redis Cluster)

```
Shard 1: Primary + 2 Replicas (key slots 0–5460)
Shard 2: Primary + 2 Replicas (key slots 5461–10922)
Shard 3: Primary + 2 Replicas (key slots 10923–16383)
```

- Data partitioned across multiple shards (up to 500 shards)
- Scales horizontally for large datasets
- Requires Redis Cluster-aware client

---

## 3. Caching Patterns

### Lazy Loading (Cache-Aside)

```
Application → Cache hit? → Return cached data
           → Cache miss? → Query database → Store in cache → Return data
```

- Only requested data is cached
- Cache miss adds latency (2 round trips)
- Stale data possible if DB updates don't invalidate cache

### Write-Through

```
Application → Write to cache AND database simultaneously
```

- Cache always up-to-date
- Write penalty (2 writes per operation)
- Cache may contain data never read (wasted memory)

### Write-Behind (Write-Back)

```
Application → Write to cache → Async write to database
```

- Low write latency
- Risk of data loss if cache fails before DB write

### TTL (Time to Live)

- Set expiry on cached items to prevent stale data
- Balance between freshness and cache hit rate

---

## 4. Session Store Pattern

Store user sessions in ElastiCache Redis instead of application servers:

```
User Request → ALB → Any EC2 instance → ElastiCache (session lookup)
```

- Enables stateless application tier
- Sessions survive instance failures and scale-out events
- Use Redis with replication for HA session storage

---

## 5. Security

- **VPC:** deploy in private subnets; use security groups
- **Encryption at rest:** KMS
- **Encryption in transit:** TLS (Redis 6.x+)
- **Auth:** Redis AUTH token or IAM authentication (Redis 7.x+)
- **RBAC:** Redis 6.x+ supports role-based access control

---

## 6. ElastiCache vs DAX

| Feature | ElastiCache | DAX |
|---|---|---|
| Database | Any (RDS, custom) | DynamoDB only |
| Protocol | Redis/Memcached | DynamoDB API |
| Code changes | Yes (use Redis/Memcached client) | Minimal (swap endpoint) |
| Data structures | Rich (Redis) | Key-value |
| Use case | General caching, sessions, pub/sub | DynamoDB read acceleration |

---

## 7. Exam Scenarios

**Scenario 1:** A web application stores user sessions on EC2 instances. When instances are replaced during Auto Scaling, users are logged out. What is the solution?

> Store sessions in **ElastiCache Redis**. The application tier becomes stateless — any instance can serve any user by looking up the session in Redis. Sessions survive instance replacements.

**Scenario 2:** A database is experiencing high read load from repeated identical queries. What is the caching strategy?

> Implement **lazy loading** with ElastiCache Redis. On a cache miss, the application queries the database, stores the result in Redis with a TTL, and returns the data. Subsequent identical queries are served from cache.

**Scenario 3:** A company needs a leaderboard for a gaming application that updates in real-time and returns the top 100 players. What is the solution?

> Use **ElastiCache Redis Sorted Sets**. Each player's score is stored as a sorted set member. Redis `ZADD` updates scores and `ZREVRANGE` retrieves the top N players in O(log N) time.

---

## 8. Exam Tips

- **Redis** = HA, persistence, complex data structures — use for almost everything
- **Memcached** = simple, multi-threaded, no persistence — rarely the right answer
- **Cluster Mode Enabled** = horizontal sharding for large datasets
- **Lazy loading** = cache only what's requested; **write-through** = always up-to-date
- ElastiCache is for **any database**; **DAX** is DynamoDB-only
- Session storage in ElastiCache = stateless application tier = Auto Scaling friendly
- Redis Sorted Sets = leaderboards; Redis Pub/Sub = real-time messaging

---

## 9. Official References

- [ElastiCache for Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/)
- [ElastiCache for Memcached](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/)
- [Caching Strategies](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Strategies.html)
