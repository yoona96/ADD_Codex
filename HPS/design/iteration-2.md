# Hotel Pricing System - Iteration 2

Completed on 2026-10-14 as an ADD design iteration. The user explicitly authorized execution through the last iteration without review pauses. Completion means the design steps and analysis are documented, not that the system is implemented or its quality targets verified.

## Iteration Goal

Refine secure, correct hotel/rate configuration, simulation, publication and price queries, with a credible path to the 100 ms target.

## Drivers to Address

HPS-1, HPS-2, HPS-3, HPS-4, HPS-5, HPS-6; QA-1, QA-5; CON-2, CON-4.

## ADD Process Steps

### Step 1: Review inputs

Start from Iteration 1's modular application and owned store. The source leaves tax/rounding/currency policies, timing percentile, workload, external identity and multi-date atomicity unspecified. Use explicit provisional contracts rather than silent defaults.

### Step 2: Establish the goal by selecting drivers

Follow Iteration 2 of [iteration-plan.md](iteration-plan.md). Refine secure, correct hotel/rate configuration, simulation, publication and price queries, with a credible path to the 100 ms target. Priorities and assumptions remain traceable to [architectural-drivers.md](../architectural-drivers.md).

### Step 3: Choose elements to refine

| Element | Refinement scope |
| --- | --- |
| Pricing engine and change service | Deterministic calculation, simulation provenance, transaction boundary and stale-change handling. |
| Configuration and permission services | Versioned drafts, explicit activation, login and scoped administration. |
| Price query adapter and store | Consistent snapshots, authorization and immediate visibility after commit. |

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
| --- | --- | --- | --- |
| Pure deterministic decimal engine | Simulation and commit share behavior; no network dependence in calculation. | Rule vocabulary and rounding need business decisions. | Arbitrary scripts: hard to constrain, audit and benchmark. |
| Atomic single-hotel/date publication | Complete matrix becomes visible together; short local transaction. | Large matrices and same-hotel lock contention threaten QA-1. | Async read projection: adds freshness lag; calculate-on-query: increases read cost. |
| Expected versions and idempotent commands | Detect stale edits; safely resolve timeout-after-commit. | Retained result records and explicit conflict UX. | Last-write-wins or blindly resubmitted POST: loses intent or duplicates changes. |
| Federated login plus current server-side grants | Provider owns credentials; scopes enforced across all transports. | Grant reads add database work; provider integration requires validation. | UI-only checks and token-only hotel grants: bypass and revocation risks. |
| Versioned configuration drafts and explicit activation | Keeps published amounts reproducible and supports prospective changes. | Per-date activation progress is visible; all-date atomicity not supplied. | Mutating rules underneath existing prices: ambiguous price semantics. |

### Step 5: Instantiate elements, sketch views and define interfaces

| Instantiation decision | Rationale |
| --- | --- |
| Single-date price changes include expected price/configuration versions and idempotency key | The UI coordinates multiple dates with individually visible outcomes. |
| One transaction locks grant, hotel control and date records, recalculates, writes prices/publication/outbox/audit/result, then commits | Queries never observe a partial matrix and successful responses imply durable query visibility. |
| Simulation is read-only and returns input versions; commit recalculates | A preview cannot silently overwrite intervening updates. |
| Provider authorization code with PKCE, server-validated tokens and grant checks | Satisfy identity validation and hotel/operation restrictions without HPS storing passwords. |
| Configuration activation creates explicit child recalculations | Tax/rule edits never silently reinterpret already published values. |

S1 covers login, S2 simulation/commit, S3 authorized reads and S4 administration. Section 8 derives REST routes, error vocabulary and transaction/locking rules from these interactions. The consolidated result is [architecture.md](architecture.md); later iterations refine these views without introducing separate conflicting architecture documents.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
| --- | --- | --- | --- |
| HPS-2; QA-1 | D04: Synchronous atomic matrix publication | Meets intended readiness boundary without a query projection hop. | Asynchronous calculation/projection. |
| HPS-1, HPS-6; QA-5; CON-2 | D05: Federated identity plus live grants | Enforces authorization on every server path. | Browser-only controls. |
| HPS-4, HPS-5 | D06: Versioned constrained rules | Preserves reproducibility and validation. | Unrestricted scripts. |
| HPS-2 | D07: Versions, idempotency and hotel sequence lock | Makes conflicts and uncertain responses recoverable. | Last-write-wins and blind retries. |

These decision IDs are recorded in section 9 of the architecture document.

### Step 7: Analyze the design and review goal achievement

| Driver | Analysis result |
| --- | --- |
| HPS-1, HPS-3, HPS-6 | Satisfied at design level: login, consistent authorized queries and audited grants defined. |
| HPS-2, HPS-4, HPS-5 | Partially satisfied: currency, tax, rule vocabulary and activation/multi-date policies need business acceptance. |
| QA-1 | Partially satisfied: synchronous path and 100 ms allocation exist; target boundary and measured worst-case performance unresolved. |
| QA-5, CON-2 | Satisfied at design level: provider identity and server-side grants; provider and security tests remain. |
| CON-4 | Partially satisfied: baseline supports MVP scope; staffing and estimates still unknown. |

The commit writes a durable delivery intent, but external receipt requires Iteration 3's dispatcher and reconciliation design.

Validation experiments are recorded in [validation-plan.md](validation-plan.md). Satisfied at design level means sufficient decisions for the architectural intent; measurements and implementation conformance are separate evidence.
