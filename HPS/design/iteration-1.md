# Hotel Pricing System - Iteration 1

Completed on 2026-10-14 as an ADD design iteration. The user explicitly authorized execution through the last iteration without review pauses. Completion means the design steps and analysis are documented, not that the system is implemented or its quality targets verified.

## Iteration Goal

Establish the initial structure and responsibility boundaries around hotel configuration, price simulation/changes and queries.

## Drivers to Address

HPS-2, HPS-3, HPS-4; CRN-1, CRN-2, CRN-3, CRN-4; CON-1, CON-2, CON-4, CON-5, CON-6; QA-6, QA-9.

## ADD Process Steps

### Step 1: Review inputs

Reviewed the current driver document, preparation domain model, supplied use-case image and iteration plan. Primary business priorities are HPS-2, HPS-3 and HPS-4. QA-1 through QA-5 constrain the baseline. Preserve the source requirements and the provisional interpretations recorded in the preparation review.

### Step 2: Establish the goal by selecting drivers

Follow Iteration 1 of [iteration-plan.md](iteration-plan.md). Establish the initial structure and responsibility boundaries around hotel configuration, price simulation/changes and queries. Priorities and assumptions remain traceable to [architectural-drivers.md](../architectural-drivers.md).

### Step 3: Choose elements to refine

| Element | Refinement scope |
| --- | --- |
| Whole Hotel Pricing System | Greenfield refinement of browser interaction, domain/application workflows, owned persistence and external integration. External systems remain collaborators, not redesign targets. |

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
| --- | --- | --- | --- |
| Modular Java application with Angular UI | Fits known team skills; local transactions; few deployment units. | Application modules need dependency enforcement; releases share a codebase. | Fine-grained microservices: distributed transaction and operations costs are premature. |
| Ports and adapters | Replaceable protocols and independently testable external dependencies. | More interface and mapping code. | HTTP and provider types in the pricing core: conflicts with QA-6. |
| HPS-owned managed relational store | Versioned configuration and atomic price publication share one transaction. | Database becomes a common capacity and availability dependency. | Shared external database: violates decoupling; multiple stores: unnecessary dual writes. |
| Separate asynchronous CMS worker | Isolates external latency and outages from query/change response paths. | Requires durable handoff and delivery-state operations. | Synchronous CMS call in price request: couples availability and latency. |

### Step 5: Instantiate elements, sketch views and define interfaces

| Instantiation decision | Rationale |
| --- | --- |
| Browser, command application, query interface, delivery worker and owned database responsibilities | Separate user, pricing and integration concerns without prematurely distributing domain ownership. |
| Framework-neutral pricing/query ports and REST adapters | Protect core logic from protocol changes and enable test doubles. |
| Responsibility areas for front end, domain/application, integration/security and platform/quality | Allocate work by contract while team size remains unknown. |

Architecture sections 2, 4, 5 and 6 establish context, domain and structural views. S2 and S3 identify the principal change/query collaboration; section 8 defines the corresponding ports. The consolidated result is [architecture.md](architecture.md); later iterations refine these views without introducing separate conflicting architecture documents.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
| --- | --- | --- | --- |
| HPS-2, HPS-3, HPS-4; CRN-1, CRN-2; CON-1, CON-4 | D01: Angular plus modular Java application | Core workflows fit a bounded initial application. | Independent service per use case. |
| QA-6, QA-9; CON-5 | D02: Ports and REST adapters | Decouple protocols and tests from core behavior. | Transport types throughout core. |
| CON-2, CON-6; CRN-4 | D03: Owned managed relational persistence | Local consistency without external table coupling. | Shared integration database. |

These decision IDs are recorded in section 9 of the architecture document.

### Step 7: Analyze the design and review goal achievement

| Driver | Analysis result |
| --- | --- |
| HPS-2, HPS-3, HPS-4 | Partially satisfied: responsibility boundaries identified; transaction and configuration behavior refined in Iteration 2. |
| CRN-1, CRN-2 | Satisfied at design level: initial structure and Java/Angular roles established. |
| CRN-3, CRN-4 | Partially satisfied: work packages and boundaries defined; owners and unresolved risks remain. |
| CON-1, CON-2, CON-5, CON-6 | Satisfied at design level: browser, cloud hosting/identity and isolated REST contracts. |
| CON-4 | Partially satisfied: limited deployment complexity supports the plan; staffing and estimates unknown. |
| QA-6, QA-9 | Partially satisfied: adapter seams established; extension experiment and complete test inventory follow in Iteration 5. |

Refine the application, authorization and store transaction boundary in Iteration 2. Do not yet claim quality targets have been met.

Validation experiments are recorded in [validation-plan.md](validation-plan.md). Satisfied at design level means sufficient decisions for the architectural intent; measurements and implementation conformance are separate evidence.
