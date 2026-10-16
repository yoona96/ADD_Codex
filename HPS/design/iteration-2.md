# Hotel Pricing System - Iteration 2

Refine the pricing and configuration modules established in iteration 1. Preserve the whole-system decomposition and concentrate on the prioritized business transaction.

## Iteration Goal

Make preview deterministic and accepted price changes atomically queryable, with recoverable publication work and explicit latency bounds.

## Drivers to Address

HPS-2, HPS-3, HPS-4, HPS-5; QA-1, QA-2; CRN-4.

## ADD Process Steps

### Step 2: Establish goal by selecting drivers

Apply iteration-plan.md iteration 2. QA-1's less-than-100-ms target and QA-2's no-loss requirement jointly determine the commit boundary. CMS receipt is a separate milestone, refined next.

### Step 3: Choose elements to refine

Refine pricing application service, calculator, configuration service, price query service and transaction/repository ports. Refine the Price concept into complete versioned PriceSets for hotel/date atomicity. Preserve fixed-rate independence and derived-rate dependencies.

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
|---|---|---|---|
| Pure calculator with versioned policies | Same result for preview and commit; easy boundary testing | Rule language, rounding and effective dates must be specified | Arbitrary executable rules or remote calculation dependency |
| Atomic price/publication/task/receipt transaction | Complete visible prices and durable delivery work share one commit | Database contention and transaction size require limits | Independent writes or synchronous CMS call inside transaction |
| Expected versions and deterministic locks | Detects stale preview and prevents lost update | Conflicts require user review; hot hotels can serialize | Last-writer-wins with silent overwrites |
| Primary reads of materialized price sets | Immediate visibility and low query computation | Database is a critical dependency | Async query projection without a freshness contract |

### Step 5: Instantiate elements, views, responsibilities and interfaces

Architecture sections 4 and 6 now distinguish PriceSet, Publication and calculator responsibility. Sequences 7.2 and 7.3 precede the simulation, change, configuration and query interfaces in section 8. Query flow is consolidated in 7.5 with later resilience refinements.

| Instantiation decision | Rationale |
|---|---|
| Calculate all room/rate amounts for affected hotel/date before commit | Prevents mixed old/new prices from becoming queryable |
| Pin configuration and base/fixed versions; recalculate during commit | Preview results cannot bypass concurrent input changes |
| Save immutable publication payload with the price rows | Delivery retry sends the accepted version rather than current mutable data |
| Store request hash and result under unique subject/hotel/idempotency key | Lost responses can be retried without duplicate accepted changes |
| Limit request dimensions and measure ingress-to-commit separately from CMS latency | Makes the QA-1 target measurable without conflating external delivery |
| Stage configuration edits and explicitly trigger affected-date recalculation | Existing prices keep explainable tax/rule versions |

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
|---|---|---|---|
| HPS-2, HPS-4, HPS-5, QA-1 | D3: bounded deterministic calculation | Predictable work and preview parity | Remote or unbounded rule execution |
| QA-1, QA-2 | D4: atomic publication and command receipt | No accepted change without complete prices and recoverable task | Database/network dual write |
| HPS-3, QA-1 | D5: authoritative price-set reads | A new query after commit sees the committed state or a later valid version | Unqualified stale cache or replica reads |

Architecture section 9 is the decision record. Numeric service budgets are engineering hypotheses and remain subject to profiling.

### Step 7: Analyze design and review achievement

| Driver | Analysis result |
|---|---|
| HPS-2 | Partially satisfied: preview/change/publication designed; CMS receipt is iteration 3 |
| HPS-3 | Satisfied at design level: authorized bounded query and complete snapshot contract |
| HPS-4, HPS-5 | Partially satisfied: versioned configuration/rules designed; business rounding, tax and activation semantics need confirmation |
| QA-1 | Partially satisfied: local deterministic calculation and atomic visibility support target; normal workload and strict timing acceptance remain unspecified and unmeasured |
| QA-2 | Partially satisfied: no-loss acceptance boundary designed; downstream delivery still needs failure handling and receiver agreement |
| CRN-4 | Partially satisfied: explicit invariants prevent hidden coupling; boundary and migration checks remain |

The transaction design goal is achieved, with quantitative performance and business policy gates open. Iteration 3 consumes the durable publication/task pair; it must not move CMS communication into the commit path.
