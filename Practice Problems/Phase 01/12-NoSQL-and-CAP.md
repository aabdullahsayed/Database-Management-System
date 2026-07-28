# 12. NoSQL, CAP Theorem & Distributed Databases

## Quick Refresher
- **NoSQL** = "Not Only SQL" — non-relational databases designed for flexible schemas and horizontal scaling.
- Main types: **Key-Value** (Redis, DynamoDB), **Document** (MongoDB), **Column-family** (Cassandra, HBase), **Graph** (Neo4j).
- **CAP Theorem**: in a distributed system experiencing a network partition, you must choose between **Consistency** and **Availability** — you can't have both perfectly at the same time.
- **Eventual Consistency**: replicas will *eventually* converge to the same value, but might briefly disagree.

## Practice Problems

### Q1 (Basic). What's the main trade-off SQL databases make that NoSQL databases often relax?
**Answer:** Traditional relational (SQL) databases prioritize strong consistency and strict schema/ACID guarantees, often at the cost of horizontal scalability. Many NoSQL databases relax consistency (favoring "eventual consistency") or schema rigidity in exchange for easier horizontal scaling and higher availability.

### Q2 (Basic). Give one example use case each for Key-Value, Document, and Graph databases.
**Answer:**
- **Key-Value** (Redis): session storage, caching — fast lookups by a single key.
- **Document** (MongoDB): product catalogs with varying attributes per item — flexible, semi-structured data.
- **Graph** (Neo4j): social networks, recommendation engines — data naturally modeled as nodes and relationships.

### Q3 (Intermediate). Explain the CAP theorem in plain English, and why "pick 2 of 3" is a bit of an oversimplification.
**Answer:** In a distributed system, if the network between nodes fails (a **P**artition — which *will* happen eventually), you must choose: respond anyway with possibly stale data (favor **A**vailability), or refuse to respond until you're sure the data is current (favor **C**onsistency). It's an oversimplification because **Partition tolerance isn't really optional** — real distributed systems must handle partitions, so the actual choice in practice is CP vs. AP *during* a partition; when there's no partition, you can often have both C and A.

### Q4 (Intermediate). What is "eventual consistency," and what kind of application can tolerate it well? Give an example where it would be a bad fit.
**Answer:** Eventual consistency means that after writes stop, all replicas will converge to the same value — but a read immediately after a write might return stale data. Good fit: a social media "like" counter (briefly showing 999 vs 1000 likes doesn't matter). Bad fit: a bank account balance check right before approving a large withdrawal — reading stale (too-high) data could let someone overdraw.

### Q5 (Intermediate). What is sharding, and what's a common pitfall when choosing a shard key?
**Answer:** Sharding splits a large dataset across multiple servers, each holding a subset ("shard") of the data, based on a **shard key**. Common pitfall: choosing a shard key that creates a **"hot shard"** — e.g., sharding by `created_date` means all of *today's* writes hit a single shard, overloading it while older shards sit idle. Better: choose a shard key with high, evenly-distributed cardinality (e.g., a hashed user ID).

### Q6 (Advanced/Interview). Compare a relational database and a document database (like MongoDB) for an e-commerce product catalog where different product categories have very different attributes (e.g., a shirt has "size" and "color"; a laptop has "RAM" and "CPU"). What are the trade-offs of each?
**Answer:**
- **Relational**: you'd need either a very wide table with many nullable columns (messy), or an EAV (Entity-Attribute-Value) pattern / separate per-category tables (more normalized but more complex joins). Strong consistency and mature querying/reporting tools.
- **Document**: each product document can simply have whatever fields are relevant to its category — no schema migration needed to add new product types. Trade-off: harder to enforce consistent structure across documents, and cross-document "join-like" queries (e.g., "find all products from suppliers in a certain region") are less natural/efficient than in SQL.
- **Practical answer**: many real systems use a **hybrid** — relational for orders/payments/inventory (needs strong consistency, well-defined structure), document store for the flexible product catalog itself.

### Q7 (Advanced/Interview). In a leader-follower (primary-replica) replicated database, explain the trade-off between synchronous and asynchronous replication, in terms of CAP-theorem-style reasoning.
**Answer:**
- **Synchronous replication**: the primary waits for the replica(s) to confirm the write before acknowledging success to the client — guarantees the replica is always up to date (favors Consistency), but write latency increases (and availability suffers if a replica is slow/unreachable — the primary might have to block or reject writes).
- **Asynchronous replication**: the primary acknowledges the write immediately and sends it to replicas in the background — favors Availability and low latency, but a replica might lag behind, and if the primary crashes before replicating a write, that write **could be lost** on failover (a consistency risk).
- Many production systems use a middle ground: synchronous replication to at least one "quorum" of replicas (not all), balancing durability/consistency against latency.

### Q8 (Advanced/Interview). Why might a well-designed system use BOTH a SQL database and a NoSQL database together (polyglot persistence)? Give a concrete example architecture.
**Answer:** Different parts of a system have different consistency/scale/query needs, and no single database is optimal for everything.
**Example**: An e-commerce platform might use:
- **PostgreSQL** for orders, payments, and inventory — needs strong ACID guarantees (can't double-sell the last item, must never lose a payment record).
- **Redis** (key-value) for session data and caching product pages — needs blazing-fast reads, data is disposable/short-lived.
- **Elasticsearch** for product search — needs powerful full-text search and fuzzy matching that SQL handles poorly.
- **Cassandra** for storing high-volume clickstream/analytics events — needs to absorb massive write throughput across many servers, and slight eventual-consistency lag is fine for analytics.
This "polyglot persistence" approach uses each database for what it's best at, at the cost of added architectural and operational complexity (keeping data in sync across systems, more infrastructure to maintain).
