# Hotel Pricing System - Iteration 5

Complete the design with protocol extensibility, independent integration testing, artifact promotion and replacement-system cutover.

## Iteration Goal

Make the proposed architecture implementable and verifiable across environments within the stated delivery constraints, with no hidden claims that design alone proves runtime quality.

## Drivers to Address

QA-6, QA-7, QA-9; CON-3, CON-4, CON-5; CRN-2, CRN-3, CRN-4, CRN-5. Review all earlier primary-driver gaps before closing design work.

## ADD Process Steps

### Step 2: Establish goal by selecting drivers

Apply iteration-plan.md iteration 5. Preserve the business and failure semantics already selected; add change isolation, delivery controls and an evidence plan that supports the six-month release.

### Step 3: Choose elements to refine

Refine application ports, REST adapters, composition root, environment configuration, build/deployment pipeline and component test harnesses. Add a migration/cutover workflow around the authoritative pricing boundary.

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
|---|---|---|---|
| Protocol-neutral core and adapter contract tests | New query protocol does not change business logic | Mapping and cross-protocol consistency tests add work | Reimplementing pricing/query policy for each transport |
| Build once and inject runtime configuration | Same code and artifacts promoted through environments | Requires configuration schema validation and secure secret delivery | Environment-specific rebuilds |
| Disposable database plus identity/CMS doubles | Repeatable tests without external-system availability | Doubles can drift from production contracts | Mocks only, or all tests requiring live partners |
| Expand/contract migrations and gradual application rollout | Supports overlap and application rollback | Destructive cleanup must be delayed | Breaking schema and application switch simultaneously |
| Snapshot import, reconciliation and controlled cutover | Avoids two authoritative writers and detects price differences | Requires export agreement and a write-freeze plan | Uncoordinated dual writes |

### Step 5: Instantiate elements, views, responsibilities and interfaces

Architecture sequences 7.6 and 7.7 define protocol extension/testing and environment promotion before their contracts in section 8. The component diagrams now expose every replaceable outbound dependency. validation-plan.md assigns component-level test coverage and release gates.

| Instantiation decision | Rationale |
|---|---|
| Keep domain DTOs independent of REST/gRPC/SDK classes | New inbound transport changes only adapters and composition wiring |
| Promote identical backend and UI digests with separately validated config | No code changes between non-production environments |
| Use existing proprietary Git platform and compatible CI runner | Meets CON-3 without replacing enterprise tooling |
| Run independent component tests, then staging tests against all external test systems | Balances repeatability and actual contract fidelity |
| Add schema compatibility and module-dependency checks to CI | Prevents accidental erosion of boundaries and rollback capability |
| Treat migration rehearsal and source-of-truth switch as release gates | Replacing HPS must preserve price correctness and delivery history |

Migration sequence: export a consistent old-system snapshot; import through a controlled loader with identifier mappings; recompute/sample and reconcile all price totals and relevant dimensions; rehearse in staging; freeze old-system edits for an agreed window; import/reconcile final changes; switch query consumers and CMS routing; then enable HPS writes. Before HPS accepts new writes, routing can return to the old system. After new writes, rollback needs explicit reverse reconciliation and delivery deduplication; never simply route back to stale data. Cutover-specific bulk data handling is outside the interactive QA-1 bound and must be separately timed.

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
|---|---|---|---|
| QA-6, QA-9, CON-5 | D9: ports, adapters and isolated contract harnesses | Maintains core stability and independent testing | Transport or external SDK dependency in core |
| QA-7, CON-3, CRN-5 | D10: immutable promotion and compatible migrations | Reproducible environments and safer rollback | Per-environment builds and destructive rolling migrations |
| CRN-2, CRN-3, CRN-4, CON-4 | D11: ownership and evidence-based milestones | Uses known skills and exposes schedule risk | Unstaffed delivery commitments |
| QA-2, CRN-4 | D12: reconciled migration and controlled cutover | Maintains a single price authority | Uncontrolled dual writes |

All decisions are recorded in architecture section 9. No vendor purchase, deployment or message to an external owner is performed as part of this design task.

### Step 7: Analyze design and review achievement

| Driver | Analysis result |
|---|---|
| QA-6 | Satisfied at design level: protocol-free query port; prove with a thin alternate-protocol adapter and unchanged core source |
| QA-7 | Satisfied at design level: identical digests and external runtime configuration; demonstrate promotion |
| QA-9 | Satisfied at design level: component inventory and replaceable external boundaries; execute all harnesses without live external systems |
| CON-3, CON-5 | Satisfied at design level: existing Git hosting and REST-first contracts preserved |
| CON-4 | Partially satisfied: staged plan supports two/six-month targets but staffing, scope and external access are unconfirmed |
| CRN-2, CRN-5 | Satisfied at design level: known stack and continuous deployment infrastructure defined |
| CRN-3 | Partially satisfied: responsibilities defined, actual named allocation pending roster |
| CRN-4 | Partially satisfied: debt controls and migration strategy exist; sustained implementation discipline is required |
| QA-1 through QA-4 | Partially satisfied at final review: designs exist; timing, receiver, workload and recovery gates remain open |
| QA-5 | Satisfied at design level with explicit access/revocation assumptions; security verification pending |
| QA-8 | Partially satisfied: publication completeness mechanism exists; collection scope and retention need confirmation |

All planned design iterations are complete. Design completion does not authorize a release or establish measured compliance. The remaining implementation tests and business decisions are consolidated in validation-plan.md, rather than hidden by marking every driver satisfied.
