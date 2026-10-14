# Hotel Pricing System - Validation and Open Decisions

This plan accompanies the completed ADD design in [architecture.md](architecture.md). All experiments below are **planned, not executed**. Documentation consistency checks do not establish application correctness, latency, reliability or availability.

## Acceptance decisions still needed

These items do not block completion of the design documents. They must be resolved by the indicated responsibility area before implementation contracts or release acceptance are finalized. The user requested no further questions during this design task.

| ID | Decision | Provisional design position | Responsible area / latest useful point |
|---|---|---|---|
| O01 | QA-1 measurement boundary, percentile and normal workload. | Ingress to query-visible durable commit; p99 below 100 ms is an engineering hypothesis, not a replacement for the original bound. | Product/domain and performance owners; before performance acceptance criteria. |
| O02 | CMS receipt, idempotency, ordering and timeout behavior. | Stable publication IDs, durable receipts and stale-sequence rejection; unknown results reconcile before later hotel deliveries. | Integration owner and CMS owner; before enabling automatic delivery. |
| O03 | Delivery delay, recoverable-failure envelope and retention. | Eventual receipt after recoverable failures; alert after five minutes, retain all unacknowledged changes. | Product/integration/operations; before delivery SLO and storage sizing. |
| O04 | QA-3 accounting period, exclusions, dependency treatment and disaster recovery. | Query availability includes database/ingress interruptions; illustrative monthly accounting, five-minute single-zone recovery and zero acknowledged loss. Region-wide targets unset. | Product/platform; before infrastructure choice and SLA agreement. |
| O05 | QA-4 latency wording and realistic peak distribution. | At most 20% mean latency increase at tenfold daily volume; provisional 12/116 requests per second peaks. | Product/performance; before capacity acceptance. |
| O06 | Currency, tax inclusion, rounding, rule vocabulary and effective dates. | Decimal arithmetic and explicit versioned configuration; reject incomplete policy configuration. | Domain/product; before calculation implementation is accepted. |
| O07 | Multi-date atomicity and configuration activation scope. | Individually atomic hotel/date changes with explicit partial outcomes. | Product/domain; before UI and command contracts freeze. |
| O08 | Provider identity capabilities, external caller grants and administrator scope. | Authorization code with PKCE for browser; workload identity for machines; DB hotel/operation grants. | Security/integration; before identity integration. |
| O09 | Required QA-8 measures and completeness/retention definition. | Durable committed-publication and delivery records; sampled traces are supplemental; total-infrastructure-loss measurement limits explicit. | Product/operations; before monitoring acceptance. |
| O10 | Cloud service capabilities, budget, staffing and kickoff date. | Existing provider, managed runtime/relational state, Java/Angular; relative two/six-month milestones. | Architecture/platform/project owner; before delivery commitments. |
| O11 | Legacy export, consumer routes and pending-delivery migration. | Shadow comparison, freeze/reconcile and single write authority. | Integration/project owner; before cutover rehearsal. |
| O12 | Retry/idempotency history retention and replay behavior. | Proposed 90-day result retention; retain key tombstones beyond result removal and reject expired replays. | Domain/operations; before retention jobs are enabled. |

## Functional and invariant verification

| Experiment | Drivers | Setup and stimulus | Required observation |
|---|---|---|---|
| V01 - Login and scopes | HPS-1, HPS-6, QA-5 | Provider simulator then staging provider; valid/expired/wrong-audience tokens, key rotation, cross-hotel reads/writes, non-admin grant edits and revocation races. | Unauthorized calls never disclose or mutate another hotel's data; permissions take effect under documented transaction-start/write-lock semantics. |
| V02 - Deterministic calculation | HPS-2, HPS-5 | Versioned base/fixed/derived rules, room factors and explicit tax/rounding fixtures; simulation then commit with unchanged versions. | Preview and committed matrix match; no floating-point drift; fixed rates unchanged by unrelated base edits. |
| V03 - Rule/configuration validation | HPS-4, HPS-5 | Cycles, missing base, arbitrary code, unknown currency/tax treatment, invalid date ranges and retired rates. | Invalid drafts cannot activate; complete errors are returned; historical prices retain original versions. |
| V04 - Concurrency and atomic visibility | HPS-2, HPS-3, QA-1, QA-2 | Concurrent edits with same expected version, query continuously during commit, race activation and change. | One conflicting command succeeds and the other conflicts; readers see whole old/new matrices; no mixed version rows. |
| V05 - Simulation isolation | HPS-2 | Simulate repeatedly, then compare DB publication/current-price/outbox state; commit after intervening change. | Simulation makes no pricing/delivery writes; stale commit conflicts rather than publishing stale preview. |
| V06 - Idempotency | HPS-2, QA-2 | Duplicate same key/body, same key/different body, concurrent duplicates, lost response after commit and expired key. | Exactly one logical committed change; replay returns original authorized result, mismatch conflicts, expired replay is rejected. |
| V07 - Multi-date and activation UX | HPS-2, HPS-4, HPS-5 | Several child changes, one conflict or validation failure; activate a draft over a date range. | UI shows each outcome; no claim of all-date success or atomicity; each committed child has its own publication. |
| V08 - Query contract | HPS-3 | Date/rate/room filters, missing prices, maximum bounded response, over-limit query, pagination across updates. | Consistent response snapshot, versions/metadata present, explicit missing results and documented cross-page semantics. |

A sample calculation fixture may use base 100.00, a 10% derived discount and fixed rate 80.00 with no tax: derived price is 90.00 and fixed remains 80.00. This is a test example only; tax and rounding examples must come from agreed O06 policies, especially where rounding intermediate values differs from rounding the final amount.

## Quality experiments

| Experiment | Drivers | Procedure | Acceptance evidence |
|---|---|---|---|
| V09 - Publication latency | QA-1 | Measure ingress-to-commit and first subsequent query visibility across agreed room/rate cardinalities, rule complexity and normal concurrent load. Include synchronous HA and hot-hotel edits. | Full distribution, maximum, errors, locks and transaction timing. Compare against O01; no result yet. |
| V10 - Publication crash matrix | QA-2 | Kill application before/after commit; kill worker before send, after CMS receipt and before ledger commit; interrupt DB connection. | Reconcile all acknowledged application changes to unique publications and pending/received CMS IDs. No lost accepted changes. |
| V11 - CMS failure and ordering | QA-2 | Simulate timeout after receipt, duplicate/late requests, 202 pending, 429, 5xx, permanent rejection, worker lease expiry and paused old worker. | Every change retained; same ID replay; no stale overwrite; per-hotel order; other hotels continue; unsupported CMS behavior surfaces as unresolved delivery. |
| V12 - Backlog recovery | QA-2, QA-3 | Disconnect CMS, continue changes and queries, restore CMS; approach configured retention/storage alarm thresholds. | Queries unaffected by CMS latency; backlog drains in sequence, no coalescing or loss; admission limits activate before storage exhaustion. Recovery rate exceeds incoming change rate. |
| V13 - Availability and recovery | QA-3 | Remove query instance/zone, fail over DB, interrupt identity/telemetry/CMS independently, restore backup in isolated environment. | Timed query availability and downtime, fenced writer, restored state reconciliation and actual RTO/RPO; outage accounting follows O04. |
| V14 - Tenfold query capacity | QA-4 | Same payload/hotel distribution at 100,000 and 1,000,000 daily-equivalent rates, warm-up then steady measurement; provisional peaks 12 and 116 requests/s, 250 burst. Repeat with hot-hotel skew and concurrent writes. | Mean ratio at most 1.2 only if O05 adopted; also absolute latency, p95/p99, throughput/error rates, cost and DB saturation. Record hardware and scaling policy for both runs. |
| V15 - Security negative paths | QA-5 | UI bypass, token replay/expiry/key rotation, revoked scopes, machine-user separation, input attacks, secret/log inspection. | Authorization policy enforced in every adapter; safe error handling; no secrets in bundle/logs; provider outage behavior documented. |
| V16 - Protocol extension | QA-6 | Add test gRPC adapter through the existing query port and execute the query contract suite. | Domain/application source unchanged; only adapter, wiring, packaging and protocol tests differ; same auth semantics. |
| V17 - Artifact promotion | QA-7, CON-3, CRN-5 | Promote exact UI/image digests through integration and staging; change only runtime config and secret references. Rehearse rollback after additive migration. | Same artifacts verified; no code edits/rebuild; environment separation and backward compatibility hold. |
| V18 - Telemetry completeness | QA-8 | Compare ledger counts and timings with exported metrics; stop collector/exporter, restart at checkpoint, force duplicate export and interrupted attempt. | Each committed publication/attempt represented once under agreed measure semantics; unknown outcomes visible; gaps recover. Whole-system-loss limits recorded. |
| V19 - Disconnected integration | QA-9 | Disable access to all live external endpoints and execute every element suite below. | Tests run independently; no accidental live traffic; failures remain diagnosable. Staging contracts separately detect simulator drift. |
| V20 - Browser compatibility | CON-1 | Responsive workflows on agreed supported browsers across Windows/macOS/Linux and selected mobile devices, with keyboard navigation. | Usable layout, login redirects, forms, conflict handling and queries at each viewport. Matrix still needs product agreement. |
| V21 - Legacy cutover | CON-4, CRN-4 | Rehearse import, shadow comparison, write freeze, final reconcile, consumer switch and pending-delivery transfer. Rehearse reverting after new accepted changes. | One write authority, no duplicate CMS sends, no lost new changes, timed rollback with data reconciliation. |

Run single-zone and total-region failure cases separately: backups alone do not establish zero loss of every acknowledged change across regional disasters. A load run that throttles or errors above target cannot be reported as successful capacity merely because successful responses remain fast.

## Independent integration-test inventory

| Element | Local or isolated dependency | Contract and failure coverage |
|---|---|---|
| Angular shell/identity client | Local identity protocol simulator and browser automation | Redirects, PKCE failures, expiry, capabilities and responsive navigation. |
| Configuration/pricing/query/status UI | Local REST contract server | Valid/error/conflict/partial outcomes, versions and pending CMS state. |
| REST and future protocol adapters | In-process application with test principals | Serialization, bounds, status/error mapping and principal propagation. |
| Authentication/authorization | Local signing-key/identity simulator and test DB grants | Issuer/audience, rotation, revocation races and cross-hotel denial. |
| Pricing engine and rule validator | Versioned fixtures; no external services | Determinism, decimal arithmetic, cycles, fixed-rate invariants and limits. |
| Change service and transaction coordinator | Isolated real relational DB instance | Locking, rollback, idempotency, atomic matrix/outbox commit and concurrency. |
| Configuration and grant service | Isolated DB and rule engine | Drafts, activation, permission races and audit. |
| Query port and read adapter | Isolated DB with seeded grants/versions | Snapshot behavior, indexes, query bounds and pagination semantics. |
| Worker lease scheduler/dispatcher | Isolated DB, controllable clock and CMS simulator | Lease expiry, ordering, jitter, retry and crash recovery. |
| CMS adapter and reconciler | Programmable local REST CMS simulator | Durable receipts, duplicate recognition, stale sequence rejection and unknown outcomes. |
| Ledger and metric exporter | Isolated DB and local telemetry sink | Checkpoints, replay, deduplication, missing export and retention. |
| Ingress, runtime config and health endpoints | Local reverse proxy/test runtime | Routing, limits, configuration validation and readiness behavior. |
| Migrations and deployment packaging | Disposable DB and local container/runtime stack | Expand/contract compatibility, artifact digest identity and rollback. |

Provider-managed replication, identity internals and external CMS are outside the HPS implementation boundary. Their adapters support independent tests, but actual service failover and protocol compatibility require isolated cloud/staging verification. This distinction prevents QA-9 from being misread as proof that simulators establish real cloud behavior.

## Release evidence and ownership

The two-month MVP demonstrates hotel/rate configuration, scoped login, simulation, change/query and visible delivery state. Mocks must be labeled in integration demonstrations; staging uses external test systems. The six-month release requires disposition of O01-O12, execution of relevant experiments, migrated-data reconciliation, operational runbooks and documented residual risk acceptance by the appropriate owners.

Keep experiment evidence with workload/configuration versions, artifact digests, time window, raw counts and failure timelines. The domain area owns calculation fixtures; integration/security owns provider/CMS evidence; platform/quality owns load/recovery/promotion/telemetry results. Assign named owners when staffing is known. None of these planned activities has been reported as executed by this documentation task.

## Documentation checks performed

Checked that all five iteration records contain ADD Steps 1 through 7, the architecture contains the nine template sections, all 26 source driver identifiers appear in the assessment, and all 15 decision identifiers are recorded. Checked local Markdown links, balanced code fences and absence of obsolete review-pending notices. Reviewed sequence success/failure paths and normalized generated text encoding. The architecture contains 15 Mermaid diagrams; they were inspected as source, not rendered with a Mermaid engine. These checks validate document structure only, not the planned system experiments above.
