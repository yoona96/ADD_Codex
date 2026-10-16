# Hotel Pricing System - ADD Preparation

## Preparation step 1: Review inputs

Reviewed architectural-drivers.md and figure 8.2.jpg on 2026-10-16. The figure confirms that administrators inherit commercial-user use cases. The existing source contains preparation text and references to design artifacts that were absent at the start of this task. This design is newly authored from the supplied inputs; the source is preserved. Skill references to architectural-driver.md resolve to the existing plural filename architectural-drivers.md.

| Input finding | Working interpretation | Required confirmation |
|---|---|---|
| Nine scenarios follow an introduction saying seven | Include QA-1 through QA-9 | Editorial only |
| HPS-2, HPS-3, HPS-4 and QA-1 through QA-5 are primary | Address these in iterations 1-4; establish security in iteration 1 | MVP scope and team capacity |
| QA-1 lacks timing boundaries and workload limits | Measure ingress receipt through database commit making the complete price set queryable; also record response latency separately | Normal workload, maximum date/room/rate dimensions and required statistical interpretation |
| QA-2 requires 100% publication and CMS receipt | Every accepted change has durable publication and delivery records; eventual receipt assumes recoverable failure and cooperative receiver | Durable acknowledgement, deduplication, ordering, outage limit and delivery deadline |
| QA-3 lacks an accounting period and recovery targets | Use eligible successful query minutes over eligible minutes, complemented by request success rate | Period, maintenance policy, RTO, RPO and dependency accounting |
| QA-4 literally limits latency decreases | Provisionally interpret as mean latency at tenfold volume no greater than 1.2 times baseline | Correct wording, peaks, query mix and baseline |
| QA-8 asks for 100% measurement collection | Persist one publication measurement and all durable delivery attempts without sampling | Retention, definition of failures before persistence and export delay |
| QA-9 says all elements support isolated integration tests | Inventory every component and replace external systems through ports and doubles | Coverage and test-fidelity acceptance criteria |
| Identity, vendor, budget and headcount unspecified | Cloud identity owns credentials; HPS owns hotel entitlements; vendor-neutral managed services | Identity capabilities, cost ceiling and team capacity |
| Pricing policies underspecified | Versioned decimal calculations with explicit currency, tax and rounding; reject invalid graphs | Rule precedence, tax inclusion, effective dating and rounding policy |
| Existing system is being replaced | Import and reconcile a consistent pricing snapshot before controlled cutover | Data ownership, export format and permitted write freeze |

These assumptions permit design progress but do not alter requirements or establish production acceptance. Decisions needing business confirmation are tracked in validation-plan.md.

## Preparation step 2: Create domain model

The conceptual model and element responsibilities are recorded in architecture.md section 4. It separates business prices from publication and delivery state. No external application accesses HPS tables.

## Preparation step 3: Create iteration plan

See iteration-plan.md. Five sequential iterations move from whole-system structure through pricing, delivery, resilience and delivery readiness. Each iteration records ADD steps 2-7; input review is the shared preparation step, not an omitted design activity.
