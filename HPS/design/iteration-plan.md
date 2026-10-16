# Hotel Pricing System - Iteration Plan

Prepared on 2026-10-16 from the supplied architectural drivers and use-case figure. This is an architecture design plan, not a staffing or delivery commitment. All five ADD iterations are documented sequentially in this directory. Implementation and validation remain future work.

## Iteration Plan Overview

| Iteration | Goal | Drivers to address |
|---|---|---|
| 1 | Establish the system structure and an authorized hotel-to-price vertical slice | CRN-1, CRN-2, CRN-3, CRN-4; HPS-1, HPS-2, HPS-3, HPS-4; CON-1, CON-2, CON-4, CON-6; QA-5 |
| 2 | Define deterministic simulation and atomic, low-latency publication | HPS-2, HPS-3, HPS-4, HPS-5; QA-1, QA-2; CRN-4 |
| 3 | Make CMS delivery durable and observable through failures | HPS-2; QA-2, QA-8; CON-5; CRN-4 |
| 4 | Refine query availability, scaling and complete authorization | HPS-1, HPS-3, HPS-6; QA-3, QA-4, QA-5; CON-2, CON-6 |
| 5 | Complete protocol extensibility, isolated testing and environment promotion | QA-6, QA-7, QA-9; CON-3, CON-4, CON-5; CRN-2, CRN-3, CRN-4, CRN-5 |

## Detailed Iteration Goals

### Iteration 1: Initial structure

Refine the whole HPS into a browser application, modular Java application, durable data store and delivery worker. Establish hotel ownership, identity integration, application ports and work ownership. Deliver diagrams and the security foundation before pricing refinement. The domain model is in architecture section 4.

### Iteration 2: Pricing correctness and publication

Refine the pricing module and persistence boundary. Define money, rules, configuration versions, simulation, concurrent writes, atomic query visibility and a durable publication record. Exit with a complete change/query contract and an explicit QA-1 measurement boundary; no performance claim is accepted without a benchmark.

### Iteration 3: Reliable integration

Refine publication delivery into recoverable work with REST adapters, receiver acknowledgement, idempotency and reconciliation. Exit with failure sequences, durable measurement records and the CMS capability dependencies needed to assess QA-2.

### Iteration 4: Operational resilience and access

Refine deployment, query capacity, scoped authorization and permission revocation. Exit with a multi-zone topology, workload hypotheses, query isolation and failure exercises. CMS failure must not sit on the query critical path.

### Iteration 5: Delivery readiness

Refine protocol ports, environment configuration, test doubles, continuous delivery and migration. Exit with a release validation plan, responsibility allocation and traceability for every driver. Review unresolved high-priority scenarios before implementation release gates.

## Implementation Timeline

Time is relative to an unspecified project kickoff. Architecture work precedes or accompanies the corresponding implementation; iteration numbers are not sprint numbers. Staffing and budget must be confirmed before committing to dates.

| Period | Implementation objective | Design inputs and exit evidence |
|---|---|---|
| Weeks 1-2 | Repository, delivery skeleton, hotel configuration and login | Iteration 1; cloud test environment and authorization smoke tests |
| Weeks 3-6 | Simulation, base/fixed edits, derived rates and queries | Iteration 2; correctness and early latency benchmarks |
| Weeks 7-8, within 2 months | Demonstrable MVP including CMS delivery against staging test system | Iterations 3 and early 4; authorized hotel-to-CMS flow, retry and query demonstration; missing external access explicitly reported |
| Months 3-4 | Complete administration, operational resilience and capacity tuning | Iteration 4; permissions, failover, recovery and load evidence |
| Months 5-6 | Full contract testing, migration rehearsal and release hardening | Iteration 5; all release gates in validation-plan.md |

MVP includes basic HPS-1 through HPS-6 to support the prioritized HPS-2, HPS-3 and HPS-4 workflows. Rich administration UX and additional protocols can follow the MVP; security and durable acceptance cannot.

## Alignment with Development Team

| Responsibility | Expertise | Coordination |
|---|---|---|
| Browser experience | Angular and accessible browser UI | Agrees API contracts with application owner |
| Pricing and configuration | Java, domain modeling, transactional SQL | Owns rule semantics and simulation parity |
| Integration | Java, REST contracts, retries and reconciliation | Agrees acknowledgement behavior with CMS owner |
| Platform and quality | Cloud operations, CI/CD and failure testing | Owns environment templates and release evidence |

These are responsibility areas, not four assumed teams or fixed headcounts. Individuals may cover multiple areas. Each module has a named owner assigned at kickoff; contract changes require consumer review. No additional vendor platform or orchestration cluster is assumed.
