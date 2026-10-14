# Hotel Pricing System - Iteration 5

Completed on 2026-10-14 as an ADD design iteration. The user explicitly authorized execution through the last iteration without review pauses. Completion means the design steps and analysis are documented, not that the system is implemented or its quality targets verified.

## Iteration Goal

Complete delivery, independent integration testing, operational readiness and consolidated assessment of the complete design.

## Drivers to Address

QA-6, QA-7, QA-8, QA-9; CON-3, CON-4; CRN-3, CRN-4, CRN-5; release validation of QA-1, QA-2, QA-3, QA-4, QA-5.

## ADD Process Steps

### Step 1: Review inputs

Use the refined application, publication and deployment views from Iterations 1-4. No application implementation or live cloud environment exists in this task. Record tests as planned evidence, never as passing results.

### Step 2: Establish the goal by selecting drivers

Follow Iteration 5 of [iteration-plan.md](iteration-plan.md). Complete delivery, independent integration testing, operational readiness and consolidated assessment of the complete design. Priorities and assumptions remain traceable to [architectural-drivers.md](../architectural-drivers.md).

### Step 3: Choose elements to refine

| Element | Refinement scope |
| --- | --- |
| Repository and deployment pipeline | Immutable artifacts, runtime config, promotion and compatible rollback. |
| All application and external adapters | Complete independently executable integration inventory and protocol-extension experiment. |
| Telemetry export and operational procedures | Measure reconciliation, backlog handling, restoration and legacy cutover. |
| Team work packages and risk register | Named ownership dependencies and acceptance closure. |

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
| --- | --- | --- | --- |
| Build once and promote identical digests | Reproducible artifact across environments; satisfies no code changes. | Runtime config schemas must be validated; secrets managed separately. | Environment-specific rebuilds: artifact drift. |
| Independent adapter suites plus real staging contracts | Tests run without external systems while staging checks actual compatibility. | Simulators must be maintained; passing mocks is insufficient. | Live-only tests: fragile; mock-only tests: miss real contracts. |
| Expand/contract schema evolution | Application rollback remains possible during compatibility window. | Temporary dual-version support and cleanup work. | Destructive migration with reverse rollback: data loss risk. |
| Durable telemetry reconciliation | Repairs collector gaps and duplicate exports. | Cannot magically recover all request metrics during total infrastructure loss. | Sampled traces as complete audit source. |
| Shadow comparison and single-writer cutover | Limits pricing divergence and duplicate CMS sends during replacement. | Legacy export and delivery-state access required. | Unreconciled big-bang switch or concurrent independent writers. |

### Step 5: Instantiate elements, sketch views and define interfaces

| Instantiation decision | Rationale |
| --- | --- |
| Pipeline on existing proprietary Git platform builds UI and Java artifacts once | Integration, staging and production receive identical digests with different runtime configuration. |
| Angular startup configuration document plus secret-store injection for backend | Front-end deployment does not require environment-specific builds; public configuration contains no secrets. |
| Isolated local DB, identity simulator, CMS simulator and telemetry sink | All identified components can run integration tests independently of live external systems. |
| Future gRPC adapter experiment invokes the existing query port | Verify domain/application source remains unchanged; packaging and adapter code may change. |
| Expand/contract migrations, rollback rehearsals and staged legacy cutover | Protect accepted changes and maintain a single write authority. |
| Validation plan and final per-driver assessment | Remaining business decisions and runtime evidence are visible rather than hidden by design completion. |

S7 documents protocol extension; S8 documents independent tests and promotion. Sections 8-9 define telemetry, runtime config, release/rollback, cutover and final assessment. The companion validation plan maps tests to every scenario and element. The consolidated result is [architecture.md](architecture.md); later iterations refine these views without introducing separate conflicting architecture documents.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
| --- | --- | --- | --- |
| QA-7; CON-3; CRN-5 | D12: Immutable promotion and runtime config | No environment-specific code changes. | Per-environment branches and rebuilds. |
| QA-9, QA-6 | D13: Adapter suites and extension test | Proves intended testability and protocol boundary when implemented. | Only live external tests or only simplistic mocks. |
| QA-8; CRN-4 | D14: Durable metric sources and reconciliation | Account for each accepted publication after export outages. | Sampled traces alone. |
| CON-4; CRN-3, CRN-4 | D15: Work packages, compatible migrations and staged cutover | Makes delivery and ownership risks reviewable. | Unrehearsed big-bang replacement. |

These decision IDs are recorded in section 9 of the architecture document.

### Step 7: Analyze the design and review goal achievement

| Driver | Analysis result |
| --- | --- |
| QA-6, QA-7, QA-9 | Satisfied at design level: protocol boundary, immutable promotion and complete element inventory; experiments not executed. |
| QA-8 | Partially satisfied: durable ledger/export design exists; completeness scope, retention and outage test needed. |
| CON-3, CRN-5 | Satisfied at design level: existing Git platform and pipeline/rollback architecture specified. |
| CON-4, CRN-3 | Partially satisfied: staged delivery and work areas established; staffing, scope estimates and named owners unknown. |
| CRN-4 | Partially satisfied: explicit decisions, compatibility and risk triggers; unresolved contracts/policies remain. |
| QA-1, QA-2, QA-3, QA-4 | Partially satisfied: mechanisms and validation experiments documented; measurements and acceptance definitions remain. |
| QA-5 | Satisfied at design level: enforced identity/scope policy; real-provider and adversarial tests still required. |

Final iteration complete as an architecture-design activity. No further ADD iteration is scheduled by this task. Implementation, stakeholder acceptance of open policies and empirical verification remain explicit follow-on work.

Validation experiments are recorded in [validation-plan.md](validation-plan.md). Satisfied at design level means sufficient decisions for the architectural intent; measurements and implementation conformance are separate evidence.
