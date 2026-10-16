# Hotel Pricing System - Validation Plan

This plan defines future implementation evidence. No load, failover, security or integration test has been executed against an HPS implementation in this design task. Documentation review is not runtime validation.

## Acceptance assumptions and decision register

| ID | Decision needed | Proposed working basis | Responsible role | Gate |
|---|---|---|---|---|
| A1 | QA-1 timing, dimensions and normal load | Ingress receipt to authoritative commit, strictly less than 100 ms for a bounded hotel/date update; record response time separately | Product owner and performance lead | Before benchmark acceptance |
| A2 | QA-2 acceptance, receipt and recovery horizon | Accepted means committed; every accepted change eventually has a durable CMS receipt under recoverable outages | Product owner and CMS owner | Before integration completion |
| A3 | CMS idempotency, ordering and receipt lookup | Publication ID deduplication, hotel sequence enforcement and durable receipt confirmation | Integration owner and CMS owner | Before QA-2 sign-off |
| A4 | QA-3 accounting, maintenance, RTO and RPO | Proposed monthly query SLI with visible exclusions; multi-zone failover and tested backup restore | Operations and product owner | Before production readiness |
| A5 | QA-4 wording, query mix, peak and baseline | Mean latency at high load at most 1.2 times low-load baseline; provisional 10x daily-average peak | Product owner and performance lead | Before load-test sign-off |
| A6 | Currency, rounding, taxes, rule precedence and effective dates | Decimal arithmetic, explicit currency and pinned policy versions; no inferred conversion | Pricing domain owner | Before calculator implementation acceptance |
| A7 | Permission scope and revocation semantics | Current entitlements checked per request; requests already authorized may finish | Security and product owner | Before access-control sign-off |
| A8 | Measurement completeness and retention | Unsampled accepted-publication ledger, delivery intents/outcomes and replayable export; pre-persistence failures separately counted | Operations and product owner | Before QA-8 sign-off |
| A9 | Team, budget, provider and project kickoff | Responsibility areas and relative milestones; managed services chosen by capability | Engineering lead | Before committing implementation dates/cost |
| A10 | Old-system export, freeze and rollback contract | Rehearsed import/reconciliation and one authoritative writer | Migration lead and system owners | Before cutover |
| A11 | Full independent-testing inventory | Components below must run with no live external identity/CMS dependency | Quality lead | Before QA-9 sign-off |

No question is silently converted into an approved requirement. These assumptions allow design work to finish while leaving concrete acceptance decisions visible.

## Scenario validation

| Driver | Experiment | Evidence and pass condition |
|---|---|---|
| HPS-1, QA-5 | Login with valid/invalid provider identity; wrong issuer/audience, expired token, unknown key, missing session and forged scope | Valid identity only; no credential persistence; consistent 401/403; no data disclosure |
| HPS-2 | Simulate then commit base and fixed changes across rooms/rates; change inputs between preview and commit | Preview has no publication side effects; commit matches pinned inputs; stale token yields conflict |
| HPS-3 | Query a newly committed hotel/date and paginate while later changes occur | Complete committed version or later consistent version; stable snapshot cursor; explicit expiry |
| HPS-4, HPS-5 | Add/edit/retire hotels, rooms, rates, taxes and cyclic rules; activate policy with affected prices | Invalid graphs rejected; references remain valid; old amounts retain policy version; recalculation explicit |
| HPS-6, QA-5 | Administrator changes permissions; directly invoke APIs for another hotel and attempt grants beyond admin scope | Next authorization check sees new revision; cross-hotel and over-scope actions denied; audit complete |
| QA-1 | Sweep approved maximum room/rate counts under normal read/write concurrency, including hot-hotel contention | Complete sets become queryable strictly below 100 ms under agreed interpretation; retain all observations, tails and errors; no percentile substituted without agreement |
| QA-2 | Crash before/after commit, before send, after CMS receipt before local acknowledgement; drop replies, duplicate requests, expire lease during slow send | Every accepted publication remains queryable and eventually has matched CMS receipt; no duplicate logical update or stale overwrite; no task silently discarded |
| QA-3 | Kill replica, drain during rollout, interrupt a zone/database primary, fail CMS and identity, restore backup | Measured query success and outage intervals within approved SLO/RTO; data loss within approved RPO; maintenance shown separately |
| QA-4 | Compare 100k/day and 1m/day equivalent loads plus agreed peaks using identical query mix; run long enough to reach steady state | High-load mean latency no greater than 1.2 times baseline under provisional reading; throughput achieved, no concealed error increase; capture p95/p99 and saturation |
| QA-6 | Add a thin alternate query adapter such as gRPC using existing application port | Core source unchanged; same authorization, validation and results; adapter/composition-only changes |
| QA-7 | Promote exact backend and UI artifact digests through integration and staging with separate runtime config | Same digests and no code edits; validated endpoints/secrets differ as intended |
| QA-8 | Disable exporter, crash worker, repeat sends, recover backlog and reconcile database versus monitoring | One authoritative record for each accepted publication and each attempt intent; no sampled omissions; missing/unknown outcomes explicitly classified; export counts reconcile |
| QA-9 | Execute every component's integration harness with external-system networking disabled | All inventory entries testable; real database where transaction semantics matter; no dependence on live CMS or identity |

Latency instrumentation must not pretend that a timestamp written inside a transaction is its commit completion time. Measure monotonic elapsed time at the application boundary and record durable publication identity; use database-supported commit evidence or explicit recovery classification for the crash-after-commit-before-measurement window. The selected database and telemetry implementation must demonstrate this boundary before claiming 100% accurate commit-latency collection. If exact reconstruction is unavailable, record an unknown/bounded timing result and keep QA-8 partially satisfied rather than inventing a number.

For reliability reconciliation, compare sets of accepted publication IDs, committed price versions, outstanding tasks, acknowledged CMS IDs and exported measurement IDs. Duplicate attempts are expected; missing logical publications are not. Storage/backlog alarms must fire before capacity exhaustion. After a recoverable CMS outage, verify sustained drain capacity greater than incoming publication rate and measure total recovery time against A2.

## Component integration-test inventory

| Element | Test boundary and substitute | Key assertions |
|---|---|---|
| Browser shell and REST client | Browser runner with API/identity redirect simulator | Login redirect, CSRF/session behavior, error mapping and no token leakage |
| Hotel/access views | Deterministic API simulator | Scope-filtered functions and conflict display |
| Pricing/query views | Versioned API simulator | Preview versus commit distinction, version presentation and pagination |
| REST adapters | Real application with identity port double and test database | Transport validation, status mapping and request bounds |
| Identity and authorization | Deterministic provider/JWKS service, real entitlement repository | Key rotation, unknown keys, expired identity and cross-hotel denial |
| Configuration/permission service | Real test database | Version conflicts, rule graph validity, audit and permission revision |
| Calculator with pricing service | Golden business examples plus real repository integration | Decimal rounding, tax/rate ordering, preview/commit parity |
| Transaction/repository adapters | Real disposable engine matching production major version | Atomic rollback, duplicate key, concurrent commits and complete visibility |
| Query service/read adapter | Real test database with representative indexes | Snapshot semantics, bounded scans and authorization |
| Task claimant/lease repository | Real database and concurrent workers | Expired lease recovery, fencing and per-hotel order |
| Delivery coordinator/CMS adapter | Programmable HTTP simulator | Timeout, duplicate, 202, 429, 5xx, permanent errors and durable receipt |
| Attempt ledger/reconciler/exporter | Real database and failing telemetry/CMS simulators | Unknown outcomes, redrive and checkpoint replay without omission |
| Ingress/runtime/configuration | Ephemeral cloud test deployment and synthetic clients | Health routing, configuration validation, secret isolation and graceful drain |
| CI and migration workflow | Disposable integration environment and old/new application versions | Same digest promotion, compatible schema overlap and rollback rehearsal |

Pure calculation unit/property checks supplement these integration tests; they do not replace transaction or receiver-contract tests. External staging contract tests remain necessary because local doubles cannot prove a partner's actual behavior.

## Release gates and responsibilities

| Gate | Required evidence | Owner role |
|---|---|---|
| MVP by end of month 2 | Browser login, scoped hotel setup, preview/change/query and staging CMS flow; early latency and recovery results | Product and engineering leads |
| Architecture boundary gate | No transport/SDK imports in core; all outbound integrations behind ports; versioned contracts | Module owners |
| Data/security gate | Agreed monetary semantics, authorization negative tests, audit, backup restore and migration rehearsal | Domain, security and migration leads |
| Quality gate | Agreed QA-1 through QA-5 acceptance definitions and passing measurements; QA-8 completeness reconciliation | Performance, quality and operations leads |
| Deployment gate | Same-digest promotion, rollback-compatible migrations, healthy rolling deployment and configured alerts | Platform owner |
| Production by end of month 6 | No unresolved high-priority acceptance blockers; staffing/scope feasibility reviewed; all external contracts verified | Product and engineering leads |

If acceptance evidence fails, revise the affected ADD decision and rerun that experiment. Do not conceal failed latency by weakening consistency, failed reliability by discarding work, or failed availability by excluding unplanned outages.

## Final driver traceability

| Driver group | Main decisions | Design evidence | Remaining qualification |
|---|---|---|---|
| HPS-1, HPS-6, QA-5 | D2 | Architecture 7.1 and 8; iterations 1 and 4 | Provider capabilities, revocation agreement and security tests |
| HPS-2, HPS-4, HPS-5, QA-1 | D3, D4 | Architecture 7.2-7.3; iteration 2 | Business policies and measured timing |
| HPS-3, QA-4 | D5, D8 | Architecture 7.5; iteration 4 | Workload agreement and capacity evidence |
| QA-2 | D4, D6, D12 | Architecture 7.3-7.4; iterations 2-3 and 5 | Receiver semantics and recovery/cutover proof |
| QA-3 | D8, D10 | Architecture 5 and 7.5; iteration 4 | SLO policy, fault/restore tests and RTO/RPO |
| QA-6, QA-9 | D9 | Architecture 6 and 7.6; iteration 5 | Alternate adapter and complete harness execution |
| QA-7 | D10 | Architecture 7.7; iteration 5 | Promotion demonstration |
| QA-8 | D7 | Architecture 7.4; iteration 3 | Timing completeness, retention and export evidence |
| CON-1 | D1 | Architecture 5-6; iteration 1 | Actual browser/device matrix |
| CON-2, CON-6 | D2, D8 | Architecture 5; iterations 1 and 4 | Cloud capability and cost selection |
| CON-3 | D10 | Architecture 7.7; iteration 5 | Existing platform pipeline setup |
| CON-4 | D1, D11 | iteration-plan.md | Kickoff, staffing and scope feasibility |
| CON-5 | D6, D9 | Architecture 8; iterations 3 and 5 | Negotiated REST contracts |
| CRN-1, CRN-2 | D1 | Architecture 5-6; iteration 1 | Implementation follows boundaries |
| CRN-3 | D11 | iteration-plan.md | Named ownership assignment |
| CRN-4 | D3, D4, D9, D10, D11, D12 | Architecture 9; all iteration reviews | Ongoing discipline and implementation evidence |
| CRN-5 | D10 | Architecture 7.7; iteration 5 | Actual delivery pipeline deployment |

## Residual risk summary

The primary database is a shared critical dependency and must be sized and fault-tested. Strict hotel ordering creates head-of-line blocking during an invalid CMS event. Synchronous calculation may miss QA-1 for large configurations or hot hotels. Provider identity outages affect new logins. Exact publication timing can be uncertain after a crash at commit. These are explicit design limits to validate; none is resolved merely by choosing a cloud service or adding more application replicas.
