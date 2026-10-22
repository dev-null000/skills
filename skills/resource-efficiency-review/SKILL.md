---
name: resource-efficiency-review
description: Conducts an evidence-based bare-metal technical review focused strictly on physical resource usage, connection resilience, saturation limits, and leak prevention. Use when asked to review or audit code for resource efficiency, performance bottlenecks, database query scaling (N+1, indexing), connection pooling, memory leaks, or bare-metal runtime limits.
---

This skill conducts a deep technical audit of code changes, abstracting entirely from aesthetic style, design patterns, naming choices, or framework preferences. Its sole focus is **physical resource consumption, connection resilience, saturation curves, and system exhaustion**.

When invoked, the agent must think as a systems engineer: trace physical boundaries (Memory $\to$ Sockets $\to$ Network $\to$ Database $\to$ Disk), consult driver and framework documentation for default pooling or buffering behaviors, simulate behavior under scale ($N \to \infty$, high concurrency, transient outages), and propose minimal, proportional remedies.

---

## The Proportionality Rule (Engineering vs. Over-Engineering)

Every finding and remediation must be calibrated to the application's actual scope and production profile:

1. **Context & Scale Calibration:**
   - Differentiate between a one-off CLI tool, an internal batch job, a low-traffic admin service (<50 RPS), and a high-concurrency public API (>1,000 RPS).
   - Do not propose streaming, chunking, or complex pagination for datasets bounded by domain rules to small sizes ($N < 100$).
   - Require strict chunking, streaming, and indexing when inputs can grow unbounded ($N > 10,000$).
2. **Native Primitives Over Custom Wheels:**
   - Always leverage built-in driver, ORM, or framework capabilities (e.g., driver pool flags, `pool_pre_ping`, framework lifespan hooks, standard library generators, context managers).
   - Never recommend custom background keeper threads, homebrew connection supervisors, or distributed retry meshes when standard configuration or basic idioms suffice.
3. **Simplicity Test:**
   - The proposed remediation must introduce fewer moving parts and failure modes than the issue it resolves.

---

## The 9 Physical Resource Pillars

Evaluate code systematically against these 9 pillars:

### 1. Connection Lifecycle & Failure Resilience
- **Pool Reuse vs. Churn:** Ensure database, Redis, and HTTP client connections are acquired from shared, pooled singletons. Reject creating new clients or connections per function or request (eliminates TCP handshakes, TLS negotiation, and `TIME_WAIT` socket exhaustion).
- **Lifespan Management:** Verify that connection pools, database engines, and shared HTTP sessions initialize during application bootstrap (e.g., FastAPI lifespan context, framework startup hooks) and gracefully drain on shutdown.
- **Stale Connection Recovery:** Ensure connection pools verify liveness before use (e.g., SQLAlchemy `pool_pre_ping=True`, Redis ping-on-connect) so dropped connections or cloud idle resets do not cause unhandled runtime crashes.
- **Connection Hoarding:** Forbid holding a checked-out database connection or active transaction across external HTTP calls, S3 uploads, or heavy CPU processing. Connections must be released before slow non-DB I/O begins.
- **Graceful Degradation:** Failure of non-critical auxiliary services (e.g., cache, telemetry, analytics) must degrade gracefully and not crash primary transactional workflows.

### 2. Database Query Patterns
- **N+1 Query Problem:** Forbid issuing queries inside loops over collections. Enforce bulk fetching via `JOIN`, eager loading (`select_related`, `prefetch_related`), or `IN (...)` batch queries.
- **Cartesian Product Joins:** Flag unbounded multi-table joins that multiply result set cardinality.
- **Unbounded Queries:** Enforce `LIMIT` or keyset pagination on all queries against growing tables; never allow unbounded `SELECT *` without row caps.

### 3. Database Indexing & Query Planning
- **Missing Filter & Join Indexes:** Ensure columns referenced in `WHERE`, `JOIN ON`, `ORDER BY`, or foreign keys have supporting indexes on tables expected to exceed trivial row counts.
- **Composite Index Alignment:** Verify multi-column queries match the left-to-right column order of existing composite indexes.
- **Type Coercion & Function Invalidation:** Flag queries where functions wrap indexed columns (e.g., `WHERE LOWER(email) = ...`) or type mismatches force full table scans.
- **Over-Indexing Guard:** Do not propose redundant indexes on high-throughput, write-heavy append tables where index maintenance penalties outweigh read benefits.

### 4. Network & RPC Efficiency
- **Loop Requests:** Flag iterative sequential HTTP or RPC calls inside loops. Require bulk endpoints or batched requests.
- **Session & Transport Reuse:** Ensure HTTP requests use a persistent client session with connection pooling and keep-alive rather than one-off client instantiations.
- **Timeouts & Circuit Breaking:** Every outbound network call must specify explicit connect and read timeouts. Retries must be bounded and use exponential backoff with jitter to prevent thundering herds.

### 5. Memory & Buffer Management
- **Streaming vs. In-Memory Buffering:** Forbid reading entire large files, HTTP payloads, or DB query results into memory at once (`file.read()`, `cursor.fetchall()`). Enforce chunked streaming, generators, or server-side cursors for payloads over a few megabytes.
- **Unbounded Collections & Caches:** Ensure in-memory caches, queues, and buffers have maximum sizes, TTLs, or LRU eviction policies to prevent memory leaks.
- **Payload Over-Fetching:** Restrict queries to required columns instead of wide table dumps when row sizes or counts are large.

### 6. Filesystem & Disk I/O
- **Inode Exhaustion:** Prevent writing unbounded files into a single flat directory; enforce partitioned directory trees (e.g., hash prefixes `ab/cd/`) when creating high-volume files.
- **Artifact Cleanup:** Ensure temporary files, downloads, and scratch buffers are cleaned up via context managers (`try / finally`) or scheduled retention sweepers.
- **Blocking Disk I/O:** Prohibit synchronous file reads/writes on asynchronous event loops or high-concurrency request threads.

### 7. CPU & Algorithmic Complexity
- **Algorithmic Inefficiency:** Identify $O(N^2)$ or exponential nested loops on unbounded collections where dictionary/set lookups ($O(1)$) or pre-sorting ($O(N \log N)$) provide linear scaling.
- **Event Loop Blocking:** Prohibit CPU-heavy serialization, compression, cryptographic hashing, or synchronous sleep inside async event loops (`asyncio`, Node.js); offload to worker threadpools or background processes.
- **Regex Compilation:** Forbid compiling regular expressions inside functions called repeatedly; compile once at module level. Reject patterns vulnerable to catastrophic backtracking (ReDoS).
- **Busy-Waiting:** Replace tight polling loops with event-driven notifications, condition variables, or exponential polling intervals.

### 8. Concurrency, Contention & Thread Pools
- **Lock Contention:** Minimize the critical section of locks/mutexes. Never hold a mutex across network I/O or slow disk access.
- **Thread & Pool Starvation:** Ensure tasks submitted to bounded thread or process pools cannot block indefinitely without timeouts. Prevent recursive task submission that leads to self-deadlock.

### 9. Descriptors & Resource Leaks
- **Handle Leakage:** Ensure all OS handles (files, sockets, subprocess pipes, database cursors) are wrapped in context managers (`with` / `try...finally`) so exceptions cannot leak descriptors.
- **Dangling Transactions:** Ensure all database transactions are explicitly committed or rolled back; prevent transactions left `IDLE in transaction` from holding table or row locks.

---

## Output Format: Resource Impact Assessment

Output the review findings in a compact, structured Markdown format:

```markdown
### Bare-Metal Resource Efficiency Review

**Context & Scale Assumed:** `<e.g., Web API service under ~200 RPS; batch worker processing up to 50k rows>`

#### Critical & High Impact Findings
| ID | Pillar | Location | Scale Failure Mode & Blast Radius | Proportional Remediation |
|---|---|---|---|---|
| R-01 | DB Query Patterns | `app/services/orders.py:45` | **N+1 Queries:** Loops over 500 orders, executing 500 individual user queries. Saturates DB pool at >20 RPS. | Use `.select_related("user")` / bulk `JOIN` to execute 1 query. |
| R-02 | Connection Lifecycle | `app/db/engine.py:12` | **Stale Connection Drops:** Missing `pool_pre_ping`. Connections reset by cloud firewall after 5 min idle throw 500 errors. | Enable `pool_pre_ping=True` on engine creation. |

#### Medium & Low Impact Findings
| ID | Pillar | Location | Scale Failure Mode & Blast Radius | Proportional Remediation |
|---|---|---|---|---|
| R-03 | CPU / Algorithmic | `app/utils/matcher.py:88` | **Regex Recompilation:** Compiles complex pattern inside loop of 1,000 items, increasing CPU latency by ~15ms per call. | Pre-compile regex at module level (`RE_PATTERN = re.compile(...)`). |

#### Verified Clean Pillars
- **Filesystem & Disk I/O:** No local disk writes or uncleaned temporary files detected.
- **Memory & Buffers:** Payloads are bounded and properly streamed.

#### Next Steps
Wait for developer confirmation before implementing any resource optimizations.
```

---

## Operational Instructions

1. **Strictly analytical & non-reproducing (Safety & Isolation):** Do NOT attempt to reproduce, benchmark, or stress-test suspected resource issues during the investigation. Never create stress-test files on disk, spin up temporary servers, connect to live network resources/databases, or trigger load/failure scenarios unless explicitly asked by the user. The investigation must remain purely analytical, relying exclusively on static code analysis, official tool/library documentation, and internal engineering knowledge.
2. **Be strictly read-only:** Do not modify, commit, or reformat files during the review.
3. **Quantify failure conditions:** Always state *at what scale or load* the issue becomes dangerous (e.g., "$N > 1,000$", "after 10 minutes of inactivity", "under >50 concurrent requests").
4. **Wait for approval:** Present the assessment and await developer feedback before applying changes.
