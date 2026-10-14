# Hotel Pricing System - Iteration 4

Completed on 2026-10-14 as an ADD design iteration. The user explicitly authorized execution through the last iteration without review pauses. Completion means the design steps and analysis are documented, not that the system is implemented or its quality targets verified.

## Iteration Goal

Refine the deployment and data-access design to support query availability and tenfold daily query growth without breaking publication consistency.

## Drivers to Address

HPS-3; QA-3, QA-4; CON-6; regression review of QA-1, QA-2, QA-5.

## ADD Process Steps

### Step 1: Review inputs

Queries and writes share the HPS-owned store but no external integration dependency. Daily totals alone do not establish peak capacity. QA-4 wording remains provisionally interpreted as at most 20% mean latency increase; SLA accounting and disaster RTO/RPO are unspecified.

### Step 2: Establish the goal by selecting drivers

Follow Iteration 4 of [iteration-plan.md](iteration-plan.md). Refine the deployment and data-access design to support query availability and tenfold daily query growth without breaking publication consistency. Priorities and assumptions remain traceable to [architectural-drivers.md](../architectural-drivers.md).

### Step 3: Choose elements to refine

| Element | Refinement scope |
| --- | --- |
| Query runtime and ingress | Independent scaling, zone redundancy and request/connection limits. |
| Managed database and connection adapters | Fenced failover, synchronous durability, backups and reconnect behavior. |
| Operational capacity model | Paired load tests, hot-hotel contention and bounded admission. |

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
| --- | --- | --- | --- |
| Independent stateless query pool | Scales reads and isolates application resources from commands/workers. | Database remains shared; more runtime configuration. | One undifferentiated pool: command/worker load can exhaust query resources. |
| Primary snapshot reads without price cache | Preserves immediate post-commit visibility and avoids invalidation. | Read load remains on writer. | Async read replica or cache: freshness semantics would need new decisions. |
| Multi-zone managed DB and runtime | Reduces single-zone failure exposure with managed operations. | Cost and synchronous commit latency; provider details matter. | Single-zone deployment: insufficient isolation; regional active-active: excessive conflict complexity. |
| Bounded connections, request shapes and measured scale-out | Protects common database and makes capacity testable. | Throttling must be visible and not counted as success. | Unlimited autoscaling: can overload the database. |

### Step 5: Instantiate elements, sketch views and define interfaces

| Instantiation decision | Rationale |
| --- | --- |
| At least two production query instances across zones behind managed ingress | A failed instance can be removed while others serve. |
| Distinct query, command and worker DB connection budgets | Prevent one workload consuming all database connections. |
| Managed writer endpoint with fenced standby promotion, fresh connections and bounded retry | Count failover interruption honestly against the SLA. |
| Provisional 12/116 requests per second paired peaks and 250 requests per second burst | Tenfold daily totals imply 1.16/11.57 average requests per second, not realistic peaks; validate these assumed peaks. |
| No stale fallback prices; fail unavailable during DB interruption | Avoid presenting outdated pricing as authoritative. |

Section 5 adds production deployment, S3 covers scaled queries and S6 covers failover. Section 8 supplies capacity calculations and provisional latency/recovery targets. The consolidated result is [architecture.md](architecture.md); later iterations refine these views without introducing separate conflicting architecture documents.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
| --- | --- | --- | --- |
| HPS-3; QA-3, QA-4, QA-1 | D10: Independent query pool and primary reads | Balance horizontal app capacity and consistent visibility. | Eventually consistent query store or cache now. |
| QA-3, QA-2; CON-6 | D11: Managed fenced failover and tested backups | Protect committed state in a defined single-zone failure model. | Single-zone DB; unproven active-active design. |

These decision IDs are recorded in section 9 of the architecture document.

### Step 7: Analyze the design and review goal achievement

| Driver | Analysis result |
| --- | --- |
| HPS-3 | Satisfied at design level: consistent query route and deployment defined. |
| QA-3 | Partially satisfied: redundancy and recovery path specified; provider behavior, SLA period and measurements pending. |
| QA-4 | Partially satisfied: scale mechanisms and paired experiment defined; target wording, peak profile and results pending. |
| CON-6 | Satisfied at design level: managed runtime, state and health interfaces. |
| QA-1 regression | Partially satisfied: primary visibility preserved; synchronous replication and lock contention must fit budget. |
| QA-2 regression | Partially satisfied: outbox survives defined durable commits; regional loss and CMS receipt scope remain unresolved. |
| QA-5 regression | Satisfied at design level: all replicas share mandatory authorization policy and current grants. |

Iteration 5 completes release mechanics, test coverage, observability and the final evidence inventory. Scaling is a design proposal, not a benchmark result.

Validation experiments are recorded in [validation-plan.md](validation-plan.md). Satisfied at design level means sufficient decisions for the architectural intent; measurements and implementation conformance are separate evidence.
