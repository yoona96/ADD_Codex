# Hotel Pricing System - Iteration 1

Establish the whole-system structure after the shared input review and domain modeling in preparation.md. This iteration is the starting point for iterations 2-5.

## Iteration Goal

Create an implementable foundation for authorized hotel configuration, price changes and queries within the MVP window.

## Drivers to Address

CRN-1, CRN-2, CRN-3, CRN-4; HPS-1, HPS-2, HPS-3, HPS-4; QA-5; CON-1, CON-2, CON-4, CON-6.

## ADD Process Steps

### Step 2: Establish goal by selecting drivers

Use iteration-plan.md iteration 1. Prioritize business workflows and hotel configuration while establishing authentication before exposing data. Input review is already completed in preparation.md; remaining ambiguities are explicitly carried forward.

### Step 3: Choose elements to refine

Refine the whole HPS, the only initial system element. Separate browser presentation, application responsibilities, durable state and external delivery. Treat cloud identity and enterprise consumers as external elements rather than components to redesign.

### Step 4: Choose design concepts

| Design concept | Pros | Cons | Discarded alternatives |
|---|---|---|---|
| Modular Java application and Angular UI | Uses team knowledge; small release surface; enforceable domain boundaries | Shared application release and some resource contention | Microservice per domain: premature distributed coordination for six-month delivery |
| Ports and adapters | Separates external contracts from business logic; enables isolated tests | Additional interfaces and mapping code | SDK/HTTP logic embedded in domain: change propagation |
| Cloud identity with server hotel authorization | Existing SSO; no HPS password ownership | Identity availability and entitlement integration must be managed | Custom identity store or browser-only authorization |
| Managed cloud runtime and relational database | Reduces operational setup; supports atomic publication | Cloud cost and service limits require sizing | Self-managed orchestration cluster without demonstrated need |

### Step 5: Instantiate elements, views, responsibilities and interfaces

Architecture sections 2, 4, 5 and 6 establish context, domain, containers and application components. Sequence 7.1 defines login and sequence 7.2 defines initial hotel configuration; their contracts appear in section 8. Pricing internals and delivery remain responsibilities to refine in iterations 2 and 3.

| Instantiation decision | Rationale |
|---|---|
| Browser shell, configuration views and pricing/query views call REST adapters | Satisfies browser constraint with clear presentation ownership |
| Java application contains authorization, configuration, pricing and query modules | Keeps core business coordination local and assigns distinct ownership |
| Cloud identity subjects link to HPS hotel permissions | Authentication and business authorization have different owners |
| Separate delivery worker is an initial container responsibility | External CMS outages must not occupy browser request processing |
| Team allocation uses responsibility areas, not assumed headcounts | Staffing is unknown; individuals may own multiple areas |

### Step 6: Record design decisions

| Driver | Decision | Rationale | Discarded alternatives |
|---|---|---|---|
| CRN-1, CRN-2, CON-4 | D1: modular Java and Angular | Familiar, small initial deployment surface | Fine-grained distributed services |
| HPS-1, QA-5, CON-2 | D2: provider login and server scopes | Prevents credential duplication and cross-hotel access | Local passwords, UI-only access checks |
| CRN-3, CRN-4 | D11: module ownership and contract review | Makes work allocation and change impact explicit | Implicit shared ownership and undocumented interfaces |

These decisions are recorded in architecture section 9. Managed runtime selection is a service-category decision, not vendor procurement.

### Step 7: Analyze design and review achievement

| Driver | Analysis result |
|---|---|
| CRN-1 | Satisfied at design level: explicit context, containers and component responsibilities |
| CRN-2 | Satisfied at design level: Java and Angular preserve known expertise |
| CRN-3 | Partially satisfied: allocation model exists; named owners require actual team roster |
| CRN-4 | Partially satisfied: boundaries and review discipline established; ongoing enforcement is required |
| HPS-1, QA-5 | Partially satisfied: identity and scope boundary selected; full permission and revocation behavior refined in iteration 4 |
| HPS-2, HPS-3 | Partially satisfied: workflow ownership and ports exist; pricing transaction and query consistency follow in iteration 2 |
| HPS-4 | Satisfied at design level: scoped, versioned configuration responsibility and workflow defined |
| CON-1, CON-2, CON-6 | Satisfied at design level: browser, cloud identity and managed cloud resources selected; platform/browser validation remains |
| CON-4 | Partially satisfied: design supports small MVP scope; schedule depends on capacity and external access |

The initial structuring goal is achieved. Carry pricing semantics and latency to iteration 2, delivery reliability to iteration 3 and complete authorization/availability to iteration 4. No implementation evidence is claimed.
