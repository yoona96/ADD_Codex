# Hotel Pricing System - Iteration 4

Refine the query deployment and complete access control on the foundation from iterations 1-3.

## Iteration Goal

Support query availability and tenfold daily volume while enforcing current hotel-level permissions across browser and external callers.

## Drivers to Address

HPS-1, HPS-3, HPS-6; QA-3, QA-4, QA-5; CON-2, CON-6.

## ADD Process Steps

### Step 2: Establish goal by selecting drivers

Apply iteration-plan.md iteration 4. Address high-priority availability and scalability without weakening iteration 2's immediate query visibility. Complete security beyond login and UI filtering.

### Step 3: Choose elements to refine

Refine API deployment, authoritative database, load balancing, query resource pools, authorization and session management. Keep the calculator and CMS worker independent of the query path.

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
|---|---|---|---|
| Multi-zone replicas and synchronous standby | Removes individual application node and zone as sole execution points | Failover still interrupts requests; higher recurring cost | Single-zone production deployment |
| Indexed primary reads and bounded queries | Simple consistency and predictable query work | Primary remains a scaling bottleneck | Early read replicas/caches with undefined freshness |
| Query bulkheads and admission control | Protects queries from write/delivery saturation | Requires tuning; excess traffic can be rejected | Unlimited threads or database connections |
| Authoritative per-request entitlements | Clear post-commit permission revocation boundary | Adds database work to protected requests | Long-lived permission claims with unbounded stale authorization |

### Step 5: Instantiate elements, views, responsibilities and interfaces

Architecture section 5 shows the two-zone application and managed standby topology. Sequences 7.1 and 7.5 define permission updates, authorized queries and failover; section 8 records the resulting query/access contracts.

| Instantiation decision | Rationale |
|---|---|
| Deploy two or more healthy API replicas and route only to ready instances | Supports node failure and rolling updates |
| Reserve bounded database connections/concurrency for query traffic | Prevents publication and polling from exhausting query resources |
| Use the primary for all authoritative queries | Avoids weakening QA-1 through replica lag |
| Validate machine token identity and current hotel entitlement | External API access must follow the same scope policy as users |
| Read updated permissions on the next authorization check | Makes revocation behavior explicit without hidden cache delay |
| Measure volume, mean latency, tails, saturation and errors | Daily totals and average latency alone can hide peak failures |

Average demand grows from approximately 1.16 to 11.57 queries per second. A provisional peak factor of ten gives approximately 116 queries per second for sizing experiments, not a contractual peak. Both load levels must use the same hotel distribution, query shape, warm-up policy and error accounting. Compare mean latencies at a stable system state and reject results that improve by dropping requests. Autoscaling does not remove the need to measure database capacity.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
|---|---|---|---|
| QA-3, QA-4, CON-6 | D8: multi-zone failover and resource reservations | Limits single-node failure and workload interference | Active-active multi-region complexity without RTO/RPO justification |
| QA-5, HPS-6 | D2 refinement: current server-side entitlements and audit | Direct-object requests cannot bypass UI restrictions | Client-side restrictions and stale authorization caches |
| HPS-3, QA-4 | D5 refinement: indexes, bounds and versioned pagination | Controls scan cost while preserving snapshot consistency | Unbounded scans and live pagination over changing rows |

The refined decisions are in architecture section 9. Database backups and restore drills are required, but retention and regional recovery commitments remain open business decisions.

### Step 7: Analyze design and review achievement

| Driver | Analysis result |
|---|---|
| HPS-1, HPS-6 | Satisfied at design level: provider login, scoped administration and permission update semantics |
| HPS-3 | Satisfied at design level: consistent authorized queries and explicit unavailable responses |
| QA-3 | Partially satisfied: multi-zone failover and isolation designed; accounting policy, RTO/RPO and fault-test evidence pending |
| QA-4 | Partially satisfied: capacity tactics and comparison method exist; wording, workload distribution and benchmark evidence pending |
| QA-5 | Satisfied at design level under stated in-flight revocation boundary; negative security tests and provider capability confirmation remain |
| CON-2, CON-6 | Satisfied at design level: provider identity and managed cloud deployment |

All primary drivers now have design responses. Quantitative compliance remains unproven. Iteration 5 completes delivery, testability and validation governance rather than treating this analysis as production acceptance.
