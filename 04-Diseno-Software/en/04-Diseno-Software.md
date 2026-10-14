
**Software Design**

Guardian Escolar

Software for the creation of the "Guardian Escolar" application

Johan Smith Santamaria Fernández  
Sharik Dayanna Rojas Ibarra  
Juan Pablo Chala Ramírez

SENA — Servicio Nacional de Aprendizaje  
Programming group 3145556  
Neiva Training Center (Huila)

Instructor: José de Jesús Motta Vargas  
Neiva, October 6, 2026

# Version control

**Table 1**. Document version control

| Version | Date       | Author                  | Change description            |
| ------- | ---------- | ----------------------- | ----------------------------- |
| 1.0     | 06/10/2026 | Guardian Escolar Team   | Initial version of the document |

**Table of contents**

Press F9 (or right-click → Update field) to generate the table of contents.

# 1 Introduction and design principles

## 1.1 Purpose and scope

This document specifies the **software design** of the Guardian Escolar platform: the architectural model that structures the system, the design patterns used, the web and mobile graphical interfaces, the persistent data model, and the integration contracts that link all the pieces together. The design translates the functional and non-functional requirements of the system into concrete and verifiable construction decisions.

The platform is made up of a **web application in Angular 21 with Angular Material 3**, a **mobile application in React + Expo** for driver and guardian, an **API Gateway (Kong OSS)** at the edge, **nine domain microservices**, an **Apache Kafka 4.1.0 event bus (KRaft mode)**, a low-latency **gRPC** channel, and a **single relational database SQL Server 2022** organized into schemas.

**Table 2**. Document content by section

| Design element        | Document content                                                                          | Section |
| --------------------- | ----------------------------------------------------------------------------------------- | ------- |
| Architectural model   | Context, container, deployment, and layer views; styles applied with justification        | 2       |
| Design patterns       | Catalog of verified patterns and non-adopted patterns                                     | 3       |
| Graphical interfaces  | Web screens by role, mobile screens, navigation map, and design tokens                    | 4       |
| Data model            | Global ER diagram, data dictionary by schema, enums, seeds, indexes, and rules            | 5       |
| Interface contracts   | REST resources, Kafka events, and gRPC flow                                               | 6       |
| Decisions             | Design decisions, discarded alternatives, and trade-offs                                  | 7       |

## 1.2 Design principles

The following principles govern every design decision of the system and are evaluated in the technical review of each increment.

**Table 3**. Design principles and their application

| Principle                                                     | Concrete application in the system                                                                                                                                                                                                                                        |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Low coupling and high cohesion**                            | Each microservice encapsulates a bounded context (identity, school, fleet, routes, notifications, audit, configuration, exceptional uses) with its own schema; services relate through contracts, never through direct reads of others' data                                    |
| **Single responsibility (SRP)**                               | A use case handles one nominal operation; controllers limit their work to translating HTTP into a use case and returning the response                                                                                                                                     |
| **Open for extension, closed for modification (OCP)**         | Search is extended by adding new strategies without modifying the use cases; alert types and configuration parameters are added as data, not as code                                                                                                                       |
| **Liskov substitution (LSP)**                                 | The implementations of the data ports and the remote adapters exactly fulfill the contract of their domain interface, so that tests with in-memory doubles substitute the real store without a change in behavior                                                          |
| **Interface segregation principles (ISP)**                    | Small, specific port interfaces: one port per use-case operation (ICreateRouteUseCase, IListRouteUseCase, IStopRepository) instead of monolithic ports                                                                                                                     |
| **Dependency inversion (DIP)**                                | The domain declares the ports; the infrastructure implements them and the dependency injection container wires them at startup                                                                                                                                            |
| **Dependency rule**                                           | Dependencies point inward: infrastructure → application → domain; the domain knows nothing of web frameworks, persistence engines, or messaging clients                                                                                                                  |
| **Design by contract**                                        | The REST contract is defined before implementation and the code never publishes a resource absent from the definition; events and RPC contracts are versioned additively                                                                                                   |
| **Fail fast, recover gracefully**                             | Input validation at the edge and in the service, per-component health check, idempotent consumers, and 503 responses when a critical dependency does not respond                                                                                                           |
| **Least privilege**                                           | Role- and permission-based authorization with default denial; each database service accesses only its own schema                                                                                                                                                          |
| **Observability from design**                                 | End-to-end correlation identifier, structured logs, and per-component metrics                                                                                                                                                                                             |
| **Accessibility and visual consistency**                      | WCAG AA contrast, keyboard operation, visible focus indicators, and exclusive consumption of the design system's design tokens                                                                                                                                           |

## 1.3 Design conventions

**Table 4**. Design conventions

| Area                    | Convention                                                                                                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Relational store        | Schema and table in PascalCase; column in PascalCase; foreign key column &lt;Entity&gt;Id                                                                                                |
| Identifiers             | UNIQUEIDENTIFIER in business entities; auto-increment TINYINT in catalogs; auto-increment BIGINT in audit; auto-increment INT in sessions                                             |
| Constraints             | Prefixes PK, FK_&lt;Table&gt;\_&lt;RefTable&gt;, UQ_&lt;Table&gt;\_&lt;Column&gt;, CK_&lt;Table&gt;\_&lt;rule&gt;, DF_&lt;Table&gt;\_&lt;Column&gt;                                       |
| Lifecycle               | Soft delete with Status in values Active/Inactive; interface contracts expose them as ACTIVE/INACTIVE                                                                                    |
| Interface surfaces      | REST routes under /api/v1, plural nouns in kebab-case; payloads in camelCase; uuid identifiers                                                                                           |
| Events                  | Name &lt;entity&gt;.&lt;action&gt;; message key = entity identifier; additive JSON payload with no sensitive data                                                                        |
| Service ports           | Use case interface I&lt;Verb&gt;&lt;Entity&gt;UseCase; store interface I&lt;Entity&gt;Repository; implementation in the infrastructure layer                                              |
| Time                    | Date and time stamps in DATETIME2 with UTC time zone in the new flows                                                                                                                    |
| Communication           | Versioned REST at the edge, gRPC for internal low-latency calls, and Kafka for asynchronous decoupling; diagrams in ASCII representation oriented to plain-text reading                   |

# 2 Architectural model

The architecture of Guardian Escolar is described in four complementary views: context, containers, deployment, and internal layers. The first three follow the architecture view notation (context, containers, and components) adapted to ASCII diagrams, so that the document is readable in any text medium and does not depend on images or proprietary graphical notations.

## 2.1. Context view

The context view places the system in front of its actors and external systems. Guardian Escolar stands between the educational community and two external free-tier providers: a transactional email provider and an instant messaging provider.

EDUCATIONAL COMMUNITY

+----------------+ +----------------+ +----------------+ +----------------+

| Super admin. | | Administrator | | Driver | | Guardian / |

| | | of the school | | | | Student |

+-------+--------+ +-------+--------+ +-------+--------+ +-------+--------+

| | | |

+-------------------+---------+---------+-------------------+

|

v

+--------------------------------------------+

| GUARDIAN ESCOLAR (platform) |

| School transportation service |

| operation: identification, users, fleet, |

| routes, events, notifications and audit |

+-------------------+------------------------+

|

+-------------------+-------------------+

| |

v v

+--------------------+ +--------------------+

| Email provider | | Messaging |

| (transactional) | | provider (WhatsApp)|

+--------------------+ +--------------------+

**Figure 1**. Context view of the platform

Actors and summarized responsibilities:

**Table 5**. Actors and their interaction with the platform

| Actor                          | Main interaction with the platform                                                                                                 |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| Super administrator            | Administers the global instance: organizations, users, permissions, catalogs, and system parameters.                              |
| School administrator           | Configures the school and its campuses, grades, families, drivers, routes, and schedules; consults reports and statuses.          |
| Driver                         | Consults the assigned route, records boarding events and trip statuses, runs the stop validations, and reads messages.            |
| Guardian                       | Consults the information of their family, the routes and trip statuses of their children, and sends exceptional use requests.     |
| Student                        | Accesses a minimal set of queries of their own and of their route.                                                                 |
| External messaging systems     | Receive the notification templates by email and by instant messaging.                                                              |

## 2.2. Container view

The container view identifies the deployable units of the platform. The system is a microservices architecture composed of nine domain and platform services, an API routing and aggregation step, an event bus, a relational database shared through logical schemas, a cache and refresh-token store, and a messaging service for notification templates.

+---------------------------+

| API gateway and router |

| (port 8000) |

+-------------+-------------+

|

+-----------+-----------+--------+-------+-----------+-----------+

| | | | | |

v v v v v v

+-----------+ +---------+ +---------+ +------------+ +---------+ +-----------+

| IAM | | Users | | School | | Fleet | | Routes | |Notificat. |

| (8080) | | (8080) | | (8080) | | (8080) | | (8080) | | (8080) |

+-----+-----+ +----+----+ +----+----+ +------+------+ +----+----+ +-----+-----+

| | | | | |

+-----------+ +---------+ +---------+ v |

| Uses | | Audit | | Config. | +------------+ |

|exceptional| | (8080) | | (8080) | | Event | |

| (8080) | +---------+ +---------+ | bus | |

+-----------+ | Kafka | |

| | (9092) | |

| +------+-----+ |

| | |

+----------------------+-----------------------------------+-----------+

|

v

+---------------------------------+

| Relational database |

| SQL Server (1433) |

| 11 schemas, 38 tables |

+---------------------------------+

Complementary storage: session cache and refresh.

Notification template channel: external messaging service.

**Figure 2**. Container view of the architecture

Container description:

**Table 6**. Containers of the architecture and their role

| Container                                     | Role in the architecture                                                                                                                           |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| API gateway and router                        | Single entry point, routing to the services, TLS termination, rate limiting, and CORS policy.                                                      |
| Identification and access service             | Token issuance, sign-in and sign-out, token refresh, password recovery in accordance with the current policy, and account lockout.                |
| User management service                       | People, roles, permissions, profiles, and the guardian-student relationship.                                                                       |
| School management service                     | Schools, campuses, grades, assignments, schedules, and family relationships.                                                                       |
| Fleet service                                 | Buses, makes, models, maintenances, and fleet statuses.                                                                                            |
| Routes service                                | Routes, stops, stop order, runs, boarding events, trip statuses, and reports.                                                                      |
| Notifications service                         | Templates, delivery queue, recipients, retries, and delivery status.                                                                               |
| Exceptional uses service                      | Requests, approvals, limits, and escort records.                                                                                                   |
| Audit service                                 | Audit events, filtered query, and export.                                                                                                          |
| Configuration service                         | System configuration and reference catalogs.                                                                                                       |
| Event bus                                     | Event channel between services (people creation topics, status changes, trip events, and telemetry events).                                        |
| Relational database                           | Transactional persistence, with one logical schema per service and supporting foreign keys between schemas.                                        |
| Cache store and template channel              | Low-latency cache of reference data and token refresh; output of the notification messages toward the end devices.                                 |

## 2.3. Deployment view

The deployment view shows how the containers are distributed across the execution resources. Each microservice is packaged in its own container, with its own lifecycle and its own configuration; the orchestration groups the containers and distributes the requests among instances.

INTERNET

|

v

+------------------------+

| Load balancer and |

| API gateway |

| (8080 -> 8000) |

+-----------+------------+

|

+-----------------------+------------------------+

| | |

v v v

+--------------+ +--------------+ +--------------+

| Service | | Service | | Service |

| containers | | containers | | containers |

| domain A..C | | domain D..F | | platform |

+------+-------+ +------+-------+ +------+-------+

| | |

+-----------+-----------+----------------------+

|

v

+-----------------------------+ +-----------------------------+

| Relational database | | Event bus |

| (1433) with persistent | | (9092) with persistent |

| volumes | | disks |

+-----------------------------+ +-----------------------------+

Cache and refresh: memory instance; secrets: execution-environment store;

observability: metrics and traces collected from each container.

**Figure 3**. Deployment view of the platform

Deployment considerations:

- Each service exposes its own application port (8080) and only the API gateway is reachable from the outside.
- The database and the event bus are deployed with persistent storage and are not exposed to the Internet.
- Horizontal scalability is applied per service according to its load: the routes and notifications services scale by event volume; the read services scale by number of requests.
- Credentials and secrets (database, token signing keys, email and messaging credentials) are injected from a secrets store and are never included in the code or in the deployment artifacts; trip statuses and service indicators are published with expiration timestamps, so that a power outage does not leave stale indicators.

## 2.4. Internal layer view of a service

Each service internally adopts a hexagonal architecture (ports and adapters) organized into layers. The dependency always points inward: the domain and application layers know nothing of the infrastructure.

+-------------------------------------------------------------------+

| Input adapters (primary) |

| REST controllers, event handlers, gRPC servers, |

| scheduled processes |

+-------------------------------+-----------------------------------+

| invokes use cases

v

+-------------------------------------------------------------------+

| Application (use cases) |

| Orchestration, transactions, process rules, authorization |

+-------------------------------+-----------------------------------+

| output ports

v

+-------------------------------------------------------------------+

| Domain |

| Entities, values, invariants, domain errors |

+-------------------------------+-----------------------------------+

| implemented by

v

+-------------------------------------------------------------------+

| Output adapters (secondary) |

| Data accesses, units of work, event clients, |

| gRPC clients, email and messaging adapters |

+-------------------------------------------------------------------+

**Figure 4**. Internal layer view of a service

Internal naming convention: output ports end in Repository or Gateway, use cases end in UseCase, query operations end in Query, and change operations end in Command. Domain errors are captured in the API layer and translated into problem responses with the corresponding HTTP status.

## 2.5. Architectural styles adopted and justification

**Table 7**. Architectural styles adopted and justification

| Style                                          | Where it is applied                                                                            | Justification                                                                                                               |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Microservices                                  | Entire platform: nine services with their own lifecycle                                        | Allows each domain to be deployed, scaled, and to fail independently; bounds the impact of a defect to a single service.    |
| Hexagonal architecture (ports and adapters)    | Interior of each service                                                                      | Separates business rules from infrastructure; enables unit testing of the domain without a database or network.             |
| Event orientation                              | Flows between services (people creation, status changes, trip events, telemetry)               | Decouples producers and consumers in time; allows adding new consumers without touching the producer.                      |
| Layers (n-tiers) within each service           | Input adapters, application, domain, output adapters                                           | Maintains a single dependency direction and a localizable business rule.                                                    |
| Unified API under a routing step               | External exposure                                                                              | Offers a single entry point, common security policy and rate limit, and avoids exposing internal services.                  |
| Database per service with logical schemas      | Persistence                                                                                    | Gives data autonomy to each service and, at the same time, allows supporting referential integrity within the relational engine. |

Discarded styles and why: the modular monolithic architecture was discarded because the platform foresees differentiated growth by domain (routes and notifications grow with trip volume; audit grows with action volume); the pure stateless server was qualified because the user session and the reference cache require shared state; the service-oriented architecture (SOAP) was discarded for its contract overhead and its lesser suitability for lightweight web and mobile clients.

## 2.6. Cross-cutting concerns

The following concerns are resolved transversally and are not confined to a single service:

**Table 8**. Cross-cutting concerns and design criteria

| Concern                       | Design criterion                                                                                                                                                                                                          |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security                     | Token authentication, role- and permission-based authorization, encrypted transport, secrets in a dedicated store, and validation of all input.                                                                             |
| Availability                 | Each service can be restarted without affecting the others; calls between services have a time limit and bounded retry; notification send failures are retried with an exponential backoff policy.                          |
| Observability                | Correlation identifier propagated in requests and events, per-service metrics, end-to-end traces, and structured logging.                                                                                                  |
| Consistency                  | Local transactions within each service and eventual consistency between services through compensating events when a distributed operation fails.                                                                          |
| Contract evolution           | Explicit API versioning in the route (/api/v1), backward compatibility in the event topics, and versioning of the RPC service in its definition.                                                                          |
| Performance                  | Indexes aligned with the listing queries, mandatory pagination in lists, projection of only the necessary fields, and cache for reference data that changes little.                                                        |
| Accessibility                | Minimum contrast of 4.5:1 for text, 3:1 for graphical elements, full keyboard navigation, and screen reader support.                                                                                                       |
| Operation traceability       | Every relevant action generates an audit event with actor, action, entity, timestamp, and origin.                                                                                                                          |

# 3 Design patterns applied

The patterns in this section were verified in the implementation of the system. The table presents them with their intent, the place where they are applied, and the concrete mechanism that materializes them.

## 3.1. Verified patterns

**Table 9**. Verified design patterns

| Pattern                                        | Intent in the system                                                    | Verified application                                                                                                                                     |
| ---------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hexagonal architecture (ports and adapters)    | Keep the domain independent of the infrastructure                        | Each service separates input and output ports from its adapters; the domain depends on neither frameworks nor databases.                                |
| Repository (data access)                       | Centralize data access and abstract the persistence engine               | Output ports typed by entity, with implementations over the service's ORM; the listing queries are in the implementations.                               |
| Unit of work                                   | Guarantee atomicity in operations that touch several entities            | Unit-of-work interface with explicit commit; use cases open a transaction, execute, and commit.                                                          |
| DTO and mapper                                 | Decouple the public contract from the internal model                     | Input and output data transfer objects with automatic mapping profiles between DTO, domain, and presentation model.                                      |
| Problem handler (ProblemDetails)               | Return normalized and documented errors                                  | Single point of translation of internal errors into problem responses, with HTTP status, type, detail, and field extension.                              |
| Search strategy                                | Compose filterable queries without scattered conditionals                | Chainable criteria chains (school, campus, grade, status, dates, free text) combinable according to the need of each listing.                          |
| Use case facade                                | Expose a single, testable operation per business case                    | Each use case (create person, assign route, approve exceptional use, register boarding) is a class with one public operation.                             |
| Observer / domain events                       | React to changes without coupling to the producer                        | Person creation, trip status change, and telemetry events are published to topics and consumed by other services.                                        |
| Remote client adapter                          | Hide the transport from one service to another behind an interface       | Typed clients of the RPC service with a shared channel, retries, and transformation of errors into domain errors.                                        |
| Dependency injection / managed factory         | Compose objects without manual factories                                 | The execution container builds controllers, use cases, data accesses, and clients; clients are registered with a shared lifetime policy.                  |
| Hosted service                                 | Run background jobs within the service lifecycle                         | Background processes for sending notifications and queue processing, with orderly cancellation when the service stops.                                  |
| Output adapter interface                       | Substitute infrastructure without touching rules                         | Dedicated ports for email sending, messaging sending, and event publication, with implementations interchangeable per environment.                       |
| Compensation in distributed operation          | Undo partial effects when an operation between services fails            | Sequences with compensating steps on inter-service call failures and explicit commit at the end.                                                        |
| Catalog-based extension point                  | Allow growing in behavior without modifying code                         | The configuration catalogs (alert types, statuses, reasons) are parameterized and consumed as data, not as a fixed condition.                            |

## 3.2. Patterns evaluated but not adopted

**Table 10**. Design patterns evaluated but not adopted

| Pattern                                                   | Reason for non-adoption                                                                                                                                          |
| --------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Decorator                                                 | The cross-cutting decoration of requests is resolved in the API gateway (rate limiting, headers, CORS) and in service middleware, without wrapping the domain.   |
| CQRS (explicit command and query separation)              | The read volume does not justify duplicating model and persistence; queries are optimized with projections, indexes, and pagination.                            |
| Orchestrated saga with a dedicated component              | The few distributed operations are resolved with events and local compensation; introducing an orchestrator would add a single point of failure.                |
| Service mesh                                              | The number of services and the current traffic do not justify the operational overhead of a sidecar in each container.                                          |
| Enriched result model (result wrapper)                    | Errors are returned with the operation's own HTTP status and a normalized problem body, with no additional wrapper in the success body.                        |

# 4 Graphical interfaces

The platform is delivered in two surfaces: a management web application and a mobile application for consultation and field recording. The web targets the administrative profiles (super administrator and school administrator) and the public screens; the mobile targets driver, guardian, and student, who operate from the vehicle or from the road. Both surfaces share tokens, components, and validation rules, so that the visual identity and behavior are a single system.

## 4.1. Interface principles

**Table 11**. Interface principles and design consequences

| Principle                      | Design consequence                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Operation first                | On the field screens, the main action (start trip, register boarding) is always one tap away and with a minimum target size of 44 pixels.              |
| Scannable information          | Lists and panels show the decisive datum first (status, time, route) and the detail in a modal or information view.                                     |
| Confirmation before destroying | Every deletion or irreversible change goes through a confirmation dialog with the explicit consequence.                                                 |
| Actionable errors              | Each error is shown next to the affected control and explains how to correct it; never only the problem code is reported.                              |
| Accessibility from design      | Label associated with each control, predictable tab order, minimum contrast met, and states never communicated by color alone.                          |
| Consistency between surfaces   | Same tokens, same validation rules, and same confirmation patterns on web and mobile.                                                                   |

## 4.2. Web interfaces by role

### 4.2.1. Public space (no session)

The public surface comprises the home page (presentation of the service) and sign-in. The home page communicates the product's capabilities (real-time monitoring, alerts, attendance control, administration panel, application for families, secure data) and institution registration.

+--------------------------------------------------------------+

| \[≡\] GUARDIAN ESCOLAR \[ Contact \] \[ Sign in \] <- 56px

+--------------------------------------------------------------+

| School safety in real time |

| Real-time GPS · Instant alerts · Attendance |

| Administration panel · App for families · Secure data |

| Registration: \[Institution name\] \[Email\] \[Message\] |

| \[ Register \] |

+--------------------------------------------------------------+

**Figure 5**. Sketch of the public home page

+--------------------------------------------------------------------------------+

| \[ ← \] GUARDIAN ESCOLAR |

+--------------------------------------------------------------------------------+

| +--------------------------------+ |

| | Sign in | |

| | Email | |

| | \[\_**\_**\_**\_**\_**\_**\_**\_**\] | |

| | Password | <- password policy |

| | \[\_**\_**\_**\_**\_**\_**\_**\_**\] | |

| | Forgot your password? | <- link to recovery |

| | \[ Sign in \] | <- var(--button-apply) |

| | (invalid credentials | |

| | message, if applicable) | |

| +--------------------------------+ |

+--------------------------------------------------------------------------------+

**Figure 6**. Sketch of the sign-in form

### 4.2.2. Super administrator role

The super administration panel concentrates the global view of the instance: organizations, schools, administrators, and recent alerts.

+--------------------------------------------------------------------------------+

| GUARDIAN ESCOLAR \[ Theme ▾ \] \[ Language ▾ \] \[ Profile ▾ \] 56px |

+-------------------+------------------------------------------------------------+

| | Welcome |

| PANEL | |

| +---------------+----------------------------------+---------------------+ |

| | Schools | \[ Cards: schools, users, | | |

| | Administrator | campuses and routes \] | | |

| | | Latest alerts | | |

| | | +----------------------------+ | | |

| | | | time | type | school | | | |

| | | +----------------------------+ | | |

| | | | ... | ... | ... | | | |

| | | +----------------------------+ | | |

| | | \[ View all \] | | |

| +---------------+-----------------------------------+---------------------+ |

| 200px sidebar with active items highlighted |

+-------------------+------------------------------------------------------------+

**Figure 7**. Sketch of the super administration panel

School and administrator management repeats the same module template (list + information modal + edit modal + delete dialog):

+--------------------------------------------------------------------------------+

| \[←\] Schools \[ + New school \] |

+--------------------------------------------------------------------------------+

| \[ Search by name.......... \] Filter: \[Status ▾\] Results: 12 |

+--------------------------------------------------------------------------------+

| Name | City | Campuses | Status | Actions |

| San José | Bogotá | 3 | Active | \[i\] \[✎\] \[🗑\] |

| Norte Abierto | Trujillo | 1 | Inactive | \[i\] \[✎\] \[🗑\] |

+--------------------------------------------------------------------------------+

| &lt; 1 2 3 ... 6 &gt; |

+--------------------------------------------------------------------------------+

\[i\] -> information modal (RecordInformation); \[✎\] -> edit modal; \[🗑\] -> delete dialog (DeleteRecord)

**Figure 8**. Sketch of the school management module

### 4.2.3. School administrator role

The administrative panel groups the operations of the school: people (users, parents, drivers, families), transport (buses, stops, routes), and profile.

+--------------------------------------------------------------------------------+

| GUARDIAN ESCOLAR \[ Theme ▾ \] \[ Language ▾ \] \[ Profile ▾ \] |

+-------------------+------------------------------------------------------------+

| USERS | Welcome, administrator |

| - Users | |

| - Parents | +----------------+ +----------------+ |

| - Drivers | | Students | | Buses | |

| - Families | | 120 | | 20 | |

| ----------------| +----------------+ +----------------+ |

| TRANSPORT | +----------------+ +----------------+ |

| - Buses | | Routes | | Stops | |

| - Stops | | 20 | | 60 | |

| - Routes | +----------------+ +----------------+ |

| ----------------| |

| | Latest school alerts |

| | +----------------------------------------------------+ |

| | | 08:12 Route deviation Route 04 \[View\] | |

| | | 07:55 Boarding not registered Route 02 \[View\] | |

| +---------------+--------------------------------------------------------+ |

+-------------------+------------------------------------------------------------+

**Figure 9**. Sketch of the school administrator panel

Student registration form (the application's form template; the other forms share the structure):

+--------------------------------------------------------------------------------+

| Register student |

| +--------------------------------------------------------------------------+ |

| | First name \* \[\_**\_**\_**\_**\_**\_\] | Last name \* \[\_**\_**\_**\_**\_**\_\] | |

| | Doc. type \* \[Document ▾\] | Doc. no. \[\_**\_**\_**\_**\___\_\] | |

| | Birth date \* \[yyyy-mm-dd \] | Gender \[\_**\_**\___\__ ▾\] | |

| | School \* \[School ▾\] | Campus \* \[Campus ▾\] | |

| | Grade \* \[Grade ▾\] | Group \[Group ▾\] | |

| | Guardian \* \[Search person ....\] | Status \* \[Active ▾\] | |

| | | |

| | \[ Cancel \] \[ Save \] | |

| +--------------------------------------------------------------------------+ |

| (inline errors below each field, only when the control was touched) |

+--------------------------------------------------------------------------------+

**Figure 10**. Sketch of the student registration form

### 4.2.4. Profiles with limited web access (driver, guardian, student)

The field profiles keep a reduced session space on the web: personal information, account data change, language, and theme. The route consultation operations and field records are resolved in the mobile application.

+--------------------------------------------------------------------------------+

| \[←\] My profile |

+--------------------------------------------------------------------------------+

| +---------------------------+ +------------------------------------------+ |

| | (avatar) | | Full name | |

| | | | Email address | |

| | First name Last name | | Contact phone | |

| | Role | | Account status | |

| +---------------------------+ | \[ Edit information \] | |

| +------------------------------------------+ |

| Account security |

| - Email change -> four-step flow with code |

| - Password change -> three-step flow with six-length code |

| - Phone or contact change -> four-step flow with code |

| Preferences: Language \[Spanish ▾\] Theme \[Light blue ▾\] |

+--------------------------------------------------------------------------------+

**Figure 11**. Sketch of the profile with limited web access

Flow for changing a sensitive datum (email, phone) in four steps with visible confirmation and return to the profile; the password change applies the password policy and uses three steps:

Step 1 Step 2 Step 3 Step 4

+--------------+ +------------------+ +---------------+ +----------------+

| Request | -> | Validate code | -> | Enter the | -> | Validate final |

| the change | | of six | | new value | | code |

| (current data)| | characters | | | | |

+--------------+ +------------------+ +---------------+ +----------------+

**Figure 12**. Four-step flow for changing a sensitive datum

### 4.2.5. Matrix of components and validations per form

**Table 12**. Matrix of components and validations per form

| Screen / form                     | Interface components                                                                                  | Validations applied                                                                                                                                                                                                                   |
| --------------------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Sign-in                           | Email field, password field, primary button, inline message                                          | Email with valid format; the password policy requires at least eight characters with uppercase, lowercase, number, and special character; the credentials error is shown in the view and does not reveal which of the two data failed. |
| Access recovery (step 1)          | Email field, primary button                                                                           | Email with valid format; generic response for account existence.                                                                                                                                                                      |
| Access recovery (step 2)          | Six segmented fields, primary button                                                                  | Alphanumeric code of length six; focus and tab order preserved between the six fields; inline error.                                                                                                                                  |
| Access recovery (step 3)          | Password field, confirmation field, primary button                                                    | match of both passwords; the password policy is applied with a minimum of eight characters, uppercase, lowercase, number, and special character; validation on submit with all controls marked as touched.                          |
| Student registration              | Identity fields, school/campus/grade/group selectors, guardian search, status selector                | Requirement of the fields marked with an asterisk; document with no duplicates in the school; existing guardian with parent role; non-future birth date.                                                                             |
| Driver registration               | Identity fields, license, phone, status selector                                                      | Requirement; phone in international format; license validity informed before accepting.                                                                                                                                               |
| Bus registration                  | Plate, make, model, capacity, internal number                                                         | Requirement; plate with no duplicates; positive integer capacity.                                                                                                                                                                     |
| Route registration                | Name, origin, destination, start time, end time, stops in order                                        | Requirement; end time later than the start time; at least one stop; unique stop order.                                                                                                                                                |
| Stop registration                 | Name, address, latitude, longitude                                                                    | Requirement; latitude between −90 and 90; longitude between −180 and 180.                                                                                                                                                             |
| Email or contact change           | Email or phone field, code fields                                                                     | Format of the new datum; code of length six; two-round verification to avoid the use of the previous datum.                                                                                                                          |
| Authenticated password change     | Current password field, new passwords, code                                                           | Password policy applied to the new password; not matching the current password; code of length six.                                                                                                                                  |
| Record deletion                   | Confirmation dialog                                                                                   | Explicit confirmation of the destructive action; in records with dependencies, warning of the affected entities.                                                                                                                     |

## 4.3. Mobile interfaces

The mobile application is organized into three zones: top bar, content area, and bottom navigation. The screens cover sign-in, the driver's route operation, the guardian's consultation, and account administration.

Sign-in:

+---------------------------+ +---------------------------+

| GUARDIAN ESCOLAR | | Recover access |

| Email | | Email |

| \[\_**\_**\_**\_**\_**\_**\_\] | | \[\_**\_**\_**\_**\_**\_**\_\] |

| | | \[ Continue \] |

| Password | | | <- password policy

| \[\_**\_**\_**\_**\_**\_**\_\] | | +---------------------+ |

| | | | | | | | | | | |

| Forgot your | | |six-character | |

| password? | | |segmented code | | <- length six

| \[ Sign in \] | | +---------------------+ |

| | | \[ Validate \] |

+---------------------------+ +---------------------------+

**Figure 13**. Mobile sketches of sign-in and recovery

Driver: route operation with boarding registration and online sync notice:

+---------------------------+ +---------------------------+

| \[≡\] My route \[🔔\] | | \[≡\] My route \[🔔\] |

+---------------------------+ +---------------------------+

| Ruta Centro | | Ruta Centro |

| Departure 07:10 4 of 12 | | Next stop |

| +---------------------+ | | +-----------------------+ |

| | Next stop: | | | | Colegio San José | |

| | Colegio San José | | | | 07:42 (6 min left) | |

| | 07:42 1.2 km | | | +-----------------------+ |

| +---------------------+ | | |

| | | Students at this stop |

| Stop list | | \[x\] Juan García 07:41 |

| 1. Av. Norte 07:15 OK | | \[ \] María Ruiz -- |

| 2. Calle 12 07:28 OK | | \[ \] Carlos Silva -- |

| 3. Colegio S.J. 07:42 .. | | |

| 4. La Finlandia 07:55 | | \[ Register boarding \] |

| \[ Register boarding \] | | (primary button, 44px) |

+---------------------------+ +---------------------------+

**Figure 14**. Mobile sketch of the driver's route operation

Driver offline state and trip monitor:

+---------------------------+ +---------------------------+

| \[←\] My route \[No network\] | | \[←\] Trip in progress \[⏸\] |

| 3 pending changes | | |

+---------------------------+ | Status: EN ROUTE |

| | | Start: 07:10 |

| Records will be saved | | Stops completed: 4 |

| and synced when | | Students on board: 18 |

| the connection returns. | | |

| | | \[ Pause \] \[ Finish \]|

| Pending: | | |

| - Boarding 07:42 Colegio SJ | | (finishing requires |

| - Status 07:55 La Finlandia| | confirmation) |

| | +---------------------------+

| \[ Retry sync \] |

+---------------------------+

**Figure 15**. Mobile sketch of the offline state and trip monitor

Guardian: family and live route:

+---------------------------+ +---------------------------+

| \[≡\] My family \[🔔\] | | \[≡\] Route \[🔔\] |

+---------------------------+ +---------------------------+

| Policyholder | | Ruta Centro |

| +---------------------+ | | +---------------------+ |

| | Ana Gómez Guardian| | | | Route map | |

| +---------------------+ | | | (with summary in | |

| | | | | alternative text) | |

| Other members | | +---------------------+ |

| +---------------------+ | | |

| | Juan García Student | | | Next arrival: 07:52 |

| | Ruta Centro \[View\] | | | Stop: Calle 12 |

| +---------------------+ | | Status: EN ROUTE |

| +---------------------+ | | |

| | María Ruiz Student | | | Stops |

| | No route -- | | | 1. Av. Norte 07:15 ✓ |

| +---------------------+ | | 2. Calle 12 07:28 ✓ |

| | | 3. Colegio S.J. 07:42 |

| +---------------------+ | +---------------------------+

| | Exceptional use | |

| | request \[Send\] | |

| +---------------------+ |

+---------------------------+

**Figure 16**. Mobile sketch of the guardian's family and route

## 4.4. Navigation map

WEB

|

+-- / (redirect) ----------------------------------------> /home

|

+-- /home ................................ public (+ /contact)

|

+-- /auth

| +-- /login ..................................... public

| +-- /recover-access

| +-- /email ...... request (password policy)

| +-- /code ....... validation of six-length code

| +-- /reset ...... new password (password policy)

|

+-- /admin-panel ................... SUPER_ADMIN, ADMIN

| +-- /admin/users, /parents ................. ADMIN

| +-- /admin/drivers, /families ............... ADMIN

| +-- /admin/buses, /admin/stops, /admin/routes ......... ADMIN

| +-- /admin/profile .................... SUPER_ADMIN, ADMIN

| +-- /email-change and /contact-change: data -> /code-first -> /reset -> /code-second

| +-- /password-change: /email -> /code -> /reset

|

+-- /superadmin-panel .............. SUPER_ADMIN

| +-- /superadmin/admins and /superadmin/schools SUPER_ADMIN

|

+-- /\* (undefined route) ------------------------------> /home

MOBILE

|

+-- Access: login and recover access (email -> code -> password policy)

|

+-- Driver

| +-- My route (stops, next stop, boarding)

| +-- Trip in progress and offline (pending, retry)

| +-- Notifications

|

+-- Guardian

| +-- My family (account holder, members, exceptional use)

| +-- Route (map with text summary, stops, status)

| +-- Notifications and my data

|

+-- Student: route and own data lookup

|

+-- Shared account: profile, security, appearance, language, about,

privacy policy and sign-out with confirmation

**Figure 17**. Platform navigation map

Navigation rules: the root redirects to the home page; an undefined route redirects to the home page; data-change flows are step-by-step sequences with explicit return to the profile; the administrator panel and the super administrator panel are distinct routes with their own side menu; access to each panel is decided by role on the server side, not only by the interface.

## 4.5. Design tokens

Design tokens are the single source of visual values: components reference variables, not loose colors or sizes.

Base palette:

**Table 13**. Base palette of design tokens

| Token                | Value   | Use                                          |
| -------------------- | ------- | -------------------------------------------- |
| \--bg-color          | #F0F4F8 | General background of the views              |
| \--card-bg           | #FFFFFF | Background of cards and modals               |
| \--card-secondary-bg | #F8FAFC | Secondary background of cards                |
| \--text-color        | #111827 | Main text                                    |
| \--title-color       | #1A2E4A | Titles                                       |
| \--text-secondary    | #64748B | Secondary text and metadata                  |
| \--border-color      | #E2E8F0 | Borders and separators                       |
| \--color-navbar      | #1A56DB | Top bar and side panel                       |
| \--accent-color      | #003AB8 | Links and emphasis                           |
| \--hover-color       | #2061EC | Pointer hover state                          |
| \--button-apply      | #1A56DB | Primary button (apply, save, continue)       |
| \--button-cancel     | #FFFFFF | Secondary button (cancel, back)              |
| \--alert-bg          | #FEF2F2 | Alert background                             |
| \--alert-title       | #7F1D1D | Alert title                                  |
| \--alert-text        | #374151 | Alert text                                   |
| \--alert-icon        | #DC2626 | Alert icon                                   |
| \--input-bg          | #FFFFFF | Background of form fields                    |
| \--input-border      | #CBD5E1 | Border of form fields                        |
| \--modal-bg          | #FFFFFF | Background of dialogs                        |
| \--mat-sys-surface   | #FFFFFF | Material system surface                      |

Themes (eight combinations; the navbar token and the primary button token change together):

**Table 14**. Theme combinations and color tokens

| Theme              | \--color-navbar | \--button-apply |
| ------------------ | --------------- | --------------- |
| light-theme-blue   | #1A56DB         | #1A56DB         |
| light-theme-green  | #16A34A         | #16A34A         |
| light-theme-yellow | #D4A017         | #D4A017         |
| light-theme-red    | #B42318         | #B42318         |
| dark-theme-blue    | #0F2E6B         | #3B82F6         |
| dark-theme-green   | #0B4A24         | #22C55E         |
| dark-theme-yellow  | #B8860B         | #FACC15         |
| dark-theme-red     | #5B1215         | #DC2626         |

Typography:

**Table 15**. Typography values

| Use                   | Value                                  |
| --------------------- | -------------------------------------- |
| Base family           | "Segoe UI", Arial, sans-serif          |
| Material family       | Roboto                                 |
| Bar title             | 20px, weight 500                       |
| Form title            | 1.5rem                                 |
| Secondary text        | 0.85rem to 1rem, color --text-secondary |
| Labels                | 0.78rem, weight 500, uppercase         |
| Illustration icons    | 1.5rem                                 |

Spacing, radii and shadows:

**Table 16**. Spacing, radii and shadows

| Use                              | Value                                   |
| -------------------------------- | --------------------------------------- |
| Spacing scale                    | 4px, 8px, 12px, 16px, 24px, 32px        |
| Compact controls                 | radius from 5px to 10px                 |
| Cards and modals                 | radius of 16px                          |
| Existing form buttons            | radius of 50px                          |
| Cards                            | shadow 0 4px 12px rgba(0, 0, 0, 0.08)   |
| Modals                           | modal's own shadow (--shadow-modal)     |

Structure and behavior:

**Table 17**. Structure and behavior of the components

| Element                                                      | Value                                                                                                                                                           |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Top bar                                                      | height 56px, background --color-navbar, title 20px                                                                                                              |
| Side panel                                                   | width 200px, background --color-navbar; items with radius 8px and 2px separation; hover with 8% white overlay; active with 15% white overlay and weight 500      |
| Material density                                             | 0                                                                                                                                                               |
| Material palette                                             | primary Azure type, tertiary blue type, Roboto typography                                                                                                       |
| Contrast                                                     | 4.5:1 for text and 3:1 for graphic elements                                                                                                                     |
| Touch target (mobile)                                        | minimum 44px                                                                                                                                                    |
| Breakpoint of the public side panel                          | hidden above 1024px                                                                                                                                             |
| Breakpoint of the information panel in session flows         | hidden below 1000px                                                                                                                                             |
| Forms and cards                                              | go from two columns to one on small screens                                                                                                                     |

## 4.6. Interface states

**Table 18**. Interface states and visual patterns

| Situation              | Visual pattern                                                                                                        |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Form error             | Inline message below the field, invalid border and typed error block                                                  |
| Invalid credentials    | Message visible in the access view without distinguishing which piece of data failed                                  |
| Confirmation           | Confirmation dialog with the consequence of the action                                                                |
| Loading and success    | Progress indicator or control disabled during the request; message visible before the return navigation.              |
| Offline (mobile)       | Visible badge, counter of pending changes and sync retry action                                                       |
| Empty (lists)          | Empty state with the creation action as the next step                                                                 |

## 4.7. Accessibility and adaptation

- Every control has an associated label; placeholder text does not replace the label.
- The tab order follows the visual order and is preserved in the segmented code fields.
- No state is communicated by color alone: it is always accompanied by text or an icon.
- The live route map is accompanied by a textual summary of the position, so as not to depend on the graphic representation.
- Standalone icons receive an accessible label or a tooltip.
- Horizontal scrolling and overlapping controls are not allowed at any breakpoint, and text can be enlarged without loss of content or overlaps.

# 5 Data model

The platform is persisted in a single relational database on SQL Server 2022 with eleven logical schemas, one per domain context, and thirty-eight tables in total. Each service accesses only its own schema; the foreign keys that cross schemas exist to guarantee referential integrity within the engine, not to enable direct reads between services. The physical schema is maintained with versioned migrations that are applied in a deterministic order on every startup.

Model conventions:

**Table 19**. Data model conventions

| Convention               | Applied rule                                                                                                                                                                                            |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Primary key              | Global unique identifier UNIQUEIDENTIFIER in business entities; auto-incrementing TINYINT in small catalogs; auto-incrementing BIGINT in the audit log and in the error log.                             |
| Foreign keys             | Declared as a referential integrity constraint; some are created together with the table and others are added separately to respect the creation order between schemas.                                 |
| Dates and times          | DATETIME2 for date and time stamps, DATE for expirations and TIME for schedules.                                                                                                                        |
| Geographic coordinates   | DECIMAL(10,7) for latitude and longitude, with sufficient precision for vehicle tracking.                                                                                                               |
| Lifecycle                | Status column with values ACTIVE and INACTIVE (soft delete) in almost all entities; physical deletion is reserved for test data.                                                                        |
| Nullability              | NULL is declared only when the data is optional by nature (end-of-validity date, website, second confirmation code).                                                                                    |
| Domain restrictions      | Inline checks for statuses, document types, days of the week and boarding type; uniqueness on documents, plates, IMEI, device codes and license numbers.                                                |

## 5.1. Global ER diagram

+--------------------------------------------------------------+

| Geographic: City |

+----------------------------+---------------------------------+

| CityId (City 1 : N School)

+----------------------------v---------------------------------+

| School: School, SchoolCampus, Course, SchoolAdmin |

+-------------------+--------------------+---------------------+

| CampuseId (1 : N) | PersonId (1 : 1)

+-------------------v--------------------v---------------------+

| UserManagement: Person, Family, FamilyMember, DriverLicense |

+-------------------+--------------------+---------------------+

| PersonId (1 : 1) | ProfileId (1 : N)

+-------------------v--------------------v---------------------+

| Iam: Profile, Role, Permissions, PermissionsRole, SessionProfile |

| FK: PersonId, CampuseId, RoleId |

+-------------------+--------------------+---------------------+

| ProfileId (1 : N) | CampuseId (1 : N)

+-------------------v--------------------v---------------------+

| Fleet: Brand, Model, Bus, DriverAssignemts |

| FK: ModelId, CampuseId, GpsDeviceId, ProfileId |

+-------------------+--------------------+---------------------+

| GpsDeviceId (1 : 1) | BusId and RouteId

+-------------------v--------------------v---------------------+

| Gps: GpsDevice, GpsLocation |

+--------------------------------------------------------------+

| Route: Route, Stop, RouteStop, RouteBusAssignments, |

| RouteStudentAssignments, RouteSchedule, RouteExecution |

| FK: CampuseId, CityId, SchoolId, BusId, ProfileId |

+----------------------------+---------------------------------+

| RouteExecutionId (1 : N)

+----------------------------v---------------------------------+

| Notification: Boarding, Alert, AlertType, AlertRecipient, |

| DeviceToken |

| FK: RouteExecutionId, RouteStopId, AlertTypeId, ProfileId |

+--------------------------------------------------------------+

| Exceptional: ExceptionalDriverUsage, ExceptionalRouteUsage |

| FK: BusId, RouteId, ProfileId |

| Audit: Audit (FK ProfileId), LogErrors |

| Settings: PasswordPolicies (password policy), |

| SecuritySettings — no foreign key |

+--------------------------------------------------------------+

**Figure 18**. Global ER diagram of the data model

Reading of the diagram: the city affiliates the schools; the school deploys campuses and courses; the campus is the anchoring point of profiles, buses and routes; the profile (identity and role) is the node that connects persons, families, licenses, drivers, assignments, sessions, alerts, boardings, exceptional uses and auditing; the route is made up of ordered stops and of bus and student assignments; the route execution is the container of the boardings and of the alerts of a real trip; and the geolocation device delivers the positions that feed the execution.

## 5.2. Iam schema — 5 tables

**Table 20**. Columns of the Iam schema

| Table           | Column          | Type             | Nullability | PK/FK             | Description                                                  |
| --------------- | --------------- | ---------------- | ----------- | ----------------- | ------------------------------------------------------------ |
| Role            | Id              | TINYINT identity | NOT NULL    | PK                | Role identifier                                              |
| Role            | Name            | VARCHAR(20)      | NOT NULL    | UNIQUE            | Role name                                                    |
| Role            | Description     | VARCHAR(100)     | NOT NULL    | —                 | Role scope                                                   |
| Role            | Status          | VARCHAR(20)      | NOT NULL    | —                 | Role lifecycle                                               |
| Permissions     | Id              | TINYINT identity | NOT NULL    | PK                | Permission identifier                                        |
| Permissions     | TypePermissions | VARCHAR(100)     | NOT NULL    | —                 | Permission code (resource.action)                            |
| Permissions     | Description     | VARCHAR(150)     | NOT NULL    | —                 | Readable description of the permission                       |
| PermissionsRole | Id              | TINYINT identity | NOT NULL    | PK                | Assignment identifier                                        |
| PermissionsRole | PermissionsId   | TINYINT          | NOT NULL    | FK → Permissions  | Assigned permission                                          |
| PermissionsRole | RoleId          | TINYINT          | NOT NULL    | FK → Role         | Receiving role                                               |
| Profile         | Id              | UNIQUEIDENTIFIER | NOT NULL    | PK                | Access identity                                              |
| Profile         | PersonId        | UNIQUEIDENTIFIER | NOT NULL    | FK → Person       | Person who owns the profile                                  |
| Profile         | CampuseId       | UNIQUEIDENTIFIER | Yes         | FK → SchoolCampus | Campus to which the profile belongs                          |
| Profile         | PasswordHash    | VARCHAR(100)     | NOT NULL    | —                 | Protected summary of the password (password policy)          |
| Profile         | RoleId          | TINYINT          | NOT NULL    | FK → Role         | Current role of the profile                                  |
| Profile         | Status          | VARCHAR(20)      | NOT NULL    | —                 | Active or inactive                                           |
| SessionProfile  | Id              | INT identity     | NOT NULL    | PK                | Session identifier                                           |
| SessionProfile  | ProfileId       | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile      | Profile of the session                                       |
| SessionProfile  | DateStart       | DATETIME2        | NOT NULL    | —                 | Session start                                                |
| SessionProfile  | DateEnd         | DATETIME2        | NOT NULL    | —                 | Session close                                                |
| SessionProfile  | IPAddress       | VARCHAR(50)      | NOT NULL    | —                 | Session origin                                               |
| SessionProfile  | StatusSession   | VARCHAR(50)      | NOT NULL    | —                 | Session status                                               |

## 5.3. UserManagement schema — 4 tables

**Table 21**. Columns of the UserManagement schema

| Table         | Column                | Type             | Nullability | PK/FK                   | Description                      |
| ------------- | --------------------- | ---------------- | ----------- | ----------------------- | -------------------------------- |
| Person        | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                      | Person identity                  |
| Person        | Name                  | VARCHAR(50)      | NOT NULL    | —                       | First name                       |
| Person        | LastName              | VARCHAR(50)      | NOT NULL    | —                       | Last name                        |
| Person        | IdentificationType    | VARCHAR(20)      | NOT NULL    | CHECK (TI, CC)          | Document type                    |
| Person        | IdentificationNumber  | VARCHAR(20)      | NOT NULL    | UNIQUE                  | Document number                  |
| Person        | Email                 | VARCHAR(50)      | NOT NULL    | —                       | Email address                    |
| Person        | Phone                 | INT              | NOT NULL    | —                       | Contact phone                    |
| Person        | ResidenceAddress      | VARCHAR(50)      | NOT NULL    | —                       | Residence address                |
| Person        | DateBirth             | DATE             | NOT NULL    | —                       | Date of birth                    |
| Person        | Status                | VARCHAR(20)      | NOT NULL    | —                       | Active or inactive               |
| Family        | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                      | Family group identifier          |
| Family        | Name                  | VARCHAR(50)      | NOT NULL    | —                       | Group name                       |
| Family        | Observations          | VARCHAR(100)     | NOT NULL    | —                       | Group observations               |
| Family        | Status                | VARCHAR(20)      | NOT NULL    | —                       | Active or inactive               |
| FamilyMember  | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                      | Link identifier                  |
| FamilyMember  | FamilyId              | UNIQUEIDENTIFIER | NOT NULL    | FK → Family             | Family group                     |
| FamilyMember  | ProfileId             | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile            | Linked profile                   |
| FamilyMember  | RelationshipType      | VARCHAR(20)      | NOT NULL    | CHECK (Parent, Student) | Relationship within the group    |
| FamilyMember  | Status                | VARCHAR(20)      | NOT NULL    | —                       | Active or inactive               |
| DriverLicense | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                      | License identifier               |
| DriverLicense | ProfileId             | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile            | License holder (driver)          |
| DriverLicense | LicenseNumber         | VARCHAR(20)      | NOT NULL    | UNIQUE                  | License number                   |
| DriverLicense | LicenseExpirationDate | DATE             | NOT NULL    | —                       | Expiration date                  |
| DriverLicense | Status                | VARCHAR(20)      | NOT NULL    | —                       | Current or not current           |

## 5.4. School schema — 4 tables

**Table 22**. Columns of the School schema

| Table        | Column    | Type             | Nullability | PK/FK                | Description                             |
| ------------ | --------- | ---------------- | ----------- | -------------------- | --------------------------------------- |
| School       | Id        | UNIQUEIDENTIFIER | NOT NULL    | PK                   | School identifier                       |
| School       | CityId    | UNIQUEIDENTIFIER | NOT NULL    | FK → City            | City of location                        |
| School       | Logo      | VARBINARY(MAX)   | NOT NULL    | —                    | Institutional logo                      |
| School       | Name      | VARCHAR(30)      | NOT NULL    | —                    | School name                             |
| School       | Address   | VARCHAR(30)      | NOT NULL    | —                    | Address                                 |
| School       | Phone     | INT              | NOT NULL    | —                    | Contact phone                           |
| School       | Email     | VARCHAR(50)      | NOT NULL    | —                    | Institutional email                     |
| School       | Website   | VARCHAR(100)     | Yes         | —                    | Institutional website                   |
| School       | Theme     | VARCHAR(20)      | Yes         | —                    | Institutional visual theme              |
| School       | Status    | VARCHAR(20)      | NOT NULL    | —                    | Active or inactive                      |
| SchoolCampus | Id        | UNIQUEIDENTIFIER | NOT NULL    | PK                   | Campus identifier                       |
| SchoolCampus | SchoolId  | UNIQUEIDENTIFIER | NOT NULL    | FK → School          | Owning school                           |
| SchoolCampus | Name      | VARCHAR(30)      | NOT NULL    | —                    | Campus name                             |
| SchoolCampus | Address   | VARCHAR(30)      | NOT NULL    | —                    | Campus address                          |
| SchoolCampus | Status    | VARCHAR(20)      | NOT NULL    | —                    | Active or inactive                      |
| Course       | Id        | UNIQUEIDENTIFIER | NOT NULL    | PK                   | Course identifier                       |
| Course       | CampusId  | UNIQUEIDENTIFIER | NOT NULL    | FK → SchoolCampus    | Campus that offers the course           |
| Course       | Name      | VARCHAR(10)      | NOT NULL    | —                    | Course name                             |
| Course       | Status    | VARCHAR(20)      | NOT NULL    | —                    | Active or inactive                      |
| SchoolAdmin  | Id        | UNIQUEIDENTIFIER | NOT NULL    | PK                   | Assignment identifier                   |
| SchoolAdmin  | SchoolId  | UNIQUEIDENTIFIER | NOT NULL    | FK → School          | Administered school                     |
| SchoolAdmin  | ProfileId | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile, UNIQUE | Assigned administrator (one per profile) |
| SchoolAdmin  | Status    | VARCHAR(20)      | NOT NULL    | —                    | Active or inactive                      |

## 5.5. Geographic schema — 1 table

**Table 23**. Columns of the Geographic schema

| Table | Column  | Type             | Nullability | PK/FK | Description                |
| ----- | ------- | ---------------- | ----------- | ----- | -------------------------- |
| City  | Id      | UNIQUEIDENTIFIER | NOT NULL    | PK    | City identifier            |
| City  | Name    | VARCHAR(40)      | NOT NULL    | —     | City name                  |
| City  | Country | VARCHAR(30)      | NOT NULL    | —     | Country of the city        |
| City  | Status  | VARCHAR(20)      | NOT NULL    | —     | Active or inactive         |

## 5.6. Fleet schema — 4 tables

**Table 24**. Columns of the Fleet schema

| Table            | Column       | Type             | Nullability | PK/FK                  | Description                    |
| ---------------- | ------------ | ---------------- | ----------- | ---------------------- | ------------------------------ |
| Brand            | Id           | TINYINT identity | NOT NULL    | PK                     | Brand identifier               |
| Brand            | Name         | VARCHAR(30)      | Yes         | —                      | Brand name                     |
| Brand            | Status       | VARCHAR(20)      | NOT NULL    | —                      | Active or inactive             |
| Model            | Id           | TINYINT identity | NOT NULL    | PK                     | Model identifier               |
| Model            | BrandId      | TINYINT          | NOT NULL    | FK → Brand             | Owning brand                   |
| Model            | Name         | VARCHAR(30)      | Yes         | —                      | Model name                     |
| Model            | Status       | VARCHAR(20)      | NOT NULL    | —                      | Active or inactive             |
| Bus              | Id           | UNIQUEIDENTIFIER | NOT NULL    | PK                     | Bus identifier                 |
| Bus              | CampuseId    | UNIQUEIDENTIFIER | NOT NULL    | FK → SchoolCampus      | Campus to which it belongs     |
| Bus              | SoatValidity | DATE             | NOT NULL    | —                      | SOAT validity                  |
| Bus              | GpsDeviceId  | UNIQUEIDENTIFIER | NOT NULL    | FK → GpsDevice, UNIQUE | Geolocation device             |
| Bus              | Capacity     | TINYINT          | NOT NULL    | —                      | Passenger capacity             |
| Bus              | Plate        | VARCHAR(15)      | NOT NULL    | UNIQUE                 | Vehicle plate                  |
| Bus              | ModelId      | TINYINT          | NOT NULL    | FK → Model             | Vehicle model                  |
| Bus              | Status       | VARCHAR(20)      | NOT NULL    | —                      | Active or inactive             |
| DriverAssignemts | Id           | UNIQUEIDENTIFIER | NOT NULL    | PK                     | Assignment identifier          |
| DriverAssignemts | ProfileId    | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile           | Assigned driver                |
| DriverAssignemts | BusId        | UNIQUEIDENTIFIER | NOT NULL    | FK → Bus               | Assigned bus                   |
| DriverAssignemts | AssignedFrom | DATE             | NOT NULL    | —                      | Start of validity              |
| DriverAssignemts | AssignedTo   | DATE             | Yes         | —                      | End of validity                |

## 5.7. Route schema — 7 tables

**Table 25**. Columns of the Route schema

| Table                   | Column                | Type             | Nullability | PK/FK                    | Description                    |
| ----------------------- | --------------------- | ---------------- | ----------- | ------------------------ | ------------------------------ |
| Route                   | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                       | Route identifier               |
| Route                   | CampuseId             | UNIQUEIDENTIFIER | NOT NULL    | FK → SchoolCampus        | Campus that operates the route |
| Route                   | Name                  | VARCHAR(30)      | NOT NULL    | —                        | Route name                     |
| Route                   | TargetSector          | VARCHAR(30)      | NOT NULL    | —                        | Coverage sector or zone        |
| Route                   | StartTime             | TIME             | NOT NULL    | —                        | Scheduled start time           |
| Route                   | EndTime               | TIME             | NOT NULL    | —                        | Scheduled end time             |
| Route                   | Status                | VARCHAR(20)      | NOT NULL    | —                        | Active or inactive             |
| Stop                    | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                       | Stop identifier                |
| Stop                    | CityId                | UNIQUEIDENTIFIER | NOT NULL    | FK → City                | City of the stop               |
| Stop                    | SchoolId              | UNIQUEIDENTIFIER | NOT NULL    | FK → School              | Destination school             |
| Stop                    | Name                  | VARCHAR(30)      | NOT NULL    | —                        | Stop name                      |
| Stop                    | Address               | VARCHAR(100)     | NOT NULL    | —                        | Stop address                   |
| Stop                    | Longitude             | DECIMAL(10,7)    | NOT NULL    | —                        | Stop longitude                 |
| Stop                    | Latitude              | DECIMAL(10,7)    | NOT NULL    | —                        | Stop latitude                  |
| Stop                    | Status                | VARCHAR(20)      | NOT NULL    | —                        | Active or inactive             |
| RouteStop               | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                       | Step identifier                |
| RouteStop               | RouteId               | UNIQUEIDENTIFIER | NOT NULL    | FK → Route               | Route                          |
| RouteStop               | StopId                | UNIQUEIDENTIFIER | NOT NULL    | FK → Stop                | Stop                           |
| RouteStop               | OrderSequence         | INT              | NOT NULL    | —                        | Order within the journey       |
| RouteStop               | Status                | VARCHAR(20)      | NOT NULL    | —                        | Active or inactive             |
| RouteBusAssignments     | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                       | Assignment identifier          |
| RouteBusAssignments     | BusId                 | UNIQUEIDENTIFIER | NOT NULL    | FK → Bus                 | Assigned bus                   |
| RouteBusAssignments     | RouteId               | UNIQUEIDENTIFIER | NOT NULL    | FK → Route               | Assigned route                 |
| RouteBusAssignments     | Status                | VARCHAR(20)      | NOT NULL    | —                        | Active or inactive             |
| RouteStudentAssignments | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                       | Assignment identifier          |
| RouteStudentAssignments | ProfileId             | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile             | Assigned student               |
| RouteStudentAssignments | RouteStopId           | UNIQUEIDENTIFIER | NOT NULL    | FK → RouteStop           | Drop-off stop                  |
| RouteStudentAssignments | Status                | VARCHAR(20)      | NOT NULL    | —                        | Active or inactive             |
| RouteSchedule           | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                       | Schedule identifier            |
| RouteSchedule           | RouteBusAssignmentsId | UNIQUEIDENTIFIER | NOT NULL    | FK → RouteBusAssignments | Assignment that is scheduled   |
| RouteSchedule           | DayOfWeek             | VARCHAR(10)      | NOT NULL    | CHECK (Monday to Friday) | Day of the week                |
| RouteSchedule           | Direction             | VARCHAR(10)      | NOT NULL    | —                        | Direction of the journey       |
| RouteSchedule           | StartTime             | TIME             | NOT NULL    | —                        | Departure time                 |
| RouteSchedule           | Status                | VARCHAR(20)      | NOT NULL    | —                        | Active or inactive             |
| RouteExecution          | Id                    | UNIQUEIDENTIFIER | NOT NULL    | PK                       | Execution identifier           |
| RouteExecution          | RouteId               | UNIQUEIDENTIFIER | Yes         | FK → Route               | Executed route                 |
| RouteExecution          | BusId                 | UNIQUEIDENTIFIER | NOT NULL    | FK → Bus                 | Bus used                       |
| RouteExecution          | DriverId              | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile             | Driver of the trip             |
| RouteExecution          | StartDateTime         | DATETIME2        | NOT NULL    | —                        | Actual start of the trip       |
| RouteExecution          | EndDateTime           | DATETIME2        | Yes         | —                        | Actual end of the trip         |
| RouteExecution          | Status                | VARCHAR(20)      | NOT NULL    | —                        | Execution status               |

## 5.8. Gps schema — 2 tables

**Table 26**. Columns of the Gps schema

| Table       | Column         | Type             | Nullability | PK/FK          | Description                      |
| ----------- | -------------- | ---------------- | ----------- | -------------- | -------------------------------- |
| GpsDevice   | Id             | UNIQUEIDENTIFIER | NOT NULL    | PK             | Device identifier                |
| GpsDevice   | Imei           | VARCHAR(20)      | NOT NULL    | UNIQUE         | Equipment identifier             |
| GpsDevice   | LastConnection | DATETIME2        | NOT NULL    | —              | Last communication received      |
| GpsDevice   | GpsStatus      | BIT              | NOT NULL    | —              | Operational status of the device |
| GpsLocation | Id             | UNIQUEIDENTIFIER | NOT NULL    | PK             | Position identifier              |
| GpsLocation | GpsDeviceId    | UNIQUEIDENTIFIER | NOT NULL    | FK → GpsDevice | Reporting device                 |
| GpsLocation | Latitude       | DECIMAL(10,7)    | NOT NULL    | —              | Reported latitude                |
| GpsLocation | Longitude      | DECIMAL(10,7)    | NOT NULL    | —              | Reported longitude               |
| GpsLocation | Speed          | DECIMAL(8,2)     | NOT NULL    | —              | Instantaneous speed              |
| GpsLocation | Course         | DECIMAL(8,2)     | NOT NULL    | —              | Vehicle heading                  |
| GpsLocation | DateTime       | DATETIME2        | NOT NULL    | —              | Position timestamp               |

## 5.9. Notification schema — 5 tables

**Table 27**. Columns of the Notification schema

| Table          | Column           | Type             | Nullability | PK/FK                       | Description                      |
| -------------- | ---------------- | ---------------- | ----------- | --------------------------- | -------------------------------- |
| AlertType      | Id               | TINYINT identity | NOT NULL    | PK                          | Alert type identifier            |
| AlertType      | Name             | VARCHAR(30)      | NOT NULL    | —                           | Type name                        |
| AlertType      | Description      | VARCHAR(100)     | NOT NULL    | —                           | Type description                 |
| AlertType      | UrgencyLevel     | TINYINT          | NOT NULL    | —                           | Urgency level                    |
| Alert          | Id               | UNIQUEIDENTIFIER | NOT NULL    | PK                          | Alert identifier                 |
| Alert          | RouteExecutionId | UNIQUEIDENTIFIER | NOT NULL    | FK → RouteExecution         | Execution that originates it     |
| Alert          | AlertTypeId      | TINYINT          | NOT NULL    | FK → AlertType              | Alert type                       |
| Alert          | ProfileId        | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile                | Who generates the alert          |
| Alert          | DateTime         | DATETIME2        | NOT NULL    | —                           | Time of generation               |
| Alert          | Description      | VARCHAR(100)     | Yes         | —                           | Detail of the event              |
| Alert          | AcknowledgedAt   | DATETIME2        | Yes         | —                           | Time of acknowledgment          |
| AlertRecipient | Id               | UNIQUEIDENTIFIER | NOT NULL    | PK                          | Recipient identifier             |
| AlertRecipient | AlertId          | UNIQUEIDENTIFIER | NOT NULL    | FK → Alert                  | Target alert                     |
| AlertRecipient | ProfileId        | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile                | Target profile                   |
| AlertRecipient | DateTimeRead     | DATETIME2        | NOT NULL    | —                           | Time of reading                  |
| Boarding       | Id               | UNIQUEIDENTIFIER | NOT NULL    | PK                          | Boarding identifier              |
| Boarding       | RouteExecutionId | UNIQUEIDENTIFIER | NOT NULL    | FK → RouteExecution         | Execution in progress            |
| Boarding       | ProfileId        | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile                | Student who boards               |
| Boarding       | RouteStopId      | UNIQUEIDENTIFIER | NOT NULL    | FK → RouteStop              | Stop of the event                |
| Boarding       | DateTime         | DATETIME2        | NOT NULL    | —                           | Time of the scan                 |
| Boarding       | BoardingType     | VARCHAR(20)      | NOT NULL    | CHECK (ON_BOARD, OFF_BOARD) | Pick-up or drop-off              |
| DeviceToken    | Id               | UNIQUEIDENTIFIER | NOT NULL    | PK                          | Token identifier                 |
| DeviceToken    | ProfileId        | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile                | Profile that owns the device     |
| DeviceToken    | ExpoToken        | VARCHAR(300)     | NOT NULL    | UNIQUE                      | Push notification token          |
| DeviceToken    | UpdatedAt        | DATETIME2        | NOT NULL    | —                           | Last update                      |

## 5.10. Exceptional schema — 2 tables

**Table 28**. Columns of the Exceptional schema

| Table                  | Column        | Type             | Nullability | PK/FK        | Description                     |
| ---------------------- | ------------- | ---------------- | ----------- | ------------ | ------------------------------- |
| ExceptionalDriverUsage | Id            | UNIQUEIDENTIFIER | NOT NULL    | PK           | Record identifier               |
| ExceptionalDriverUsage | BusId         | UNIQUEIDENTIFIER | NOT NULL    | FK → Bus     | Bus used                        |
| ExceptionalDriverUsage | ProfileId     | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile | Person who requests it          |
| ExceptionalDriverUsage | DateTime      | DATETIME2        | NOT NULL    | —            | Time of use                     |
| ExceptionalDriverUsage | Reason        | VARCHAR(200)     | NOT NULL    | —            | Reason for the exceptional use  |
| ExceptionalRouteUsage  | Id            | UNIQUEIDENTIFIER | NOT NULL    | PK           | Record identifier               |
| ExceptionalRouteUsage  | ProfileId     | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile | Person who requests it          |
| ExceptionalRouteUsage  | RouteId       | UNIQUEIDENTIFIER | NOT NULL    | FK → Route   | Affected route                  |
| ExceptionalRouteUsage  | StartDateTime | DATETIME         | NOT NULL    | —            | Start of the authorized window  |
| ExceptionalRouteUsage  | EndDateTime   | DATETIME         | NOT NULL    | —            | End of the authorized window    |
| ExceptionalRouteUsage  | Reason        | VARCHAR(200)     | NOT NULL    | —            | Reason for the deviation        |

## 5.11. Settings schema — 2 tables

**Table 29**. Columns of the Settings schema

| Table            | Column                 | Type             | Nullability | PK/FK  | Description                                      |
| ---------------- | ---------------------- | ---------------- | ----------- | ------ | ------------------------------------------------ |
| PasswordPolicies | Id                     | TINYINT identity | NOT NULL    | PK     | Password policy identifier                       |
| PasswordPolicies | MinLongitude           | TINYINT          | NOT NULL    | —      | Minimum length of the password policy            |
| PasswordPolicies | MaxLongitude           | TINYINT          | NOT NULL    | —      | Maximum length of the password policy            |
| PasswordPolicies | RequiresCapitalLetters | TINYINT          | NOT NULL    | —      | Requires uppercase letters in the password policy |
| PasswordPolicies | RequiresNumbers        | TINYINT          | NOT NULL    | —      | Requires numbers in the password policy          |
| PasswordPolicies | RequiresSymbols        | TINYINT          | NOT NULL    | —      | Requires symbols in the password policy          |
| PasswordPolicies | ExpirationDays         | TINYINT          | NOT NULL    | —      | Expiration days of the password policy           |
| SecuritySettings | Id                     | TINYINT identity | NOT NULL    | PK     | Parameter identifier                             |
| SecuritySettings | NameSettings           | VARCHAR(100)     | NOT NULL    | UNIQUE | Name of the security parameter                   |
| SecuritySettings | ValueSettings          | VARCHAR(100)     | NOT NULL    | —      | Value of the security parameter                  |
| SecuritySettings | Description            | VARCHAR(150)     | NOT NULL    | —      | Description of the security parameter            |

## 5.12. Audit schema — 2 tables

**Table 30**. Columns of the Audit schema

| Table     | Column      | Type             | Nullability | PK/FK        | Description                        |
| --------- | ----------- | ---------------- | ----------- | ------------ | ---------------------------------- |
| Audit     | Id          | BIGINT identity  | NOT NULL    | PK           | Event identifier                   |
| Audit     | ProfileId   | UNIQUEIDENTIFIER | NOT NULL    | FK → Profile | Actor who performs the action      |
| Audit     | Action      | VARCHAR(255)     | NOT NULL    | —            | Action performed                   |
| Audit     | DateTime    | DATETIME2        | NOT NULL    | —            | Time of the action                 |
| Audit     | Description | VARCHAR(500)     | NOT NULL    | —            | Detail of the action               |
| Audit     | IPAddress   | VARCHAR(50)      | NOT NULL    | —            | Origin of the action               |
| Audit     | Application | VARCHAR(255)     | NOT NULL    | —            | Application from which it runs     |
| LogErrors | Id          | BIGINT identity  | NOT NULL    | PK           | Error identifier                   |
| LogErrors | TypeError   | VARCHAR(100)     | NOT NULL    | —            | Classification of the error        |
| LogErrors | Description | VARCHAR(150)     | NOT NULL    | —            | Description of the error           |
| LogErrors | IPAddress   | VARCHAR(50)      | NOT NULL    | —            | Origin of the error                |
| LogErrors | OccurredAt  | DATETIME2        | NOT NULL    | —            | Time at which it occurred          |

## 5.13. Enums and seed data

The sets of closed values are protected with inline restrictions:

**Table 31**. Sets of admitted values

| Set                     | Admitted values                             | Table                               |
| ----------------------- | ------------------------------------------- | ----------------------------------- |
| Lifecycle status        | ACTIVE, INACTIVE                            | Almost all business entities        |
| Role                    | SUPER_ADMIN, ADMIN, DRIVER, PARENT, STUDENT | Role                                |
| Identification type     | TI, CC                                      | Person                              |
| Family relationship     | Parent, Student                             | FamilyMember                        |
| Boarding type           | ON_BOARD, OFF_BOARD                         | Boarding                            |
| Day of the week         | Monday … Friday                             | RouteSchedule                       |

Seed data with which the system starts:

**Table 32**. Seed data and affected tables

| Content               | Seed rules                                                                                                                       | Affected tables                                                                                     |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Roles and permissions | Five canonical roles; 20 domain permissions (resource.action) and their assignments per role                                     | Role, Permissions, PermissionsRole                                                                  |
| Test identities       | Test persons and profiles for development and demonstration                                                                      | Person, Profile, SessionProfile                                                                     |
| Location              | 20 cities with country                                                                                                           | City                                                                                                |
| Institutions          | 3 schools, 15 campuses and 90 courses                                                                                            | School, SchoolCampus, Course                                                                        |
| Fleet                 | 20 brands, 20 models, 20 buses and 20 driver assignments                                                                         | Brand, Model, Bus, DriverAssignemts                                                                 |
| Routes                | 20 routes, 20 stops, 60 route steps, 20 bus assignments, 20 student assignments, 40 schedules and 20 executions                  | Route, Stop, RouteStop, RouteBusAssignments, RouteStudentAssignments, RouteSchedule, RouteExecution |
| Geolocation           | 20 devices and 20 reference positions                                                                                            | GpsDevice, GpsLocation                                                                              |
| Notifications         | 20 alert types, 20 alerts, 20 recipients and 20 boardings                                                                        | AlertType, Alert, AlertRecipient, Boarding                                                          |
| Families and licenses | 20 families, 40 members and 20 driver licenses                                                                                   | Family, FamilyMember, DriverLicense                                                                 |
| Exceptional uses      | 20 bus use records and 20 route deviation records                                                                                | ExceptionalDriverUsage, ExceptionalRouteUsage                                                       |

## 5.14. Indexes and constraints

**Table 33**. Indexes and constraints of the model

| Mechanism                            | Application                                                                                                                                          | Purpose                                              |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| IDX_Role_Name                        | Search by role name                                                                                                                                  | Resolve the role by name in authorization             |
| IDX_Profile_RoleId_PersonId          | Profile by role and person                                                                                                                           | Listing queries by role                              |
| IDX_Person_Email                     | Person by email                                                                                                                                      | Credential lookup and rotation                       |
| IDX_Person_Name, IDX_Person_LastName | Person by first and last name                                                                                                                        | Search in administrative listings                    |
| IDX_Person_IdentificationNumber      | Person by document                                                                                                                                   | Duplicate verification                                |
| IDX_Person_Phone                     | Person by phone                                                                                                                                      | Search and contact                                   |
| Declared uniqueness                  | Person.IdentificationNumber, Bus.Plate, GpsDevice.Imei, DriverLicense.LicenseNumber, DeviceToken.ExpoToken, Role.Name, SecuritySettings.NameSettings | Prevents duplicate records at the database level     |
| Business uniqueness                  | SchoolAdmin.ProfileId (an administrator administers a single school)                                                                                 | Business rule guaranteed in the engine               |
| Domain restrictions                  | Statuses, document types, family relationship, boarding type and day of the week                                                                     | Prevents out-of-catalog values                       |
| Referential integrity                | 45 foreign keys declared between schemas                                                                                                             | Prevents orphans between domain contexts             |

## 5.15. Business rules and retention

**Table 34**. Business rules and data retention

| Rule                        | Expression in the model                                                                                                                                                         |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Soft delete                 | The disabling of an entity is done with Status = INACTIVE; operational listings filter by ACTIVE.                                                                                |
| One profile per campus      | Profile.CampuseId links the profile with a specific campus; the school is derived from that campus.                                                                              |
| One school per administrator | The unique constraint on SchoolAdmin.ProfileId prevents an administrator from serving several institutions.                                                                    |
| Validity of assignments     | DriverAssignemts.AssignedTo admits NULL, which indicates an open validity; the active driver is the one that contains the current date.                                        |
| Stop order                  | RouteStop.OrderSequence is the effective order of the journey; there cannot be two stops with the same order on a route.                                                        |
| Container execution         | Boardings and alerts always reference a RouteExecution; they cannot exist without a real trip.                                                                                  |
| Drop-off stop               | RouteStudentAssignments.RouteStopId sets the student's drop-off stop within the route.                                                                                           |
| Route of the execution      | RouteExecution.RouteId is allowed to be null so as not to block the recording of a trip when the route is reordered on the fly; it is completed at the close of the trip.       |
| End of trip                 | RouteExecution.EndDateTime admits NULL while the trip is in progress and is written when it finishes.                                                                           |
| Exclusive device            | A bus has a single geolocation device and a device is not shared between buses.                                                                                                  |
| License expiration          | DriverLicense.LicenseExpirationDate enables the blocking of driving assignment when the license is expired.                                                                      |
| Exceptional use record      | Every use of a bus outside the plan and every route deviation require a reason and a time window.                                                                                |
| Audit retention             | Audit events and LogErrors records are not deleted by business operation; their lifecycle is defined by the database backup and archiving policy.                               |
| Operational retention       | GpsLocation positions and SessionProfile sessions are data of growing volume and are purged by age according to the platform's retention policy.                                |
| Backups                     | The database is backed up on a scheduled basis with verified restoration; the schema is rebuilt with migrations if restoration is not possible.                                  |

# 6 Interface contracts

The contracts are grouped into three families: versioned REST resources under the /api/v1 prefix and exposed through the API gateway, domain events published to the broker, and internal low-latency RPC calls. Authorization is evaluated by role and permission both at the API gateway and in each service, so that the role column of the following table represents the effective access matrix.

## 6.1. REST resources

**Table 35**. REST resources and authorized roles

| Verb             | Path                                             | Role                  | Description                                                                               |
| ---------------- | ------------------------------------------------ | --------------------- | ----------------------------------------------------------------------------------------- |
| POST             | /api/v1/iam/auth/login                           | Public                | Authenticates and issues access and refresh token                                         |
| POST             | /api/v1/iam/auth/refresh                         | Public                | Rotates the refresh token and renews the session                                          |
| POST             | /api/v1/iam/auth/logout                          | Authenticated         | Revokes the current session                                                               |
| POST             | /api/v1/iam/auth/forgot-password                 | Public                | Starts recovery in accordance with the password policy                                    |
| POST             | /api/v1/iam/auth/reset-password                  | Public                | Resets the password by verifying the six-length code                                      |
| POST             | /api/v1/iam/auth/change-password                 | Authenticated         | Changes the current password applying the password policy                                 |
| GET              | /api/v1/iam/jwks                                 | Public                | Publishes the keys to verify the token signature                                          |
| GET, POST        | /api/v1/iam/roles                                | SUPER_ADMIN           | Query and creation of roles                                                               |
| GET              | /api/v1/iam/permissions                          | SUPER_ADMIN           | Query of the permission catalog                                                           |
| GET, PATCH       | /api/v1/iam/profiles\[/{id}/status\]             | ADMIN                 | Listing of the institution's profiles and activation or deactivation of a profile          |
| POST, DELETE     | /api/v1/users/persons\[/{id}\]                   | ADMIN                 | Registration of persons (student, guardian, driver, administrator) and soft delete        |
| POST             | /api/v1/users/families                           | ADMIN                 | Creation of a family group                                                                |
| POST             | /api/v1/users/families/{id}/members              | ADMIN                 | Linking of a member to the family group                                                   |
| POST             | /api/v1/users/driver-licenses                    | ADMIN                 | Registration of a driver license with validity                                             |
| POST, PUT        | /api/v1/schools/schools\[/{id}\]                 | SUPER_ADMIN           | Registration and update of the educational institution                                    |
| POST             | /api/v1/schools/campuses                         | SUPER_ADMIN           | Registration of a campus                                                                  |
| POST             | /api/v1/schools/courses                          | ADMIN                 | Registration of a course in a campus                                                      |
| POST             | /api/v1/schools/cities                           | SUPER_ADMIN           | Registration of a reference city                                                          |
| GET              | /api/v1/buses/brands, /models                    | ADMIN                 | Brand and model catalogs                                                                  |
| POST, PUT        | /api/v1/buses/buses\[/{id}\]                     | ADMIN                 | Registration and update of a bus with plate, capacity and device                           |
| DELETE           | /api/v1/buses/buses/{id}/driver                  | ADMIN                 | Removal of the assigned driver                                                            |
| POST, DELETE     | /api/v1/routes/routes\[/{id}\]                   | ADMIN                 | Registration of a route with sector and schedule, and deletion of the route                |
| POST             | /api/v1/routes/routes/{id}/stops                 | ADMIN                 | Incorporation of a stop in order within the route                                         |
| POST             | /api/v1/routes/routes/{id}/students              | ADMIN                 | Assignment of a student to a stop                                                         |
| POST             | /api/v1/routes/routes/{id}/bus                   | ADMIN                 | Assignment of a bus to the route                                                          |
| GET              | /api/v1/routes/routes/{id}/schedules             | ADMIN                 | Weekly schedule of the route                                                              |
| POST             | /api/v1/routes/executions                        | ADMIN, DRIVER         | Opening of the real execution of a trip                                                   |
| GET              | /api/v1/notifications/alert-types                | ADMIN                 | Catalog of alert types and urgency level                                                  |
| POST, PATCH      | /api/v1/notifications/alerts\[/{id}/status\]     | ADMIN, DRIVER         | Generation of an alert on an execution and acknowledgment                                  |
| GET              | /api/v1/notifications/boarding                   | ADMIN, DRIVER         | Query of boarding events                                                                  |
| GET              | /api/v1/notifications/recipients                 | ADMIN                 | Query of recipients and reading of alerts                                                 |
| POST, GET, PATCH | /api/v1/gps/devices\[/{id}\[/status\]\]          | ADMIN                 | Registration, status and last-connection query, and activation or suspension of the device |
| GET              | /api/v1/gps/locations                            | ADMIN, DRIVER, PARENT | Query of recorded positions                                                               |
| POST             | /api/v1/exceptional/driver-usages, /route-usages | ADMIN                 | Registration of exceptional bus use or route deviation with reason and window              |
| GET, PUT         | /api/v1/config/password-policies, /settings      | SUPER_ADMIN           | Query and adjustment of the password policy and the security parameters                    |
| POST             | /api/v1/audit/entries                            | Internal              | Ingestion of audit events from the services                                               |
| GET              | /api/v1/audit/errors                             | SUPER_ADMIN           | Query of the error log                                                                    |
| GET              | /health (API gateway)                            | Public                | Health check of the entry point                                                           |

All business routes are protected with a Bearer token except those declared as public; error responses follow a common structure with a timestamp, message and path detail, and lists are served with pagination by page and size.

## 6.2. Domain events (Kafka)

The broker operates in KRaft mode without an external coordination component and publishes six topics verified in the runtime environment. The partition key is the entity identifier, to preserve order per aggregate.

**Table 36**. Domain events and their consumers

| Event           | Producer                               | Consumer                              | Summary payload                                                                              |
| --------------- | -------------------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------------- |
| student.created | User management service                | Identification and access service     | Event identifier, person, email, document number and campus                                  |
| driver.created  | User management service                | Identification and access service     | Same content as student registration, with the driver role                                   |
| admin.created   | User management service                | Identification and access service     | Same content with the school to administer instead of campus                                 |
| parent.created  | User management service                | Identification and access service     | Event identifier, person, email, document and campus                                         |
| profile.created | Identification and access service      | No registered consumer                | Confirmation of the creation of the access profile for a person                              |
| student.scanned | Notification service                   | Notification service                  | Boarding, alert, student profile, route execution, boarding type and timestamp               |

Payload contract for person registration (EventId, PersonId, Email, IdentificationNumber, optional CampusId and SchoolId only in administrator registration) and for scanning (BoardingId, AlertId, StudentProfileId, RouteExecutionId, BoardingType, Timestamp): the fields are additive, consumers tolerate unknown fields and the producer owns the schema.

Consumption rules: a consumer that fails reads the message again according to the retry policy and the counterpart is not blocked; the creation of the access profile and the generation of the notification are idempotent operations with respect to the event identifier.

## 6.3. gRPC flow

Internal low-latency calls use RPC over a persistent channel on port 5001, in plain text (h2c) within the platform's private network; the encrypted transport toward the outside is terminated by the API gateway.

+--------------------------------+ +----------------------------------+

| Identification service | | School management service |

| and access (client) | RPC | (server, port 5001) |

| after consuming admin.created: +--------->| SchoolManagementService |

| LinkAdminSchool, GetCampus, | h2c | GetSchool, GetCampus, |

| GetCampusByName | | GetCampusByName, GetAdminSchool|

+--------------------------------+ | LinkAdminSchool |

| User management service| +----------------------------------+

| (client) queries campus and |

| school when registering persons |

+--------------------------------+

+--------------------------------+ +----------------------------------+

| Route service (server) | | User management service |

| RouteService | | FamilyService |

| CreateRoute, GetRoute, | | RegisterFamilyGroup |

| ListRoutes, UpdateRoute, | | (group name, |

| DeleteRoute, GetCurrentRoute,| | observations and members |

| GetStudentRoute, StartTrip, | | with relationship type) |

| EndTrip, AssignStudentToRoute| +----------------------------------+

| AssignBusToRoute, |

| AttachStopToRoute |

+--------------------------------+

**Figure 19**. Flow of gRPC calls between services

Client behavior: shared channel with lifecycle managed by the container, timeout per call, bounded retry on transient failure, and transformation of the remote error into a domain error that the API layer translates into a problem response. The request and response messages carry identifiers as strings and statuses as boolean or catalog values, avoiding deeply nested structures.

# 7 Design decisions and trade-offs

**Table 37**. Design decisions and discarded alternatives

| Decision                                                                                | Discarded alternative                     | Criterion and trade-off                                                                                                                                                                         |
| --------------------------------------------------------------------------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A single relational database with schema per service                                    | Independent database per service          | Cross-cutting referential integrity and a single backup operation are gained; the discipline of not reading other services' data and coupling in migrations are accepted.                       |
| Single API gateway with routing                                                         | Exposing each service directly            | Common security policy, rate limit and CORS are gained; a single point that must scale and be available is accepted.                                                                            |
| Event orientation for user registration and scans                                       | Synchronous chaining of calls             | Producer and consumer are decoupled in time and occasional outages are tolerated; eventual consistency between person and access profile is accepted.                                            |
| Internal RPC for low-latency queries and trips                                          | Everything over REST                      | Response times on the hot path are reduced; a second contract that must be versioned with the same discipline is accepted.                                                                      |
| Access token signed with RS256 and private key only in the identification service        | Shared symmetric key                      | Any service can verify without holding the key; renewal management and publication of public keys are added.                                                                                    |
| Hexagonal architecture within each service                                              | Single-layer MVC model                    | The domain is isolated from frameworks and databases at the cost of more types and more files per use case.                                                                                     |
| Soft delete with ACTIVE/INACTIVE                                                        | Physical deletion                         | Traceability and reversibility are preserved; it is accepted that listings must filter and that volume grows.                                                                                   |
| Globally unique identifiers as primary key                                              | Shared sequential identity                | Allows creating records without querying the sequence center and merging data between services; key locality in range lookups is lost.                                                          |
| Web application for administration and mobile application for field operations           | A single responsive web application       | Each surface is optimized for its context (tables and forms versus large actions and offline recording); a common design system is maintained to avoid duplicating visual language.              |
| Data transfer contract separated from the internal model                                | Exposing domain entities directly         | Internal changes are prevented from breaking clients; the cost is additional mappings in each operation.                                                                                        |
| Design tokens with eight closed themes                                                  | Free color customization                  | Contrast and visual coherence are guaranteed; the expressiveness of institutional customization is limited.                                                                                     |
| Scheduling through parameterized catalogs                                              | Fixed conditions in code                  | New alert types or security parameters are enabled as data; care is required not to turn the catalog into a source of opaque rules.                                                             |
