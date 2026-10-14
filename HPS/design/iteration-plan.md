# Hotel Pricing System - Iteration Plan

Regenerated on 2026-10-14 from the current `../architectural-drivers.md` and its preparation model. All five ADD design iterations are complete. The user's subsequent instruction to proceed through the last iteration without asking supersedes intermediate review pauses. Planning assumptions remain provisional; quality targets require validation, not just architectural decisions.

The consolidated deliverable is [architecture.md](architecture.md). Read [validation-plan.md](validation-plan.md) for unresolved acceptance definitions and experiments that have not yet been executed.

| Iteration record | Status | Main decision outcome |
|---|---|---|
| [Iteration 1](iteration-1.md) | Design steps complete | Modular Java/Angular structure and ports/adapters. |
| [Iteration 2](iteration-2.md) | Design steps complete | Secure versioned simulation, configuration and atomic price publication. |
| [Iteration 3](iteration-3.md) | Design steps complete | Durable ordered CMS delivery and receipt reconciliation. |
| [Iteration 4](iteration-4.md) | Design steps complete | Independent query scaling and managed failover design. |
| [Iteration 5](iteration-5.md) | Design steps complete | Artifact promotion, complete test inventory and operational validation plan. |

## Iteration Plan Overview

| Iteration | Goal | Drivers to address |
|---|---|---|
| 1 | Establish initial system structure around the primary business workflows. | HPS-2, HPS-3, HPS-4; CRN-1 through CRN-4; CON-1, CON-2, CON-4, CON-5, CON-6; QA-6, QA-9 |
| 2 | Refine secure hotel configuration, pricing simulation, changes and queries. | HPS-1 through HPS-6; QA-1, QA-5; CON-2, CON-4 |
| 3 | Refine reliable price publication and CMS delivery with observable recovery. | HPS-2, HPS-3; QA-2, QA-8, QA-9; CON-5; CRN-4 |
| 4 | Refine query availability and scalability. | HPS-3; QA-3, QA-4; CON-6; regression review of QA-1, QA-2, QA-5 |
| 5 | Complete deployment, independent testing and operational readiness. | QA-6 through QA-9; CON-3, CON-4; CRN-3 through CRN-5; release validation of QA-1 through QA-5 |

All primary quality drivers are addressed before the final readiness iteration. Protocol isolation and testability enter the initial structure to limit rework. Pipeline implementation begins early even though its final refinement occurs in Iteration 5.

## Detailed Iteration Goals

### Iteration 1: Initial structure

Refine the whole HPS, the sole initial greenfield element. Establish responsibilities for browser interaction, application workflows, domain behavior, persistence and external integration. Evaluate boundaries using the Java/Angular expertise, cloud-native preference, REST compatibility and delivery deadlines; do not assume a service topology in advance.

Drivers: HPS-2, HPS-3, HPS-4; CRN-1 through CRN-4; CON-1, CON-2, CON-4, CON-5, CON-6; QA-6, QA-9. Outputs after review: structural views, responsibility allocation, principal sequences, interface boundaries and trade-offs. QA-1 through QA-5 constrain choices but remain for detailed refinement.

### Iteration 2: Secure pricing workflow

Refine hotel/rate configuration, identity, permissions, simulation, recalculation and query visibility. Establish business invariants, concurrency handling and consistent price versions. Resolve tax, currency, rounding and effective-date policies before finalizing calculation contracts.

Drivers: HPS-1 through HPS-6; QA-1, QA-5; CON-2, CON-4. Evidence: authorized and denied sequences, simulation isolation, calculation examples, visibility contracts and a latency budget. Define the QA-1 measurement boundary and workload before treating the 100 ms target as testable.

### Iteration 3: Reliable publication

Refine accepted-change durability, price visibility and CMS delivery. Analyze crashes, timeouts, duplicate requests, delivery ordering, retry exhaustion and prolonged CMS failure. Compare durable delivery and reconciliation options against operational complexity and external capabilities.

Drivers: HPS-2, HPS-3; QA-2, QA-8, QA-9; CON-5; CRN-4. Evidence: failure sequences, acknowledgement/recovery contracts, telemetry inventory and fault-injection criteria. The provisional eventual-delivery interpretation must be confirmed; REST alone provides no exactly-once guarantee.

### Iteration 4: Available and scalable queries

Refine deployment topology, capacity, dependency isolation and data access. Evaluate replicas and caches only with explicit freshness and authorization semantics. Define failover and recovery behavior.

Drivers: HPS-3; QA-3, QA-4; CON-6; regression review of QA-1, QA-2, QA-5. Evidence: deployment/failure views and load/failover test plans. Confirm SLA accounting, RTO/RPO, peak traffic and the provisional QA-4 interpretation of at most 20% average-latency increase at ten times daily volume.

### Iteration 5: Delivery and readiness

Refine artifact promotion, environment configuration, independent integration tests, observability, rollback and migration procedures. Evaluate adding a query protocol without modifying the pricing core. Consolidate open risks and empirical evidence from earlier iterations.

Drivers: QA-6 through QA-9; CON-3, CON-4; CRN-3 through CRN-5; validation of QA-1 through QA-5. Evidence: pipeline/environment views, test-element inventory, telemetry completeness criteria, protocol-extension assessment and release criteria. Distinguish design sufficiency from measured acceptance.

## Implementation Timeline

Relative dates start at an unspecified project kickoff. This planning envelope needs team-capacity validation; implementation and design overlap, and evidence can reopen earlier decisions.

| Window | Design focus | Implementation milestone |
|---|---|---|
| Month 1 | Iterations 1–2 | Establish proprietary Git repository and initial pipeline; implement the authenticated configuration/pricing/query skeleton. |
| Month 2 | Iteration 3 and early Iteration 4 risk analysis | Demonstrate MVP within two months: configuration, rate setup, simulation, price changes and queries with scoped authorization. Exercise CMS delivery with a test endpoint or clearly identified integration mock. |
| Months 3–4 | Complete Iteration 4 and revisit earlier findings | Run representative load/failure tests and integrate external test systems. |
| Months 5–6 | Iteration 5 and closure of gaps | Complete staging, recovery, deployment and security validation; deliver initial release within six months. |

MVP breadth is proposed, not an approved scope commitment. Integration may use mocks; staging must use test versions of all external systems, including for its sprint demonstrations. Begin quality validation before the final release window.

## Alignment with Development Team

| Responsibility area | Capability | Coordination |
|---|---|---|
| Browser workflows | Existing Angular expertise | Agree simulation, validation and authorization behavior. |
| Pricing/configuration | Existing Java expertise and domain knowledge | Own calculation examples, invariants and visibility contracts. |
| Identity/integration | Java, cloud identity and REST integration | Agree permissions, CMS acknowledgements and recovery semantics. |
| Platform/quality | Cloud operations, delivery automation and testing; availability unconfirmed | Own environments, observability, load/failure testing and evidence. |

These areas do not imply separate teams or services. Team size, cloud vendor and budget remain unknown. Assign actual people after capacity review and track unresolved decisions explicitly to limit technical debt.
