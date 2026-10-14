## Business Case

AD&D Hotels is a mid-sized business hotel chain (currently around 300 hotels) which has been experiencing robust growth in recent years. The IT infrastructure of the company is composed of many different applications such as a Property Management System, a Commercial Analysis System, an Enterprise Reservation System, a Channel Management System and, at the center of this system of systems, is the Hotel Pricing System, as seen in the following context diagram:

```mermaid
flowchart TD
    subgraph System
        EndUser[End User] -->|Sends price changes| HPS
        HPS -->|Prices| PropertyManagementSystem[Property Management System]
        HPS -->|Prices| ChannelManagementSystem[Channel Management System]
        HPS -->|Prices| CommercialAnalysisSystem[Commercial Analysis System]
        HPS -->|User credentials| UserIdentityService[User Identity Service]
        HPS -->|Prices| OtherSystems[Other Systems]
    end
```  
Context diagram for the Hotel Pricing System
   
The Hotel Pricing System (HPS) is used by sales managers and commercial representatives to establish prices for rooms at specific dates for the different hotels in the company. Prices are associated with different rates (for example a public rate, or a discount rate) and most of the prices for the different rates are calculated by taking a base rate and applying business rules to it (although some of the rates can also be fixed and not depend on the base rate). Managers typically change prices of the base and fixed rates, and using this information, the Hotel Pricing System calculates the prices of all the rates for all rooms in all hotels which also vary according to the types of rooms available in each hotel. Prices that are calculated by the HPS are used by other systems in the company to make reservations and they are also sent to different online travel agencies through the Channel Management System (CMS). The company’s systems are hosted with a cloud provider which offers a user identity service that manages users and provides Single Sign On functionalities.

AD&D Hotels wants to modernize its IT infrastructure, the first step being the complete replacement of the existing pricing system which was developed several years ago and is suffering from reliability, performance, availability and maintainability issues, and this has resulted in financial losses. Furthermore, the company has experienced difficulties because many of its systems are connected using traditional SOAP and REST request-response endpoints: changes to one application frequently impact other applications and complicate the deployment of individual updates to specific applications. Also, the failure of a particular application can propagate through the entire system. Furthermore, some of the applications interact using what are now well-known anti-patterns such as integration through a shared database. Recently revised Enterprise Architecture principles within the organization are mandating a migration of the system toward a more decoupled model.  

As part of the modernization effort, AD&D hotels also wants to integrate Agile (specifically Scrum) and DevOps practices in the development of the Hotel Pricing System. Artifacts in the development process move through four different environments:

-   Development: This is a local environment on the developers’ computers
-   Integration: This is an environment in the cloud where an integrated version of the HPS is tested. In this environment, the system is not connected with all of the external systems and so some of these external systems are substituted by mocks.
-   Staging: This is an environment in the cloud where the system’s final tests (including load testing) are performed prior to deployment. Here the system is connected to test versions of all the external systems. At the end of a sprint, the system is typically demonstrated from this environment.
 -   Production: This is the real-world execution environment.   

##  System Requirements

Requirement elicitation activities had been previously performed, and the following is a summary of the most important requirements collected.

### Primary functionality

The Hotel Pricing System’s functionality is conceptually simple; the main user stories for the system are shown using a Use Case diagram in `figure 8.2.jpg` (Figure 8.2).


Figure 8.2: Initial Use Case Diagram for the Hotel Pricing System

  

Each of these user stories is described in the following table:

|User story  |Description  |
|--|--|
|HPS-1: Log In  |A user (commercial or administrator) provides their credentials in a login window. The system checks these credentials against a user identity service and, if successful, provides access to the system. Once logged in, a user can only make queries and changes to the hotels for which they have been authorized. |
|HPS-2: Change Prices|A user selects a specific hotel for which they are authorized to change prices and selects particular dates where they want to make price changes either to a base rate or a fixed rate. All of the prices for the rates that are calculated from the base rate are calculated at that point. The system allows price changes to be simulated before they are actually changed. When the prices are changed, they are pushed to the Channel Management System and they become available for querying by external systems.|
|HPS-3: Query Prices|A user or an external system queries prices for a given hotel through the user interface or a query API.|
|HPS-4: Manage Hotels|An administrator adds, changes or modifies hotel information. This includes editing the hotel’s tax rates, available rates, and room types.|
|HPS-5: Manage Rates|An administrator adds, changes or modifies rates. This includes defining the calculation business rules for the different rates.|
|HPS-6: Manage Users|An administrator changes permissions for a given user.|
 

### Quality attribute scenarios

In addition to the above use cases, a number of quality attribute scenarios were elicited and documented. The seven most relevant and important ones are presented in the following table. For each scenario we also identify the user story that it is associated with.


| ID  | Quality Attribute  | Scenario  | Associated User story  |
|---|---|---|---|
|QA-1|Performance|A base rate price is changed for a specific hotel and date during normal operation, the prices for all the rates and room types for the hotel are published (ready for query) in less than 100 ms.|HPS-2|
|QA-2|Reliability|A user performs multiple price changes on a given hotel. 100% of the price changes are published (available for query) successfully and they are also received by the channel management system.|HPS-2|
|QA-3|Availability|Pricing queries uptime SLA must be 99.9% outside of maintenance windows.|All|
|QA-4|Scalability|The system will initially support a minimum of 100,000 price queries per day through its API and should be capable of handling up to 1,000,000 without decreasing average latency by more than 20%.|HPS-3|
|QA-5|Security|A user logs into the system through the front-end. The credentials of the user are validated against the User Identity Service and, once logged in, they are presented with only the functions that they are authorized to use.|All|
|QA-6|Modifiability|Support for a price query endpoint with a different protocol than REST (e.g. gRPC) is added to the system. The new endpoint does not require changes to be made to the core components of the system.|All|
|QA-7|Deployability|The application is moved between non production environments as part of the development process. No changes in the code are needed.|All|
|QA-8|Monitorability|A system operator wishes to measure the performance and reliability of price publication during operation. The system provides a mechanism that allows 100% of these measures to be collected as needed.|HPS-2|
|QA-9|Testability|100% of the system and its elements should support integration testing independently of the external systems.|All|


### Constraints

Finally, a set of constraints on the system and its implementation were collected. These are presented in the following table.

| ID  | Constraint  |
|---|---|
|CON-1|Users must interact with the system through a web browser in different platforms Windows, OSX, and Linux, and different devices.|
|CON-2|Manage users through cloud provider identity service and host resources in the cloud.|
|CON-3|Code must be hosted on a proprietary Git-based platform that is already in use by other projects in the company.|
|CON-4|The initial release of the system must be delivered in 6 months, but an initial version of the system (MVP) must be demonstrated to internal stakeholders in at most 2 months.|
|CON-5|The system must interact initially with existing systems through REST APIs but may need to later support other protocols.|
|CON-6|A cloud-native approach should be favored when designing the system.|


### Architectural concerns

Since this is greenfield development, only a few general concerns were identified initially and these are shown in the following table.

| ID  | Concern  |
|---|---|
|CRN-1|Establish an overall initial system structure.|
|CRN-2|Leverage the team’s knowledge about Java technologies and the Angular framework.|
|CRN-3|Allocate work to members of the development team.|
|CRN-4|Avoid introducing technical debt.|
|CRN-5|Set up a continuous deployment infrastructure.|

##  Priorities

The primary user stories were determined to be:

* HPS-2: Change Prices \- Because it directly supports the core business  
* HPS-3: Query Prices \- Because it directly supports the core business  
* HPS-4: Manage Hotels \- Because it establishes a basis for many other user stories

The scenarios for the HPS have been prioritized as follows:

| Scenario ID | Importance to the customer | Difficulty of implementation according to the architect |
| :---- | :---- | :---- |
| QA-1 \- Performance | High | High |
| QA-2 \- Reliability | High | High |
| QA-3 \- Availability | High | High |
| QA-4 \- Scalability  | High | High |
| QA-5 \- Security | High | Medium |
| QA-6 \- Modifiability | Medium | Medium |
| QA-7 \- Deployability | Medium | Medium |
| QA-8 \- Monitorability | Medium | Medium |
| QA-9 \- Testability | Medium | Medium |

From this list, QA-1, QA-2, QA-3, QA-4 and QA-5 are selected as primary drivers.

## ADD Preparation: Input Review

Reviewed against the current source on 2026-10-14. The following interpretations support design planning; they do not amend the requirements above.

| Item | Finding and working interpretation | Confirmation needed |
|---|---|---|
| Priorities | HPS-2, HPS-3, HPS-4 and QA-1 through QA-5 remain primary drivers. Supporting login, rates and permissions are necessary for a usable pricing workflow. | Agree MVP scope and available team capacity. |
| Scenario inventory | Nine scenarios are listed although the introduction says seven. Preserve all nine identifiers and priorities. | Editorial correction only. |
| QA-1 | Publication means prices are ready for query; CMS receipt is separately addressed by QA-2. | Define timing start, percentile, normal workload and maximum rate/room counts for the 100 ms target. |
| QA-2 | Provisionally interpret 100% as no lost accepted changes and eventual CMS receipt after recoverable failures. | Agree acceptance and acknowledgement semantics, delivery delay, duplicate handling and recovery window. Permanent external outage cannot guarantee receipt. |
| QA-3 | Query availability target is 99.9% outside maintenance. | Define accounting period, maintenance exclusions, dependency treatment, RTO and RPO. |
| QA-4 | Wording says latency must not decrease by more than 20%; provisionally plan for no more than a 20% increase at ten times daily volume. | Confirm wording, baseline latency, peak concurrency and traffic distribution. |
| QA-5 | Enforce hotel scope and allowed operations at the server as well as in the browser. | Define external caller identities, administrator scope and permission revocation behavior. |
| QA-8, QA-9 | All publication measures and independently testable elements need explicit inventories. | Define telemetry completeness, retention and test coverage criteria. |
| Integration | Initial REST compatibility can coexist with decoupling by isolating external contracts and failure handling. | Confirm CMS capabilities and Enterprise Reservation System query contract; the latter is mentioned in prose but not separately drawn in the context diagram. |
| Delivery | Cloud vendor, team size, budget and project kickoff are unspecified. | Use relative milestones and avoid vendor or staffing commitments. |

The supplied use-case figure was reviewed: administrators inherit commercial-user capabilities. Some source punctuation is corrupted; original requirement text is preserved. Skill references to the singular driver filename resolve to this existing plural filename, and the canonical plan path is `HPS/design/iteration-plan.md`.

## Domain Model

This conceptual model represents business concepts rather than services or database tables. Price versions, publications and delivery tracking are proposed concepts for reasoning about reliability; storage and messaging mechanisms remain undecided.

Preparation baseline: the completed ADD iterations refine these concepts in [the architecture document](design/architecture.md), especially sections 4 and 8. Original requirements above remain unchanged; final design coverage and unresolved acceptance details are recorded in that document and its validation plan.

```mermaid
classDiagram
    class Hotel
    class RoomType
    class HotelRate {
        rateKind
    }
    class CalculationRule
    class TaxPolicy
    class Price {
        stayDate
        amount
        currency
        version
    }
    class PriceChange {
        changeId
        status
    }
    class Simulation
    class UserReference {
        identitySubject
    }
    class HotelPermission
    class Publication
    class Delivery
    class ExternalConsumer
    Hotel "1" --> "0..*" RoomType : offers
    Hotel "1" --> "0..*" HotelRate : enables
    Hotel "1" --> "0..*" TaxPolicy : applies
    HotelRate "1" --> "0..*" CalculationRule : defines
    HotelRate "0..*" --> "0..1" HotelRate : derivesFrom
    Hotel "1" --> "0..*" Price : owns
    RoomType "1" --> "0..*" Price : pricedBy
    HotelRate "1" --> "0..*" Price : pricedBy
    UserReference "1" --> "0..*" HotelPermission : holds
    HotelPermission "0..*" --> "1" Hotel : scopes
    UserReference "1" --> "0..*" PriceChange : requests
    Hotel "1" --> "0..*" PriceChange : receives
    PriceChange "1" --> "0..*" Simulation : previews
    PriceChange "1" --> "0..*" Publication : produces
    Publication "1" --> "1..*" Price : identifiesVersions
    Publication "1" --> "0..*" Delivery : tracks
    ExternalConsumer "1" --> "0..*" Delivery : receives
```

| Element | Meaning and responsibility |
|---|---|
| Hotel | Hotel configuration and authorization scope; incomplete configuration may exist during setup. |
| RoomType | A room category offered by a hotel. |
| HotelRate | A rate enabled at a hotel, classified as base, fixed or derived. Shared rate definitions across hotels require clarification. |
| CalculationRule | Rule used to derive prices; effective dates and versioning require refinement. |
| TaxPolicy | Hotel tax settings; inclusion and rounding policies remain open. |
| Price | A versioned amount for a hotel, room type, rate and stay date, with explicit currency. |
| PriceChange | Requested base or fixed price edits and their acceptance state. |
| Simulation | A preview of proposed changes without authoritative price or publication side effects. |
| UserReference | Reference to a cloud-managed identity, without ownership of credentials. |
| HotelPermission | Allowed user operations within a hotel. |
| Publication | Proposed record identifying price versions committed by a change. |
| Delivery | Proposed record of publication receipt status for an external consumer. |
| ExternalConsumer | CMS or another authorized consuming system; query-only consumers need no delivery record. |

Proposed invariants: prices reference dimensions belonging to the same hotel; derived rates depend on a base without cycles; fixed rates do not derive from a base; publishing requires valid room/rate configuration; simulations never publish; authorization covers every affected hotel. Currency rules, tax treatment, calculation order, concurrent changes and configuration-change effects on existing prices require business clarification during refinement.
