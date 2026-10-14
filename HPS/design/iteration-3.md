# Hotel Pricing System - Iteration 3

Completed on 2026-10-14 as an ADD design iteration. The user explicitly authorized execution through the last iteration without review pauses. Completion means the design steps and analysis are documented, not that the system is implemented or its quality targets verified.

## Iteration Goal

Make every accepted price change durably query-visible and recoverably deliverable to CMS, with complete publication accounting.

## Drivers to Address

HPS-2, HPS-3; QA-2, QA-8, QA-9; CON-5; CRN-4.

## ADD Process Steps

### Step 1: Review inputs

Iteration 2 establishes atomic accepted changes. CMS acknowledgement, duplicate handling and ordering support remain unknown. Interpret QA-2 as no lost accepted changes plus eventual receipt after recoverable failures, explicitly conditional on an agreed CMS contract.

### Step 2: Establish the goal by selecting drivers

Follow Iteration 3 of [iteration-plan.md](iteration-plan.md). Make every accepted price change durably query-visible and recoverably deliverable to CMS, with complete publication accounting. Priorities and assumptions remain traceable to [architectural-drivers.md](../architectural-drivers.md).

### Step 3: Choose elements to refine

| Element | Refinement scope |
| --- | --- |
| Publication store and outbox | Durable immutable payload and pending destination records in the price transaction. |
| Delivery worker and CMS REST adapter | Leases, ordering, timeout classification, replay and receipt confirmation. |
| Attempt ledger and metric exporter | Persistent evidence and reconciliation after telemetry outages. |

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
| --- | --- | --- | --- |
| Transactional outbox with direct REST dispatcher | Closes price-commit/delivery-intent gap without a mandatory broker. | DB polling and backlog retention consume resources. | Direct dual write: can lose delivery; broker-only publication: not atomic with DB. |
| At-least-once transmission with receipt deduplication | Survives acknowledgement loss and process crashes. | Requires compatible CMS or manual reconciliation; no blanket exactly-once claim. | Assume a timeout means not received: duplicate effects. |
| Per-hotel ordered delivery | Preserves each accepted change and isolates blocked hotels. | A poison record stalls later changes for that hotel. | Coalescing superseded updates: violates every-change receipt. |
| Durable ledger and checkpointed export | Can reconcile missing or duplicated publication measures. | Additional retention, access controls and export logic. | Sampled traces alone: cannot prove complete counts. |

### Step 5: Instantiate elements, sketch views and define interfaces

| Instantiation decision | Rationale |
| --- | --- |
| Worker leases a hotel/destination and loads the earliest unacknowledged publication | Parallelism across hotels with ordered delivery within a hotel. |
| Persist attempt start before HTTP; confirm receipt after response; no DB transaction held during network call | Crash outcomes remain detectable without holding locks across CMS latency. |
| Stable publication ID, sequence, immutable matrix and receipt ID | Retries cannot accidentally send later prices as if they belonged to an earlier change. |
| Unknown receipts and permanent rejections enter retained reconciliation state | No silent deletion, skipped publication or false success after retry exhaustion. |
| CMS must deduplicate publication IDs and reject stale sequences | Leases alone cannot fence a paused worker's late request. |

Worker component view, S5 delivery/recovery sequence and section 8 CMS envelope/state/failure table capture the refinement. Section 9 records D08-D09. The consolidated result is [architecture.md](architecture.md); later iterations refine these views without introducing separate conflicting architecture documents.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
| --- | --- | --- | --- |
| QA-2; CON-5 | D08: Atomic outbox and direct CMS REST worker | Durable scheduling and external failure isolation. | Synchronous dual write; broker introduced without need. |
| HPS-2; QA-2, QA-8 | D09: Ordered immutable delivery and receipt ledger | Preserves all accepted changes and supports recovery. | Coalescing or dropping failed publications. |

These decision IDs are recorded in section 9 of the architecture document.

### Step 7: Analyze the design and review goal achievement

| Driver | Analysis result |
| --- | --- |
| HPS-2, QA-2 | Partially satisfied: commit and recovery design established; CMS contract and permanent-failure boundaries prevent unconditional 100% guarantee. |
| HPS-3 | Satisfied at design level: queries do not wait for CMS delivery. |
| QA-8 | Partially satisfied: durable evidence and export recovery defined; required measure inventory/retention and total-failure completeness unresolved. |
| QA-9 | Partially satisfied: worker test seams and failure cases defined; full element inventory completed in Iteration 5. |
| CON-5 | Satisfied at design level: REST adapter isolates the existing CMS contract. |
| CRN-4 | Partially satisfied: failure modes and alternatives documented; CMS capability debt is explicit. |

Iteration 4 refines query scaling and database availability while preserving primary-read visibility and durable accepted-change semantics.

Validation experiments are recorded in [validation-plan.md](validation-plan.md). Satisfied at design level means sufficient decisions for the architectural intent; measurements and implementation conformance are separate evidence.
