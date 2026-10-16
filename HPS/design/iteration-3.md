# Hotel Pricing System - Iteration 3

Refine durable tasks created by iteration 2 into a recoverable delivery mechanism. Preserve atomic local publication and REST compatibility with existing systems.

## Iteration Goal

Provide loss-resistant CMS delivery and complete, replayable publication measurements without coupling query availability to CMS health.

## Drivers to Address

HPS-2; QA-2, QA-8; CON-5; CRN-4.

## ADD Process Steps

### Step 2: Establish goal by selecting drivers

Apply iteration-plan.md iteration 3. Publication readiness and CMS receipt are separate durable milestones. Prioritize no silent loss, explicit unknown outcomes and receiver acknowledgement over an unsupported exactly-once claim.

### Step 3: Choose elements to refine

Refine the delivery worker into task claimant, lease repository, coordinator, CMS adapter, attempt ledger, reconciler and measurement exporter. Refine Delivery states and Publication measurement responsibilities. External CMS remains a negotiated dependency.

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
|---|---|---|---|
| Database-backed transactional delivery queue | Reuses atomic task creation; no extra messaging platform initially | Polling, retention and database load require management | Mandatory broker from day one without measured need |
| At-least-once delivery and receiver deduplication | Recovers lost responses and sender crashes | Receiver contract is necessary; duplicate transmission remains possible | Exactly-once HTTP delivery claim |
| Per-hotel ordering, leases and fencing | Limits concurrent senders and stale acknowledgements | Blocked event delays later events for that hotel | Unordered overwrite or skipping failed changes |
| Durable unsampled ledger and replayable export | Complete accounting despite monitoring outage | Storage, retention and reconciliation overhead | Sampled traces or volatile counters as sole evidence |

### Step 5: Instantiate elements, views, responsibilities and interfaces

Architecture section 6 contains the worker component diagram. Sequence 7.4 defines success and ambiguous-outcome recovery before section 8's CMS and measurement contracts.

| Instantiation decision | Rationale |
|---|---|
| Delivery states are Pending, Leased, RetryDue, Blocked and Acknowledged | Separates temporary, administrative and confirmed outcomes |
| Persist attempt intent before network send and outcome afterwards | Crash between send and acknowledgement remains detectable as unknown |
| Use publication ID and per-hotel sequence on every retry | Receiver can deduplicate and prevent late old writes |
| Require durable receipt; poll on asynchronous acceptance | HTTP success alone may not prove receipt semantics |
| Retain blocked work, alert and redrive after repair | A retry cap must not become a data-loss mechanism |
| Export by durable checkpoint with record-count reconciliation | Monitoring failure does not erase publication evidence |

An expiring lease protects progress after crashes; fencing protects local state updates. Neither alone prevents an old network request from reaching CMS, so receiver-side sequencing remains essential. A circuit breaker limits pressure during outages while retaining work. Unbounded outage duration still implies unbounded backlog, which cannot be guaranteed on finite storage: alert and apply explicit admission/backpressure policy before capacity exhaustion, never falsely accept unpersisted work.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
|---|---|---|---|
| QA-2, CON-5 | D6: leased REST sender and receipt reconciliation | Durable recovery while respecting current contracts | Synchronous CMS call during price commit |
| QA-8 | D7: unsampled publication/attempt ledger | Reconstructs completeness and timing evidence | Sampling-only monitoring |
| CRN-4, HPS-2 | Preserve D4's immutable accepted payload | Replays exactly the accepted change | Reading mutable latest prices during retry |

Decisions are consolidated in architecture section 9. Publication-to-receipt delay is a distinct metric from QA-1 readiness latency.

### Step 7: Analyze design and review achievement

| Driver | Analysis result |
|---|---|
| HPS-2 | Satisfied at design level for internal workflow; external delivery depends on agreed CMS contract |
| QA-2 | Partially satisfied: durable recovery and no silent discard designed; receiver receipt/deduplication and maximum outage/recovery horizon are unconfirmed |
| QA-8 | Partially satisfied: complete accepted-publication ledger designed; retention and pre-persistence failure measurement scope require agreement |
| CON-5 | Satisfied at design level: initial REST delivery/query contracts remain isolated behind adapters |
| CRN-4 | Partially satisfied: explicit state machine avoids hidden retry semantics; operational ownership and exercises remain |

Recovery architecture is defined. Carry CMS contract confirmation and storage/recovery limits to the release gates. Proceed to query resilience without making CMS a query dependency.
