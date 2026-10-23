---
name: resource-efficiency-review
description: Conducts an evidence-based bare-metal technical review focused strictly on physical resource usage, connection resilience, saturation limits, and leak prevention. Use when asked to review or audit code for resource efficiency, performance bottlenecks, database query scaling (N+1, indexing), connection pooling, memory leaks, or bare-metal runtime limits.
---

This skill conducts a technical audit of code changes. It ignores style, design patterns, and naming choices. It evaluates only physical resource usage, connection resilience, saturation limits, and system exhaustion.

Trace physical boundaries: Memory $\to$ Sockets $\to$ Network $\to$ Database $\to$ Disk. Check driver defaults for pooling and buffering. Simulate behavior under scale ($N \to \infty$, high concurrency, network outages). Propose minimal, proportional fixes.

## Output Style Requirement

You **MUST** write all review assessments, failure descriptions, and remediations 80% of the way to ASD-STE100:
- Keep sentences under 20 words.
- Use active voice and imperative commands.
- State facts directly. Omit conversational filler, narrative intros, and speculative hedging.
- Quantify scale failure thresholds (e.g., `>20 RPS`, `N > 1,000`, `idle timeout > 5 min`).
- State direct imperative remediations.
- Use one term per concept. Do not rotate synonyms.

---

## The Proportionality Rule

Calibrate every finding to the application scope and traffic profile:

1. **Scale Calibration:**
   - Distinguish CLI tools, internal batch jobs, low-traffic APIs (<50 RPS), and high-concurrency APIs (>1,000 RPS).
   - Do not propose streaming or complex pagination when domain rules limit datasets to small sizes ($N < 100$).
   - Require chunking, streaming, and indexing when inputs can grow unbounded ($N > 10,000$).
2. **Native Primitives:**
   - Use built-in driver, ORM, and framework features.
   - Do not build custom keeper threads, homebrew connection supervisors, or distributed retry meshes.
3. **Simplicity Test:**
   - Ensure the remediation introduces fewer moving parts and failure modes than the detected issue.

---

## The 9 Physical Resource Pillars

Evaluate code against these 9 pillars:

### 1. Connection Lifecycle & Failure Resilience
- **Pool Reuse:** Acquire database, Redis, and HTTP client connections from shared singletons. Do not create clients per request.
- **Lifespan Management:** Initialize connection pools during application startup. Drain pools during graceful shutdown.
- **Stale Connection Recovery:** Verify connection health before use (e.g., `pool_pre_ping=True`).
- **Connection Hoarding:** Do not hold active transactions across external HTTP calls or heavy CPU operations. Release connections before non-database I/O.
- **Graceful Degradation:** Ensure failures in auxiliary services (cache, telemetry) do not crash core transactions.

### 2. Database Query Patterns
- **N+1 Queries:** Do not execute database queries inside loops. Use `JOIN`, eager loading, or batched `IN` queries.
- **Cartesian Joins:** Flag unbounded multi-table joins that multiply result row counts.
- **Unbounded Queries:** Enforce `LIMIT` or keyset pagination on growing tables. Prohibit unbounded `SELECT *`.

### 3. Database Indexing & Query Planning
- **Missing Indexes:** Index columns used in `WHERE`, `JOIN ON`, `ORDER BY`, and foreign keys for large tables.
- **Composite Index Alignment:** Align multi-column query filters with the left-to-right column order of composite indexes.
- **Type Coercion:** Prohibit function wrappers on indexed columns (e.g., `WHERE LOWER(email) = ...`) that cause full scans.
- **Over-Indexing:** Do not add redundant indexes on write-heavy append tables.

### 4. Network & RPC Efficiency
- **Loop Requests:** Prohibit sequential HTTP or RPC calls in loops. Require bulk endpoints.
- **Session Reuse:** Use persistent HTTP client sessions with connection pooling and keep-alive.
- **Timeouts & Retries:** Set explicit connect and read timeouts on every network call. Use exponential backoff with jitter.

### 5. Memory & Buffer Management
- **Streaming Buffers:** Do not read entire large files or database results into memory. Use chunked streaming or generators.
- **Bounded Caches:** Enforce maximum size, TTL, or LRU eviction policies on in-memory caches.
- **Column Filtering:** Fetch required columns only. Do not fetch entire tables when payloads are large.

### 6. Filesystem & Disk I/O
- **Directory Partitioning:** Do not write unbounded files into a single flat directory. Use partitioned subdirectories.
- **Artifact Cleanup:** Delete temporary files and scratch buffers using context managers (`try / finally`).
- **Async I/O:** Do not execute synchronous disk reads or writes on async event loops.

### 7. CPU & Algorithmic Complexity
- **Algorithmic Scaling:** Replace $O(N^2)$ nested loops on unbounded collections with $O(1)$ lookups or $O(N \log N)$ sorts.
- **Non-Blocking Loops:** Offload CPU-heavy compression, cryptographic hashing, and serialization from event loops to worker pools.
- **Regex Compilation:** Compile regular expressions once at module scope. Prohibit regex patterns vulnerable to catastrophic backtracking.
- **Polling Intervals:** Replace tight polling loops with event-driven notifications or exponential backoff.

### 8. Concurrency & Contention
- **Lock Contention:** Keep critical sections minimal. Never hold locks across network I/O or disk operations.
- **Pool Starvation:** Set timeouts on bounded worker pools. Prevent recursive task submission that causes deadlock.

### 9. Descriptors & Resource Leaks
- **Handle Closure:** Wrap all OS handles (files, sockets, database cursors) in context managers.
- **Transaction Termination:** Commit or roll back all database transactions explicitly. Prevent idle transaction lock holding.

---

## Output Format: Resource Impact Assessment

Output review findings using this Markdown structure. Write all text adhering to the 80% ASD-STE100 rule.

```markdown
### Bare-Metal Resource Efficiency Review

**Context & Scale Assumed:** `<e.g., Web API service under ~200 RPS; batch worker processing up to 50k rows>`

#### Critical & High Impact Findings
| ID | Pillar | Location | Scale Failure Mode & Blast Radius | Proportional Remediation |
|---|---|---|---|---|
| R-01 | Database Query Patterns | `app/services/orders.py:45` | **N+1 Queries:** Loops over 500 orders. Executes 500 individual queries. Saturates pool at >20 RPS. | Use `.select_related("user")` to execute 1 bulk query. |
| R-02 | Connection Lifecycle | `app/db/engine.py:12` | **Stale Connection Drops:** Missing `pool_pre_ping`. Firewalls drop idle connections after 5 minutes, raising 500 errors. | Set `pool_pre_ping=True` during engine creation. |

#### Medium & Low Impact Findings
| ID | Pillar | Location | Scale Failure Mode & Blast Radius | Proportional Remediation |
|---|---|---|---|---|
| R-03 | CPU & Algorithmic Complexity | `app/utils/matcher.py:88` | **Regex Recompilation:** Compiles pattern inside loop of 1,000 items. Adds ~15ms latency per request. | Pre-compile regex at module scope (`RE_PATTERN = re.compile(...)`). |

#### Verified Clean Pillars
- **Filesystem & Disk I/O:** No local disk writes or uncleaned temporary files detected.
- **Memory & Buffers:** Payloads are bounded and properly streamed.

#### Next Steps
Wait for developer confirmation before implementing any resource optimizations.
```

---

## Operational Instructions

1. **Static Analysis Only:** Do NOT attempt to reproduce, benchmark, or stress-test suspected resource issues. Do not create stress-test files, spin up temporary servers, connect to live databases, or trigger failure scenarios. Rely exclusively on static code analysis, official library documentation, and systems knowledge.
2. **Read-Only Mode:** Do not edit, stage, or reformat files during review.
3. **Quantify Failure Limits:** Always state the scale or load threshold that triggers failure (e.g., `$N > 1,000$`, `idle time > 5 min`, `>50 concurrent requests`).
4. **Wait for Approval:** Present the assessment. Await developer approval before implementing optimizations.
