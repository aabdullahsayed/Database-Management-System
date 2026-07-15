# Key Value

## Scenario
You need a session store: `session_token -> { user_id, expires_at, permissions }`, accessed millions of times per second, always by exact token, with sub-millisecond latency requirements.

## Diagram
```
Key-Value store (e.g. Redis):
"sess:abc123" -> {"user_id": 5, "expires_at": "2026-07-16T00:00:00Z"}
"sess:xyz789" -> {"user_id": 9, "expires_at": "2026-07-16T02:00:00Z"}
   GET "sess:abc123"  -> O(1) in-memory lookup, no query planning needed
```

## Problem
Why is a relational database a worse fit than a key-value store for this specific use case, even though it *could* technically do the job?

## Solution
A relational database gives you rich query capability (joins, filtering, transactions, ordering) that this use case doesn't need at all - every access is a pure "exact key in, exact value out" operation. That extra machinery (query parsing, planning, ACID transaction overhead, disk-based durability by default) adds latency you don't need to pay for. A key-value store like Redis is purpose-built for exactly this pattern: in-memory by default, minimal per-request overhead, and horizontally shardable by key with no coordination needed between unrelated keys.

## Takeaway
Key-value stores excel when your access pattern is purely "exact key lookup, no relationships, no complex queries" - using a full RDBMS for this is technically possible but pays for capabilities (joins, complex transactions) you're never using.
