# Software Requirements Specification

**Guardian Escolar**

*Software requirements specification for the Guardian Escolar system — safe school transportation platform*

**Project team members:**

- Johan Smith Santamaria Fernández
- Sharik Dayanna Rojas Ibarra
- Juan Pablo Chala Ramírez

**Program:** Technology in Software Analysis and Development
**Cohort (Ficha):** 3145556
**Training center:** Sede La Industria, La Empresa y Los Servicios - SENA
**Location:** Neiva, Huila, Colombia
**Date:** October 6, 2026

---

## Table of contents

1. [Introduction](#1-introduction)
2. [General Description](#2-general-description)
3. [Functional Requirements](#3-functional-requirements)
4. [Non-Functional Requirements](#4-non-functional-requirements)
5. [Interface Requirements](#5-interface-requirements)
6. [Data Requirements](#6-data-requirements)
7. [User Stories](#7-user-stories)
8. [General Acceptance Criteria and Verification Strategy](#8-general-acceptance-criteria-and-verification-strategy)
9. [Glossary](#9-glossary)
10. [Appendix: US↔FR Traceability Matrix](#10-appendix-usfr-traceability-matrix)
11. [References](#11-references)

---

# 1. Introduction

## 1.1 Purpose

This report specifies the requirements for the **Guardian Escolar** software, a web and mobile platform for monitoring and securing school transportation. The specification establishes, in a verifiable manner, what the system must do, what qualities it must operate with, how it communicates with its components and its users, and what data it manages.

This specification is written in accordance with the structure recommended by the **IEEE 830** standard for software requirements specifications (SRS) and adopts the quality-in-use characteristics model of **ISO/IEC 25010** as its product quality model. Its audience is the software design, construction and verification team, the instructor responsible for the project, and the educational institution that receives the solution.

Each requirement in this document is **necessary and sufficient** to guide subsequent implementation and verification: functional requirements describe observable functions and their business rules; non-functional requirements set measurable metrics; and the traceability appendix links each user story to the requirements it implements.

## 1.2 Product Scope

**Guardian Escolar** is a centralized platform that integrates GPS tracking, boarding control through QR code scanning, real-time notifications and alerts, a web administrative panel, and mobile applications for the driver and the guardian. The software comprises:

- A **web application** built with Angular 21 and Angular Material 3, aimed at the administrative and platform operation profiles.
- A **mobile application** built with React and Expo, aimed at the driver, the guardian and the student.
- **Nine domain microservices**: identity and access (IAM), user management, school management, fleet, routes, notifications, exceptional usages, audit and settings.
- An **API Gateway (Kong OSS)** as the single entry point, which terminates TLS, applies rate limiting and CORS, and routes the versioned /api/v1 paths.
- An **event bus** with Apache Kafka in KRaft mode and a persistent low-latency **gRPC channel** for geolocation telemetry.
- A **single relational database** (SQL Server 2022) with a schema per service and versioned migrations using Liquibase.

The following elements are deliberately excluded and are not part of the scope of this specification:

**Table 2**. _Elements excluded from the product scope_

| **Out-of-scope element**                                                               | **Reason**                                                             |
| ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Charges, billing or payment for the school transportation service                               | Outside the purpose of safety and monitoring                            |
| Mechanical maintenance, fuel or fleet workshop                                    | Does not constitute a confirmed user flow                           |
| Academic management (grades, attendance, learning)                                 | The platform manages transportation, not academic life                |
| Public transportation and non-school routes                                                     | The domain is limited to school transportation                             |
| Purchase, administration or physical installation of GPS devices                             | Responsibility external to the software                                    |
| Autonomous emergency response or automatic dispatch                                     | Human and institutional response is external                         |
| Biometric identification or facial recognition                                           | Not required for the current scope and raises privacy obligations |
| Multinational regulatory localization                                                      | The initial context is Neiva (Huila), Colombia                         |
| Autonomous driver navigation and optimal route generation through artificial intelligence | Outside the scope of the current product                                  |

The system **does not offer self-registration**: all user accounts are created by the institution's administrator. There is no desktop application; access is carried out through the web browser and the mobile application.

## 1.3 Definitions and Acronyms

**Table 3**. _Definitions and Acronyms_

| **Term**   | **Definition**                                                                                      |
| ------------- | --------------------------------------------------------------------------------------------------- |
| SRS           | _Software Requirements Specification_: specification of software requirements.                    |
| RF            | Functional requirement: function or behavior that the system must exhibit.                         |
| RNF           | Non-functional requirement: measurable quality or constraint of the system.                                 |
| HU            | User story: brief description of a need from the point of view of the role.            |
| Epic          | Grouping of user stories related by the same business value.                      |
| API           | Application programming interface.                                                           |
| REST          | Architectural style for exchanging resources over HTTP.                                     |
| gRPC          | High-efficiency remote procedure call communication protocol.                              |
| JWT           | _JSON Web Token_: signed token used for authentication and authorization.                            |
| RBAC          | Role-based access control.                                                                  |
| QR            | Two-dimensional quick response code.                                                           |
| Boarding      | Event of a student boarding or alighting a bus that has been registered.                                    |
| GPS           | Global positioning system.                                                                  |
| Guardian      | Adult responsible for one or more students, authorized to consult their transportation information. |
| SLO           | Service level objective.                                                                      |
| P95 / P99     | 95th or 99th percentile of a distribution of measurements.                                                   |
| RTO / RPO     | Recovery time objective / recovery point objective.                             |
| PII           | Personally identifiable information.                                                             |
| Error rate    | Percentage of requests that end with a server response.                                   |

## 1.4 Normative References

The following standards and regulatory frameworks are mandatory or reference material for this specification:

**Table 4**. _Reference standards and regulatory frameworks_

| **Reference**                                     | **Subject**                                                                                    |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| IEEE 830-1998                                      | Guide for the preparation of software requirements specifications.                       |
| ISO/IEC 25010:2011                                 | Software product quality characteristics model.                                 |
| ISO/IEC/IEEE 12207                                 | Software life cycle processes.                                                      |
| ISO/IEC 29119                                      | Software testing standards.                                                                |
| OWASP Top 10                                       | Critical security risks in web applications.                                               |
| Ley 1581 de 2012 and Decreto 1377 de 2013 (Colombia) | Personal Data Protection and habeas data, with special treatment for minors' data. |
| General Data Protection Regulation (GDPR)   | Applicable if the product is offered in the European Union.                                       |

# 2. General Description

## 2.1 Product Perspective and Characteristics

Guardian Escolar solves the lack of a reliable and timely view of the school bus route: guardians do not know whether the student boarded or alighted, drivers record the information manually, and, in the event of incidents, there is neither traceability nor timely alerts. The solution replaces those manual processes with a single platform that connects families, students, drivers, administrators and platform operators around the school transportation operation.

The product relies on the following structural decisions:

- An **event-driven microservices architecture** behind an API Gateway (Kong OSS) on port **8000**; the business services listen on port **8080**.
- A **single relational database** (SQL Server 2022, port **1433**) with a schema per service and Liquibase migrations; no service accesses another service's schema directly.
- **Apache Kafka 4.1.0** in KRaft mode (port **9092**) for the domain's high-level events and **gRPC** on port **5001** for the low-latency geolocation telemetry stream.
- Authentication with **JWT RS256**: an access token of **15 minutes** and a refresh token of **30 days**; only the identity and access service holds the private key.
- Frontends: **Angular 21 + Angular Material 3 web application** and **React + Expo mobile application** (Expo push notifications and camera for QR scanning).

```
  Guardian / Student / Driver            Administrator / Operator
        (React + Expo mobile application)          (Angular web application)
                      │                                        │
                      └───────────────┬────────────────────────┘
                                      │ HTTPS · /api/v1
                                      ▼
                        ┌─────────────────────────────┐
                        │  API Gateway (Kong) :8000   │
                        │  TLS · CORS · rate limit    │
                        │  JWT RS256 · routing        │
                        └──────────────┬──────────────┘
                                       │ REST/JSON :8080
        ┌──────────┬──────────┬────────┼────────┬──────────┬──────────┐
        ▼          ▼          ▼        ▼        ▼          ▼          ▼
      IAM       User        School   Fleet    Routes   Notifications  …
               management  management
        └──────────┴──────────┴────────┬────────┴──────────┴──────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              ▼                        ▼                        ▼
      SQL Server :1433          Kafka :9092              gRPC :5001
     (schema per service)   (domain events)       (GPS positions)
```

**Figure 1**. _General overview of the platform architecture_

The following table summarizes the product characteristics and their expected satisfaction status, in accordance with the ISO/IEC 25010 quality-in-use model:

**Table 5**. _Product quality characteristics and their manifestation_

| **Characteristic**   | **Manifestation in the product**                                                             |
| -------------------- | -------------------------------------------------------------------------------------------- |
| Functional suitability | Coverage of the 30 functional requirements in section 3.                                  |
| Performance          | Latency and delivery times set in section 4.1.                                     |
| Compatibility       | Integration through REST, gRPC, events and mobile push notification.                          |
| Usability           | Role-based interfaces, side navigation, validated forms and immediate feedback. |
| Reliability           | 99.9% SLO availability, failure recovery and GPS signal loss alerts.   |
| Security            | JWT RS256 authentication, role-based authorization, auditing and personal data protection.   |
| Maintainability       | Test coverage, controlled complexity and versioned migrations.                      |
| Portability         | Containerized deployment independent of the host operating system.                    |

The product targets three objectives: (1) guarantee real-time traceability of every school journey; (2) notify guardians of the boarding and alighting status of their students; (3) provide the institution with a management panel for fleet, routes and users with roles.

## 2.2 User Classes

The system defines five roles, encoded as enumeration values. Each role authenticates and only accesses the functions and resources authorized by its role and permissions:

**Table 6**. _User classes and functions per role_

| **Role** | **Title** | **Profile and main functions** | **Interface** |
| --- | --- | --- | --- |
| `SUPER_ADMIN` | Platform operator | Manages institutions, global configuration and platform auditing. | Web |
| `ADMIN` | School administrator | Manages campuses, courses, people, fleet, routes and reports of its institution. | Web |
| `DRIVER` | Driver | Executes routes, confirms boardings through QR scanning and receives alerts in the mobile application. | Mobile |
| `PARENT` | Guardian | Tracks the route, receives pickup and drop-off notifications and consults the boarding history. | Mobile |
| `STUDENT` | Student | Consults the QR identification linked to the profile in order to board. | Mobile |

The external actors that interact with the system, without being authenticated users, are the on-board GPS device that emits positions and the push notification provider. The educational institution, the families and the emergency services remain outside the system as parties that act on the information it delivers to them.

## 2.3 Constraints

**Table 7**. _Project constraints_

| **Type**          | **Constraint**                                                                                                                                                                                                                         |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Academic         | The project is developed in the Software Analysis and Development program, ficha 3145556, at the Neiva (Huila) Training Center, Servicio Nacional de Aprendizaje.                                               |
| Geographic        | The initial operating context is Neiva (Huila), Colombia.                                                                                                                                                                               |
| Technological       | Web frontend in Angular 21 with Angular Material 3; mobile application in React with Expo; services in Java 21 with Spring Boot (identity and access) and in C# with .NET 10 (rest of the domain services).                               |
| Architectural    | Microservices style with event orientation behind an API Gateway; single relational database with schema per service; mandatory Liquibase migrations at every startup.                                             |
| Communications | Versioned synchronous interface /api/v1 over REST/JSON; gRPC on port 5001 for telemetry; events in Apache Kafka in KRaft mode (Kafka without ZooKeeper).                                                                            |
| Security      | JWT RS256 authentication with a 15-minute access token and a 30-day refresh token; private key exclusive to the identity and access service; least privilege per role; secrets only in environment variables. |
| Regulatory       | Compliance with Ley 1581 de 2012 and Decreto 1377 de 2013, with special treatment and consent for minors' data; GDPR if the product is offered in the European Union.                                                      |
| Operational    | Deployment in Docker containers and orchestration through Docker Compose; local development, testing and production environments.                                                                                                           |
| Language         | Interface with at least four languages (Spanish, English, French and Portuguese) and Spanish as the primary language of the first version.                                                                                                      |
| Process        | Specification, design, construction, testing and deployment conducted with agile methodology; any scope modification is documented and approved before construction.                                                               |

## 2.4 Assumptions and Dependencies

**Table 8**. _Project assumptions and dependencies_

| **#** | **Assumption or dependency**                                                                                                | **Type**    | **Consequence if not met**                                               |
| ----- | ------------------------------------------------------------------------------------------------------------------------- | ----------- | ------------------------------------------------------------------------------ |
| 1     | Institutions and families provide correct data for students, guardians, drivers, vehicles, routes and stops. | Assumption    | Route assignments and safety notifications would be incorrect. |
| 2     | Users have a compatible web browser or a mobile device with a camera.                                | Assumption    | An assisted or offline channel would be required.                                |
| 3     | SQL Server is available for the project's persistence.                                                             | Assumption    | The persistence architecture and the deployment plan would change.            |
| 4     | The programming interface exposes the data required by the web and mobile frontends.                                    | Assumption    | The frontends' contracts, flows and delivery dates would change. |
| 5     | A positioning provider exposes the vehicle's location and status while monitoring is active.             | Assumption    | The real-time location functions would remain unavailable.         |
| 6     | Notification delivery is available through the configured provider or service.                                  | Assumption    | Notifications would be limited to the application or to simulated data.    |
| 7     | Roles and permissions are defined coherently between frontends and services.                                      | Assumption    | Unauthorized access and inconsistent panels could occur.               |
| 8     | The initial deployment targets Neiva (Huila), Colombia.                                                               | Assumption    | The data, language, regulation and operation assumptions would have to be reviewed.     |
| 9     | SQL Server environment available before the integrated tests.                                                            | Dependency | Integrated verification would not be possible.                                    |
| 10    | Definition of the GPS provider and its contract before live monitoring.                                             | Dependency | The real-time location visualization would remain inactive.            |
| 11    | Definition of the push notification service before production.                                                      | Dependency | Notifications would not reach the devices.                          |
| 12    | Valid school and transportation data before the acceptance tests.                                               | Dependency | The real business flows could not be validated.                         |
| 13    | Role and authorization contract aligned between the service and frontend teams.                                  | Dependency | Publishing with access control would be blocked.                            |

# 3. Functional Requirements

This section specifies **30 functional requirements** organized into **10 functional groups**. Each requirement states its identifier, its name, a detailed description, its inputs and outputs, its preconditions and postconditions, and the business rules that govern it. Priority uses the High, Medium and Low scale according to business value and operational dependency.

**Table 9**. _Summary of the functional requirement groups_

| **Group** | **Title**                                   | **Functional Owner**                                        | **RF** | **Dominant Priority** |
| --------- | -------------------------------------------------- | ---------------------------------------------------------------- | ------ | ----------------------- |
| RF-01     | User Management                                | User management and identity and access service        | 4      | High                    |
| RF-02     | Authentication                                      | Identity and access service (IAM)                        | 2      | High                    |
| RF-03     | School Route, Fleet & Assignments Management   | Route service and fleet service                            | 8      | High                    |
| RF-04     | QR Scanning Management: Boarding Control | Identity and access service and notification service | 3      | High                    |
| RF-05     | Boarding Notifications                          | Notification service                                       | 4      | High                    |
| RF-06     | Real-Time Bus Tracking                     | Geolocation module and gRPC channel                           | 3      | Medium                   |
| RF-07     | Trip Management                                  | Route service                                                | 1      | Medium                   |
| RF-08     | Incidents                                         | Notification service                                       | 1      | High                    |
| RF-09     | Route Inquiry                                  | Route service                                                | 2      | Medium                   |
| RF-10     | Personalization                                    | Web and mobile client applications                                 | 2      | Low                    |

## 3.1 Group RF-01 — User Management 

This group covers the lifecycle of accounts in the school ecosystem: creation, update, activation and deactivation, and linking students to their guardian. The account is split into two responsibilities: the person's data belongs to the user management service; the authentication profile, with its role and password hash, belongs to the identity and access service.

### 3.1.1 RF-01.1 — Account Creation with Assigned Role

The administrator creates student, driver, guardian and administrator accounts, assigning in the same action the role that will define their permissions. The initial password is generated by the system and delivered to the user through a secure channel; the role is stored together with the profile and enables the corresponding functions from the first sign-in.

**Table 10**. _Specification of requirement RF-01.1: account creation with assigned role_

| **Field**       | **Detail**                                                                                                                                                                                                                               |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Document type and number, full name, email address, phone number, address, date of birth and role to assign (ADMIN, DRIVER, PARENT, STUDENT); student academic data and student–guardian link when applicable. |
| Outputs         | Account created and active with the assigned role; error message if the email or document already exists; audit record of the creation.                                                                                                |
| Preconditions  | The requester is authenticated with the ADMIN or SUPER_ADMIN role and has the user management permission.                                                                                                                            |
| Postconditions | The account exists, is active and can sign in with the initial password; the user creation event is published for dissemination in the system.                                                                             |

**Business Rules**

- There is no self-registration: all accounts are created by the institution's administrator; the administrator account is created directly by the implementation team.
- The email address and document number are unique in the system; if they already exist, the user is informed that the account is registered and the record is not duplicated.
- A student is linked to a single active guardian account at a time.
- Any missing mandatory field prevents creation and the system preserves the entered data for correction.

### 3.1.2 RF-01.2 — Account Data Update

The administrator updates the account's personal data (name, document, email, phone, address, date of birth) and the student's academic data, without altering the boarding history or the relationships already established.

**Table 11**. _Specification of requirement RF-01.2: account data update_

| **Field**       | **Detail**                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------- |
| Inputs        | Account identifier and set of fields to modify.                                      |
| Outputs         | Account updated with last modification date; error message on uniqueness conflict. |
| Preconditions  | Existing account and requester authenticated with user management permission.                    |
| Postconditions | The current data reflects the last modification; the update event is published.   |

**Business Rules**

- The creation date is immutable; the last modification date is updated automatically.
- If the modified email or document already belongs to another account, the operation is rejected without altering the record.
- Changes to people are recorded in the audit of critical actions.

### 3.1.3 RF-01.3 — Account Activation and Deactivation

The administrator activates or deactivates accounts. Deactivation prevents sign-in and blocks any subsequent assignment of the user to routes, buses or trips, while preserving their history.

**Table 12**. _Specification of requirement RF-01.3: account activation and deactivation_

| **Field**       | **Detail**                                                                                    |
| --------------- | ---------------------------------------------------------------------------------------------- |
| Inputs        | Account identifier and target status (ACTIVE / INACTIVE).                              |
| Outputs         | Updated account status; rejection of sign-in if the account is deactivated.   |
| Preconditions  | Existing account; requester authenticated with the corresponding permission.                         |
| Postconditions | An INACTIVE account cannot authenticate or be assigned; its history remains consultable. |

**Business Rules**

- The lifecycle of entities is governed by the ACTIVE and INACTIVE states, that is, logical deletion rather than physical deletion.
- Deactivation blocks access without deleting the user's history.
- An INACTIVE profile cannot start a new authenticated session.

### 3.1.4 RF-01.4 — Linking Students to Guardian Account

The administrator links one or more students to a guardian's account so that the guardian can track all of their children from a single account.

**Table 13**. _Specification of requirement RF-01.4: linking students to a guardian's account_

| **Field**       | **Detail**                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------- |
| Inputs        | Guardian account identifier and identifiers of the students to link.                   |
| Outputs         | Guardian–student relationship recorded; error message if the student is already associated.              |
| Preconditions  | Existing and active guardian and student accounts; requester with user management permission. |
| Postconditions | The guardian views the information and notifications of the linked student.                       |

**Business Rules**

- A student can only be linked to a single active guardian account at a time.
- The guardian account must exist beforehand to receive links.
- The link enables consultation of the student's route and reception of their notifications.

## 3.2 Group RF-02 — Authentication 

This group specifies access to the system. The identity and access service issues JWT tokens signed with the RS256 algorithm and applies role-based access control; the access token expires after 15 minutes and the refresh token after 30 days.

### 3.2.1 RF-02.1 — Authentication and Token Issuance

The system authenticates the user with their credentials in accordance with the current password policy and, if they are valid and the account is active, issues a signed access token that limits the session to the functions permitted by their role.

**Table 14**. _Specification of requirement RF-02.1: authentication and token issuance_

| **Field**       | **Detail**                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Email address and password according to the policy (length, uppercase letters, numbers, symbols and expiration in days).         |
| Outputs         | JWT RS256 access token valid for 15 minutes with role claim; "invalid credentials" error without issuing a token. |
| Preconditions  | Account created by the administrator and in ACTIVE status.                                                                    |
| Postconditions | Session started and recorded with IP address, dates and application; user profile and session creation events are published. |

**Business Rules**

- Incorrect credentials or a deactivated account result in rejection and do not issue a token.
- The signing algorithm is RS256 and only the identity and access service holds the private key.
- Role-based access control is applied by default: without an explicit role and permission, there is no access.

### 3.2.2 RF-02.2 — Logout and Token Renewal

The user logs out and, while their refresh token remains valid, renews the access token without re-entering their credentials.

**Table 15**. _Specification of requirement RF-02.2: logout and token renewal_

| **Field**       | **Detail**                                                                                                            |
| --------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Valid refresh token (30 days) for renewal; access token for logout.                    |
| Outputs         | New 15-minute access token; session closed and its record finalized.                                          |
| Preconditions  | Session previously started and account still active.                                                                       |
| Postconditions | The previous access token is invalidated after logout; the recorded session stores its start and end dates. |

**Business Rules**

- Renewal is rejected if the refresh token is expired, has been revoked or the account is INACTIVE.
- Sessions are recorded with IP address and dates for consultation and auditing.
- The authentication and renewal routes are the only public ones; the rest of the interface requires a valid Bearer token.

## 3.3 Group RF-03 — School Route, Fleet & Assignments Management 

This group concentrates the operational configuration of transportation: definition of routes with stops and schedules, administration of the bus fleet, and establishment of bus, driver and student assignments. Route configuration and its assignments correspond to the route service, and bus administration and the driver's responsibility for each bus correspond to the fleet service.

### 3.3.1 RF-03.1 — Route Creation with Data, Ordered Stops & Schedules

The administrator creates a school route by entering its data, its stops in itinerary order and its schedules, so that drivers, students and families have a defined and consistent itinerary.

**Table 16**. _Specification of requirement RF-03.1: route creation with data, ordered stops and schedules_

| **Field**       | **Detail**                                                                                                                                                      |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Route name and sector, campus of belonging, stops with address and coordinates, stop order, day of the week, journey direction and scheduled time. |
| Outputs         | Route registered with its ordered stops and schedules; route visible in the listing.                                                                            |
| Preconditions  | Campus and master data of the institution available; requester authenticated as administrator.                                                                 |
| Postconditions | The route exists in a usable state and its stops preserve the itinerary order.                                                                              |

**Business Rules**

- A route must belong to a campus of an institution.
- Stops are ordered by itinerary, never by creation order, and cannot be repeated in the same position.
- A route whose required configuration is not complete cannot be activated.

### 3.3.2 RF-03.2 — Route Uniqueness Validation

Before persisting a route, the system validates that no other route with the same name exists in the corresponding scope.

**Table 17**. _Specification of requirement RF-03.2: route uniqueness validation_

| **Field**       | **Detail**                                                            |
| --------------- | ---------------------------------------------------------------------- |
| Inputs        | Proposed name for the route.                                         |
| Outputs         | Route persisted, or "the route already exists" message without a duplicate record. |
| Preconditions  | Route creation request received.                                |
| Postconditions | No two routes with the same name exist.                              |

**Business Rules**

- Uniqueness is evaluated by route name before any write.
- The duplicate attempt is reported to the user and the entered data is preserved.

### 3.3.3 RF-03.3 — Route Data Update

The administrator modifies the data of an existing route, its stops and its schedules, while maintaining the coherence of the current assignments.

**Table 18**. _Specification of requirement RF-03.3: route data update_

| **Field**       | **Detail**                                                                          |
| --------------- | ------------------------------------------------------------------------------------ |
| Inputs        | Route identifier and fields to modify.                                       |
| Outputs         | Route updated; rejection if the modification violates uniqueness or integrity.          |
| Preconditions  | Existing route and requester with permission.                                            |
| Postconditions | The changes take effect and are reflected in the driver and guardian queries. |

**Business Rules**

- The update cannot introduce duplicate stops in the same position or break the itinerary order.
- Every route configuration modification is recorded in the audit.

### 3.3.4 RF-03.4 — Route Data Deletion

The administrator deletes a route when it no longer operates, unless an active trip depends on it.

**Table 19**. _Specification of requirement RF-03.4: route data deletion_

| **Field**       | **Detail**                                                              |
| --------------- | ------------------------------------------------------------------------ |
| Inputs        | Identifier of the route to delete.                                     |
| Outputs         | Route deleted, or rejection message if there is a dependent active trip. |
| Preconditions  | Existing route; requester with permission.                                 |
| Postconditions | The route disappears from operation without leaving orphaned references.      |

**Business Rules**

- Deleting a route with an active trip is not allowed.
- The associated assignments are removed together with the route to preserve integrity.

### 3.3.5 RF-03.5 — Bus Assignment and Removal from Route

The administrator assigns a bus to a route and can unassign it, so that each route operates with an available vehicle.

**Table 20**. _Specification of requirement RF-03.5: assignment and removal of a bus from a route_

| **Field**       | **Detail**                                                                                         |
| --------------- | --------------------------------------------------------------------------------------------------- |
| Inputs        | Route and bus identifiers; assignment or unassignment action.                         |
| Outputs         | Route–bus relationship recorded or removed; bus status updated to "en route" when applicable. |
| Preconditions  | Active route and active bus without an incompatible current assignment.                                       |
| Postconditions | The route has at most one assigned bus and the bus serves at most one route.                    |

**Business Rules**

- A bus is assigned to a single route at a time and a route receives a single bus at a time.
- Buses in INACTIVE status are not assigned.
- The driver's route coverage is derived from the bus assigned to the route.

### 3.3.6 RF-03.6 — Bus Creation, Edition, Activation & Deactivation

The administrator creates, edits and activates or deactivates buses in the fleet, keeping the physical identity of the vehicle up to date for assignment to routes.

**Table 21**. _Specification of requirement RF-03.6: creation, edition and activation or deactivation of buses_

| **Field**       | **Detail**                                                                                  |
| --------------- | -------------------------------------------------------------------------------------------- |
| Inputs        | License plate, capacity, model and make, SOAT data, associated GPS device and initial status. |
| Outputs         | Bus registered with available status; error message if the license plate already exists.                |
| Preconditions  | Requester authenticated with fleet management permission.                                     |
| Postconditions | The bus appears in the listing and is eligible for assignment when active.               |

**Business Rules**

- The license plate is unique in the system and the capacity must be greater than zero.
- A bus belongs to an institution and a GPS device cannot be assigned to several buses simultaneously.
- An INACTIVE bus cannot be assigned to a route.

### 3.3.7 RF-03.7 — Driver Assignment and Removal from Bus

The administrator assigns a driver as responsible for a bus and can reassign or unassign them; the routes covered by that bus are covered by that driver.

**Table 22**. _Specification of requirement RF-03.7: assignment and removal of a driver from a bus_

| **Field**       | **Detail**                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------- |
| Inputs        | Bus and driver identifiers; assignment or unassignment action.                      |
| Outputs         | Driver–bus relationship recorded with its validity; assignment event published.                |
| Preconditions  | Driver with an active account and existing bus; requester with permission.                             |
| Postconditions | The driver responsible for the bus can consult the routes covered by their bus in the application. |

**Business Rules**

- A driver is responsible for a single bus at a time until the administrator reassigns or unassigns them.
- There are no drivers in INACTIVE status eligible for assignment.
- The driver reaches their routes transversally, through the bus assigned to each route; there is no direct driver–route assignment.

### 3.3.8 RF-03.8 — Student Assignment to Route with Pickup & Drop-off Stops

The administrator assigns students to a route by indicating the pickup stop and the drop-off stop, so that the system knows who is waiting, at which stop and at what time.

**Table 23**. _Specification of requirement RF-03.8: student assignment to a route with pickup and drop-off stops_

| **Field**       | **Detail**                                                                                          |
| --------------- | ---------------------------------------------------------------------------------------------------- |
| Inputs        | Route identifier, student identifier and pickup and drop-off stops.              |
| Outputs         | Route–student relationship recorded; assignment event published.                                 |
| Preconditions  | Active route, student with an active account and selected stops belonging to the route.          |
| Postconditions | The guardian can consult their student's route and the QR boarding flow is enabled. |

**Business Rules**

- The selected stops must belong to the route; otherwise it is rejected with the message "the stop does not belong to this route".
- A student keeps a single active route at a time.
- The assigned student must belong to the school context of the route.

## 3.4 Group RF-04 — QR Scanning Management: Boarding Control 

This group specifies the identification of the student through a QR code and the recording of their boardings. The individual code is generated by the identity and access service; the pickup and drop-off recording is processed by the notification service, which validates the operating conditions of the trip.

### 3.4.1 RF-04.1 — Individual Student QR Code Generation

The system generates for each student an individual QR code with a unique identifier, subject to a rotation and expiration policy, and displays it in the mobile application for visual verification.

**Table 24**. _Specification of requirement RF-04.1: generation of the student's individual QR code_

| **Field**       | **Detail**                                                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Student identifier with an active account; current rotation and expiration policy.                                      |
| Outputs         | Individual QR code with a unique identifier and the student's name for verification.                                         |
| Preconditions  | Student with an ACTIVE profile and their own authenticated session.                                                                       |
| Postconditions | The code is consultable while it is valid; when it expires or the profile is deactivated, the user is informed that the code is not available. |

**Business Rules**

- Each student has an individual code and a unique identifier.
- The code is rotated or regenerated according to its expiration policy and when credentials are replaced.
- Only the authenticated student themselves can consult their code.

### 3.4.2 RF-04.2 — QR Code Reading to Register Pickup

When the student's code is scanned, the system records their pickup with the time and the stop of the event, validating that the student is assigned to the route being operated.

**Table 25**. _Specification of requirement RF-04.2: QR code reading to register the pickup_

| **Field**       | **Detail**                                                                                                                                    |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | QR code scanned from the driver's application, active trip and route context.                                                     |
| Outputs         | Pickup boarding recorded with date and time and stop; scanning event published; rejection with the message "student not assigned to this route". |
| Preconditions  | Authenticated driver, active trip and student assigned to the route.                                                                           |
| Postconditions | A current pickup boarding exists for the student and the event that triggers the notification to the guardian is published.                  |

**Business Rules**

- The pickup only proceeds if the student is assigned to the route in progress.
- The boarding is mandatorily associated with student, bus and stop.
- The scan must be legible in low light and supports brief connectivity losses through a local queue with retry.

### 3.4.3 RF-04.3 — QR Code Reading to Register Drop-off

When the student's code is scanned at the destination stop, the system records their drop-off with the time and the location of the event, provided that an active pickup boarding exists.

**Table 26**. _Specification of requirement RF-04.3: QR code reading to register the drop-off_

| **Field**       | **Detail**                                                                                                                                           |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Scanned QR code, active trip and current pickup boarding of the student.                                                                         |
| Outputs         | Drop-off boarding recorded with date, time and location; drop-off event published; rejection with the message "there is no active boarding for the student". |
| Preconditions  | Active pickup boarding of the student on the trip in progress.                                                                                         |
| Postconditions | The student's boarding is closed and the event that notifies the guardian is published.                                                             |

**Business Rules**

- No drop-off is recorded without an active pickup on the same trip.
- The boarding type only accepts the values pickup and drop-off.
- The drop-off record preserves the date and time of the event, different from the creation date of the record.

## 3.5 Group RF-05 — Boarding Notifications 

This group specifies the automatic communication to the guardian. The notification service consumes the boarding events and delivers real-time notices, in addition to alerting abnormal situations during the journey.

### 3.5.1 RF-05.1 — Student Pickup Event Detection

The system detects and records the student's pickup event from the QR code scan, as a business fact that triggers the associated communications.

**Table 27**. _Specification of requirement RF-05.1: student pickup event detection_

| **Field**       | **Detail**                                                                               |
| --------------- | ----------------------------------------------------------------------------------------- |
| Inputs        | Scanning event coming from the boarding record.                                    |
| Outputs         | Pickup event detected and published for asynchronous processing.                   |
| Preconditions  | Pickup boarding record accepted.                                                   |
| Postconditions | The event is available for sending the notification and for the boarding history. |

**Business Rules**

- The event is derived exclusively from a valid boarding record, never from a manual capture.
- Detection is idempotent: the same event does not generate two notifications.

### 3.5.2 RF-05.2 — Automatic Pickup Notification to Guardian

The system automatically sends, in real time, a notification to the guardian with the pickup time and the corresponding stop.

**Table 28**. _Specification of requirement RF-05.2: automatic pickup notification to the guardian_

| **Field**       | **Detail**                                                                                     |
| --------------- | ----------------------------------------------------------------------------------------------- |
| Inputs        | Pickup event, guardians linked to the student, template and configured channel.           |
| Outputs         | Notification delivered to the guardian with pickup time and stop; delivery status record. |
| Preconditions  | Guardian linked to the student and with a device enabled to receive notifications.     |
| Postconditions | The notification is recorded with its delivery status and its date.                           |

**Business Rules**

- Only the guardians linked to the student who is the subject of the event are notified.
- Delivery is automatic: it does not require intervention by the driver or the administrator.
- Retries do not duplicate the original notification.

### 3.5.3 RF-05.3 — Automatic Drop-off Notification to Guardian

The system automatically sends, in real time, a notification to the guardian with the drop-off time and the location of the event.

**Table 29**. _Specification of requirement RF-05.3: automatic drop-off notification to the guardian_

| **Field**       | **Detail**                                                                           |
| --------------- | ------------------------------------------------------------------------------------- |
| Inputs        | Drop-off event, linked guardians, template and configured channel.               |
| Outputs         | Notification delivered with drop-off time and location; delivery status record. |
| Preconditions  | Drop-off successfully recorded and guardian linked.                                    |
| Postconditions | The guardian knows the time and place at which their student alighted from the bus.             |

**Business Rules**

- The communicated location corresponds to the stop or point recorded in the boarding.
- The message includes the data in a readable form, without exposing information about other students.

### 3.5.4 RF-05.4 — Prolonged Bus Stop Alert

When the bus remains stopped beyond the configured time threshold, the system alerts the guardian and includes the vehicle's location.

**Table 30**. _Specification of requirement RF-05.4: prolonged bus stop alert_

| **Field**       | **Detail**                                                                                                              |
| --------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Inputs        | Bus GPS position stream and stop threshold configurable per route.                                             |
| Outputs         | Prolonged stop alert sent to the guardian with the bus location; record of the alert and its recipients. |
| Preconditions  | Route in operation with an active position stream and configured threshold.                                                   |
| Postconditions | The guardian receives the alert with the location; if the bus resumes movement before the threshold, no alert is sent.        |

**Business Rules**

- The stop threshold is configurable per route and is never hard-coded.
- The alert is only issued when the threshold is exceeded; timely resumption of the trip suppresses the alert.
- Each alert declares its type and urgency level, and records the reading performed by each recipient.

## 3.6 Group RF-06 — Real-Time Bus Tracking 

This group specifies the capture and exposure of the bus location. Telemetry is received through the persistent gRPC channel on port 5001 and the positions are stored associated with the registered GPS device.

### 3.6.1 RF-06.1 — Continuous GPS Location Capture

The system continuously captures the geographic position of the bus during its operation, with its date and time, speed and heading.

**Table 31**. _Specification of requirement RF-06.1: continuous capture of the bus GPS location_

| **Field**       | **Detail**                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Positions emitted by the on-board GPS device (latitude, longitude, speed, heading, device date and time). |
| Outputs         | Position persisted with its reception timestamp on the server; position available for live tracking.            |
| Preconditions  | Registered GPS device, with a unique identifier, and associated with the bus.                                                   |
| Postconditions | The last known position of the bus is available for consultation.                                                     |

**Business Rules**

- A position always belongs to a registered GPS device; the device identifier is unique.
- Latitude must be between −90 and 90 and longitude between −180 and 180; speed cannot be negative.
- Only registered devices can contribute valid position data.

### 3.6.2 RF-06.2 — Interactive Map Location Visualization

The system displays the bus location on an interactive map that updates without the need for a manual refresh.

**Table 32**. _Specification of requirement RF-06.2: interactive map location visualization_

| **Field**       | **Detail**                                                                              |
| --------------- | ---------------------------------------------------------------------------------------- |
| Inputs        | Most recent bus positions of the queried route.                                  |
| Outputs         | Bus marker on the map with continuous updating.                               |
| Preconditions  | Guardian authenticated with a student assigned to a route, or driver with an assigned route. |
| Postconditions | The location is perceived as updated without user intervention.                        |

**Business Rules**

- The update arrives through a continuous stream, without manual reloading of the view.
- The guardian only consults the location of the routes of their linked students.

### 3.6.3 RF-06.3 — Alert for GPS Signal Loss

On loss of the GPS signal, the system alerts and shows the last known position with a "no signal" marker.

**Table 33**. _Specification of requirement RF-06.3: alert for GPS signal loss_

| **Field**       | **Detail**                                                                                                              |
| --------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Inputs        | Comparison between the last received position and the signal validity threshold.                                          |
| Outputs         | "No signal" status with the last known position; signal loss alert directed to the corresponding recipient. |
| Preconditions  | Position history of the device in operation.                                                                    |
| Postconditions | The user distinguishes between a current location and a location without signal.                                                      |

**Business Rules**

- The halt in reception for longer than the configured threshold is interpreted as signal loss.
- The alert preserves the last valid position and does not present invented data.
- The urgency level of the alert comes from the catalog of alert types.

## 3.7 Group RF-07 — Trip Management 

This group specifies the trip lifecycle, the operational unit that enables boardings and notifications with meaning.

### 3.7.1 RF-07.1 — Trip Lifecycle: Start and Completion

The driver starts and finishes their route's trip from the mobile application. The start marks the trip as active; the finish closes it by validating that there are no pending drop-offs.

**Table 34**. _Specification of requirement RF-07.1: trip lifecycle: start and completion_

| **Field**       | **Detail**                                                                                                                                              |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Current assigned route for the driver; "start trip" or "finish trip" action; time of each action.                                                |
| Outputs         | Trip started or finished with its timestamps; start and end events published; rejection with the message "a trip is already active for this bus". |
| Preconditions  | Driver authenticated with an assigned route; to finish, an active trip and known boarding status.                                                     |
| Postconditions | There is at most one active trip per bus; the finished trip leaves traceability of its execution with bus, driver and start and end.                     |

**Business Rules**

- A bus can only have one active trip at a time; a second start attempt is rejected.
- A trip is not finished if there remain pickup boardings without their corresponding drop-off.
- The trip start and end events are what enable the recording of boardings with operational meaning.
- Each route execution preserves the bus and the driver that served it.

## 3.8 Group RF-08 — Incidents 

This group specifies the communication of abnormal situations during the journey.

### 3.8.1 RF-08.1 — Incident Report and Automatic Notification

The driver reports an incident or emergency during the trip by indicating type, optional description and location; the system records the report and automatically notifies the institution and the guardians of the students traveling on board.

**Table 35**. _Specification of requirement RF-08.1: incident report and automatic notification_

| **Field**       | **Detail**                                                                                                                                                      |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Incident type from the configurable catalog, optional description, active trip and GPS location at the moment.                                                     |
| Outputs         | Incident recorded and linked to the trip; alert generated and notified to the institution and to the guardians of active boardings; rejection if there is no active trip. |
| Preconditions  | Driver authenticated with an active trip; incident type catalog available.                                                                               |
| Postconditions | There is traceability of the incident with its type, its moment, its location and its notified recipients.                                                         |

**Business Rules**

- The trip is a mandatory field: without an active trip the report is rejected.
- The GPS location is attached to the report at the moment of the event.
- The list of incident types is configurable by the institution, it is not hard-coded.
- The notification is directed to the institution and to the guardians of the students with an active boarding on that trip.
- The incident report is modeled as an alert with its type, its urgency level and its recipient record.

## 3.9 Group RF-09 — Route Inquiry 

This group specifies the consultation of the route configuration by the guardian and the driver.

### 3.9.1 RF-09.1 — Guardian Consultation of Their Student's Route

The guardian consults the details of their student's route: bus, driver, ordered stops and scheduled times.

**Table 36**. _Specification of requirement RF-09.1: guardian consultation of their student's route_

| **Field**       | **Detail**                                                                                                                         |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Identifier of the student linked to the guardian's account.                                                                   |
| Outputs         | Route detail: bus, driver, ordered stops and times; message "a route has not yet been assigned" when no assignment exists. |
| Preconditions  | Guardian authenticated and student linked to their account.                                                                           |
| Postconditions | The guardian knows the itinerary and the vehicle assigned to their student.                                                          |

**Business Rules**

- The guardian only accesses the routes of the students linked to their account, a verification performed on each consultation.
- The consultation delivers the route configuration, not the live position, which corresponds to real-time tracking.

### 3.9.2 RF-09.2 — Driver Consultation of Their Assigned Route

The driver views in the mobile application their assigned route with the ordered stops, the scheduled times and the corresponding bus.

**Table 37**. _Specification of requirement RF-09.2: driver consultation of their assigned route_

| **Field**       | **Detail**                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------------ |
| Inputs        | Identifier of the authenticated driver.                                                               |
| Outputs         | Current route with ordered stops, times and bus; message "no assigned route" when none corresponds. |
| Preconditions  | Driver authenticated with a current bus and route assignment.                                            |
| Postconditions | The driver knows their itinerary before starting the trip.                                           |

**Business Rules**

- The displayed route is derived from the bus assigned to the driver and from the route assigned to that bus.
- The stops are presented in itinerary order.
- The consultation does not require an active trip for its visualization.

## 3.10 Group RF-10 — Personalization 

This group specifies the adaptation of the interface to the user's preferences and to the institution's identity. Its scope is exclusively client-side: it does not concern any domain service.

### 3.10.1 RF-10.1 — System Color (Theme) Personalization

The authenticated user selects a theme color that is applied to all screens of the application and is preserved between sessions.

**Table 38**. _Specification of requirement RF-10.1: system color (theme) personalization_

| **Field**       | **Detail**                                                                            |
| --------------- | -------------------------------------------------------------------------------------- |
| Inputs        | Theme color chosen by the user with any role.                                |
| Outputs         | Theme applied in all views; preference persisted on the device.           |
| Preconditions  | User authenticated with an active account.                                                 |
| Postconditions | The change is immediately visible and survives the closing and reopening of the application. |

**Business Rules**

- The preference is stored on the client side, in the device's local configuration.
- The theme contrast must preserve the accessibility required by the interface.
- The change does not affect other users and does not require writing to domain services.

### 3.10.2 RF-10.2 — Multi-language Interface Configuration

The user selects the interface language from at least four supported languages, and the system shows the default language on first use.

**Table 39**. _Specification of requirement RF-10.2: interface configuration in at least four languages_

| **Field**       | **Detail**                                                                                                                  |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Inputs        | Identifier of the language chosen from the supported ones (Spanish, English, French and Portuguese).                                |
| Outputs         | Interface labels loaded from the dictionary of the chosen language; default language in the absence of a preference. |
| Preconditions  | Authenticated user; language dictionaries available in the application.                                                    |
| Postconditions | The language preference persists per device.                                                                           |

**Business Rules**

- The system supports at least four languages; Spanish is the default language of the first version.
- The dictionaries are resolved on the client and the preference is stored per device.
- Changing the language does not alter the stored data or the business messages already recorded.

# 4. Non-Functional Requirements

The eight non-functional requirements set verifiable metrics. Any unmeasured quality claim is considered insufficient: "the system must be fast" is not a requirement; "the P95 latency of the critical endpoints must be under 300 ms at 50 requests per second" is.

**Table 40**. _Summary of the non-functional requirements_

| **ID** | **Requirement**               | **Priority** | **Continuous integration validation**            |
| ------ | --------------------------- | ------------- | ------------------------------------------------- |
| RNF-01 | Performance                 | P1            | Yes, with load tests in the testing environment |
| RNF-02 | Availability              | P1            | Yes, health checks                       |
| RNF-03 | Scalability               | P2            | Manual, quarterly                       |
| RNF-04 | Security                   | P1            | Yes, static analysis and OWASP review            |
| RNF-05 | Observability              | P1            | Yes, smoke test                                |
| RNF-06 | Maintainability            | P2            | Yes, test coverage                          |
| RNF-07 | Portability                | P2            | Yes, image build                        |
| RNF-08 | Disaster recovery | P1            | Manual, with drills                            |

## 4.1 RNF-01 Performance

**Table 41**. _Performance metrics and verification conditions_

| **Metric**                                            | **Target value**                      | **Verification condition**                                                                |
| ------------------------------------------------------ | --------------------------------------- | -------------------------------------------------------------------------------------------- |
| P95 latency of critical endpoints                     | < 300 ms                                | 50 requests per second per service, in the peak boarding window from 6:30 to 8:00 a.m. |
| P99 latency of critical endpoints                     | < 500 ms                                | Peak window conditions                                                                  |
| P95 latency of non-critical endpoints                  | < 1000 ms                               | Normal load                                                                                 |
| Minimum sustained throughput                           | 50 requests per second per service | Without response degradation                                                                 |
| Boarding or drop-off event → notification to the guardian | P95 < 5 seconds                        | End to end, during the peak window                                                   |
| GPS position → live map                            | P95 < 5 seconds                        | Continuous tracking                                                                         |
| Service startup time                      | < 30 seconds                           | Cold start                                                                             |

Critical endpoints subject to the metric: the boarding record by QR scanning (POST /api/v1/boarding), the bus location query (GET /api/v1/buses/{busId}/location) and the login (POST /api/v1/auth/login). Load tests are run with tools from the k6, Apache JMeter, Locust or Gatling ecosystem and are validated in the integration and deployment pipeline before production.

## 4.2 RNF-02 Availability

**Table 42**. _Service level objectives per environment_

| **Environment** | **SLO** | **Maintenance window**  | **Maximum monthly downtime** |
| ----------- | ------- | ----------------------------- | ------------------------ |
| Production  | 99.9%  | Sundays from 2:00 to 4:00 a.m. | 44 minutes               |
| Testing     | 95%    | Unrestricted               | 36 hours                 |

The monthly error budget in production is 44 minutes. If more than 50% of the budget is consumed in the first half of the month, feature deployments are frozen until the following month and stability is prioritized. Each service exposes health checks: a liveness check (GET /health) that responds while the process is active, and a readiness check (GET /health/ready) that responds only if the service can process traffic with its dependencies connected; in the Java services the actuator health endpoint (/actuator/health) is also published.

## 4.3 RNF-03 Scalability

**Table 43**. _Scalability scenarios and expected behavior_

| **Scenario**                   | **Expected behavior**                                                        |
| ------------------------------- | ---------------------------------------------------------------------------------- |
| Gradual load growth | Activation of horizontal autoscaling when CPU utilization exceeds 70% |
| Sudden demand spike          | Scaling in less than 2 minutes                                                     |
| Decrease in load            | Reduction of instances without interrupting active traffic                          |
| Horizontal scaling limit   | Up to 10 instances per service                                                   |

The strategy is horizontal scaling of stateless instances: no instance keeps state in memory; persistent data resides in SQL Server. The architecture supports up to 5,000 concurrent users without degradation.

## 4.4 RNF-04 Security

**Table 44**. _Measurable security requirements per area_

| **Area**              | **Measurable requirement**                                                                                                                                                                           |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Authentication         | Every private endpoint requires a valid Bearer token in the Authorization header; JWT RS256 access token valid for 15 minutes and a 30-day refresh token.                       |
| Authorization          | Role-based access control with explicit permissions and default denial; least privilege per role.                                                                                       |
| Transmission           | Mandatory HTTPS in production with TLS 1.2 or higher; HTTP only in local development.                                                                                                    |
| Passwords           | Storage with irreversible hash (bcrypt with cost ≥ 12 or Argon2id); the policy sets length, uppercase letters, numbers, symbols and expiration in days; the hash is never written to logs. |
| Personal data      | Personally identifiable data is encrypted at rest and its logging is minimized.                                                                                                             |
| Secrets              | Credentials and keys only in environment variables or a secrets manager, never in the source code.                                                                                       |
| Security review | Review against the OWASP Top 10 on every release, with static analysis, dependency scanning and dynamic analysis in testing.                                                                 |
| Compliance          | Ley 1581 de 2012 and Decreto 1377 de 2013, with consent and authorization for the processing of minors' data; GDPR if the product is offered in the European Union.                          |

The trust surface is organized into four zones: public (clients toward the gateway), services (internal communication), data (database and cache, with no public access) and administration (network-restricted). Between services, gRPC communication is protected with mutual network trust and a circuit breaker; the event bus topics apply topic-level access control.

## 4.5 RNF-05 Observability

**Table 45**. _Observability requirements and tools_

| **Pillar** | **Requirement**                                                                                              | **Tool**                                      |
| --------- | ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| Logs | Structured JSON format with a correlation identifier propagated across all traces of the transaction | Serilog in .NET services; Logback in Java services |
| Metrics  | Rate, error and duration indicators per endpoint                                                       | Prometheus + Grafana                                 |
| Traces    | End-to-end distributed trace                                                                     | OpenTelemetry + Jaeger                               |
| Alerts   | Operational alert in less than 5 minutes when the service level objective is violated                    | Alertmanager or an external alerting service            |

Every external request generates a unique correlation identifier that is propagated in its logs and traces, which makes it possible to reconstruct a complete transaction among client, gateway and services.

## 4.6 RNF-06 Maintainability

**Table 46**. _Maintainability objectives_

| **Metric**                  | **Objective**                                                                                              |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| Test coverage         | ≥ 80% of lines; ≥ 90% in the domain layer                                                            |
| Cyclomatic complexity      | ≤ 10 per function                                                                                          |
| Technical debt                | Resolution in less than one development cycle from its recording                                           |
| Onboarding time      | A new developer deploys locally in less than 1 hour by following the environment setup guide |
| Average build time | < 5 minutes in the integration pipeline                                                             |

## 4.7 RNF-07 Portability

**Table 47**. _Portability requirements_

| **Requirement**     | **Detail**                                                                              |
| ----------------- | ---------------------------------------------------------------------------------------- |
| Containers      | All services are deployed as container images                            |
| Orchestration      | The images operate in any environment with Kubernetes 1.28 or higher                  |
| Operating system | No service depends on the host operating system                              |
| Configuration     | Environment variables are the only source of configuration specific to each environment |

## 4.8 RNF-08 Disaster Recovery

**Table 48**. _Disaster recovery objectives per scenario_

| **Scenario**                         | **RTO**                                       | **RPO**                        |
| ------------------------------------- | --------------------------------------------- | ------------------------------ |
| Failure of a service                  | < 2 minutes (automated restart)           | 0 (stateless service)        |
| Failure of the primary database    | < 5 minutes (failover to replica)           | < 1 second (synchronous replica) |
| Loss of an availability zone | < 15 minutes                                  | < 5 minutes                    |
| Total region disaster           | < 4 hours (recovery in a secondary region) | < 1 hour                       |

# 5. Interface Requirements

## 5.1 User Interface

**Web application (Angular 21 + Angular Material 3).** It presents a top bar, a side navigation menu according to the role, tables with pagination, forms with inline validation, _snackbar_-type feedback messages and indicator cards on the main panel. The views are organized into administration panel, platform operation panel, user management, campuses and courses, fleet, routes and profile. The design is responsive to desktop, tablet and mobile, and respects the design tokens (color, typography and density) defined for the interface.

**Mobile application (React + Expo).** Navigation differs by role: the driver finds their route of the day, the QR scanner, the incident report and the connection status; the guardian finds the route tracking, the notifications and the boarding history; the student finds their personal QR code. The interface explicitly reports the connected and disconnected states.

**Common interactions.** Explicit confirmation for accepted or irreversible actions; consultable listings with filters; consistent error messages in the chosen language; color contrast verifiable against the accessibility criteria; at least four interface languages and theme personalization per user.

## 5.2 Hardware Interface

**Table 49**. _Required hardware interface_

| **Component**            | **Required interface**                                                                                                                               |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Desktop computer      | Web browser with support for HTML5, CSS and script execution; printer not required.                                                              |
| Mobile device         | Android or iOS with a camera for QR code scanning and push notification support.                                                             |
| On-board GPS device | Emitter of geographic positions through a persistent TCP connection toward the geolocation component, with a unique device identifier (IMEI). |
| Server                  | SQL Server instance reachable on port 1433 within the services network; not publicly exposed.                                        |
| Network                       | Stable internet connection for clients; data connectivity for receiving positions and sending notifications.                       |

## 5.3 Software Interface

**Table 50**. _Required software interactions_

| **Software**                     | **Required interaction**                                                                         |
| -------------------------------- | ------------------------------------------------------------------------------------------------- |
| SQL Server 2022                  | Relational persistence with schema per service and versioned migrations.                       |
| Apache Kafka 4.1.0 (KRaft)       | Publication and consumption of domain events on port 9092.                                    |
| API Gateway (Kong OSS)           | TLS termination, rate limiting, CORS and routing toward the services.                                |
| Liquibase                        | Deterministic application of schema changes at every service startup.                       |
| Angular 21 web application        | Exclusive consumption of the versioned REST interface through the gateway.                            |
| React + Expo mobile application    | Consumption of the REST interface, notification token registration and camera access.              |
| Maps service                | Representation of the interactive map and of the bus location in real time.                     |
| Push notification provider | Delivery of notices on the mobile devices, identified by device token per profile. |

## 5.4 Communications Interface

**REST/JSON.** All client–service interactions are performed over HTTP with the /api/v1 prefix, plural resources, standard HTTP verbs, filtering, sorting and pagination by page and size. The Bearer token is mandatory except on the login and renewal routes. Error responses follow a consistent structure with timestamp, message and details or requested path. The gateway applies rate limiting and restricts the CORS origin to the web domain.

**gRPC.** Persistent channel on port 5001 for the low-latency telemetry and geolocation stream between the position reception component and the location consumers; the communication supports a circuit breaker on failures.

**Events (Apache Kafka, port 9092).** The event bus transports the six topics for user creation and profile and for QR scanning:

**Table 51**. _Event bus topics and their participants_

| **Topic** | **Producer** | **Consumer** | **Use** |
| --- | --- | --- | --- |
| `student.created` | User management service | Subscribed services | Student creation in the ecosystem |
| `driver.created` | User management service | Subscribed services | Driver creation |
| `admin.created` | User management service | Subscribed services | Administrator creation |
| `parent.created` | User management service | Subscribed services | Guardian creation |
| `profile.created` | Identity and access service | Subscribed services | Authentication profile creation |
| `student.scanned` | Identity and access service | Notification service | QR scan that triggers the notification and the boarding record |

Additionally, the services emit internal domain events —route creation and update, bus and driver assignment, trip start and end, pickup and drop-off, alert creation— that are processed asynchronously and idempotently within the event-driven model.

**Push notifications.** The notification service delivers notices to the devices through the notification token registered per profile; each delivery preserves its status (queued, sent, failed, read) and its date.

**Table 52**. _Communication resources, ports and purpose_

| **Communication resource** | **Port** | **Purpose**                        |
| --------------------------- | ---------- | ------------------------------------ |
| API Gateway (Kong)          | 8000       | Public client entry point |
| Business services        | 8080       | Internal REST interface of services   |
| SQL Server                  | 1433       | Relational persistence              |
| Apache Kafka                | 9092       | Event bus                       |
| gRPC                        | 5001       | Geolocation telemetry        |

# 6. Data Requirements

## 6.1 Logical Data Model (Summary)

The software manages **38 tables** distributed across **11 schemas**, one per service. A single relational instance concentrates the information with logical isolation per schema; no service accesses another's schema directly and, when it needs its data, it obtains it through an interface or an event.

**Table 53**. _Database schemas and their key content_

| **Schema** | **Tables and key content** |
| --- | --- |
| `Iam` | Profile (person, campus, password hash and role), Role, Permissions, PermissionsRole, SessionProfile |
| `UserManagement` | Person (name, identification type and number, email, phone, address, date of birth), Family, FamilyMember (guardian–student relationship), DriverLicense (number and expiration) |
| `School` | School (city, logo, contact data, theme), SchoolCampus, Course, SchoolAdmin |
| `Geographic` | City (name and country) |
| `Fleet` | Bus (license plate, capacity, SOAT, GPS device, model), Brand, Model, DriverAssignemts (driver–bus with validity) |
| `Route` | Route (campus, sector, schedule), Stop (coordinates with seven-decimal precision), RouteStop (order), RouteBusAssignments, RouteStudentAssignments, RouteSchedule (day of the week, direction and time), RouteExecution (actual execution: bus, driver, start and end) |
| `Gps` | GpsDevice (IMEI, last connection, status), GpsLocation (latitude, longitude, speed, heading, date and time) |
| `Notification` | Boarding (boarding with type and date and time), Alert, AlertType (urgency level), AlertRecipient (reading), DeviceToken (notification token per profile) |
| `Exceptional` | ExceptionalDriverUsage (bus use by third parties with reason), ExceptionalRouteUsage (route deviation with reason and window) |
| `Settings` | PasswordPolicies, SecuritySettings (parameter and value) |
| `Audit` | Audit (action, date, description, IP and application), LogErrors (type, description, IP and date) |

## 6.2 Integrity and Validation Rules

**Table 54**. _Integrity and validation rules_

| **#** | **Rule**                                                                                                                                                  |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Primary keys are implemented as unique identifiers, except for small type catalogs and audit counters.                        |
| 2     | Every external reference column follows the naming convention of the referenced entity plus the identification suffix.                               |
| 3     | Dates are stored with date and time precision and in coordinated universal time; local time without offset is not stored.                     |
| 4     | A person's document number is unique; a bus's license plate is unique; a GPS device's IMEI identifier is unique.                         |
| 5     | A route belongs to a campus; a bus belongs to an institution; a stop belongs to a route or to its reference institution.                       |
| 6     | A bus is assigned to a single route at a time; a driver is responsible for a single bus at a time; a student keeps a single active route.               |
| 7     | A boarding mandatorily references student, bus and stop, and its type only accepts pickup or drop-off.                                                   |
| 8     | A geographic position references a registered device; latitude is between −90 and 90, longitude between −180 and 180 and speed is not negative.  |
| 9     | There are no physical cross-references between schemas: cross-relations are resolved by identifier or code in the application and their consistency is eventual. |
| 10    | Filter and join columns have an index; time series are indexed by device and descending date.                                |
| 11    | Business uniqueness is enforced with unique constraints and range rules with check constraints.                                       |

## 6.3 Data Lifecycle and Retention

- **Logical deletion.** Entities for administrative use are deactivated through ACTIVE/INACTIVE status and a logical deletion flag, without physical deletion, so that the history remains consultable.
- **Auditing.** The audit history is insert-only: it does not allow update or deletion from the application and is preserved with its timestamp in coordinated universal time.
- **GPS telemetry.** High-frequency positions do not use logical deletion: their history is managed through configurable retention jobs, and this document does not set a number of retention days.
- **Events and notifications.** Each boarding, alert and notification delivery preserves its occurrence and creation timestamps, which may differ when the data comes from a device without connectivity.
- **Schema changes.** Every structural change is versioned as a migration applied by the pipeline; manual modifications are never executed on the production environment, and incompatible changes follow the sequence of expansion, data migration and contraction.
- **Backups.** The database is backed up in accordance with the environment's operating plan and its restoration is rehearsed within the recovery objectives of section 4.8.

## 6.4 Personal Data Protection

**Table 55**. _Personal data protection requirements_

| **Requirement**     | **Detail**                                                                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Classification     | Email, name, document number, family links and live location are treated as sensitive information.                                                  |
| Minimization      | Personal information is logged only to the extent necessary for operation and auditing.                                                     |
| Encryption           | Personally identifiable data is encrypted at rest and travels encrypted via TLS.                                                                           |
| Access control | Consultation is restricted by role and by institutional scope: the guardian only accesses their linked students and the administrator the scope of their institution. |
| Consent    | The processing of minors' data requires the express consent and authorization of the guardian.                                                                      |
| Traceability      | Every critical action on personal data is recorded with who, what, when, IP address and application.                                                     |

# 7. User Stories

## 7.1 Epics

The product backlog is organized into **7 epics** that group **20 user stories**.

**Table 56**. _Product backlog epics_

| **Epic** | **Title**              | **Description**                                                                   | **HU** |
| --------- | ----------------------------- | --------------------------------------------------------------------------------- | ------ |
| EP-001    | Pickup and Drop-off via QR   | Records the student's pickup and drop-off through scanning their QR code | 3      |
| EP-002    | Notifications                | Notifies the guardian of the pickup and drop-off and the route alerts              | 2      |
| EP-003    | Route & Fleet Management      | Manages routes, buses, stops and driver and student assignments        | 8      |
| EP-004    | User Management and family | Manages users and links students to the guardian's account               | 3      |
| EP-005    | Live Tracking           | Shows the bus location on the map in real time                            | 1      |
| EP-006    | Incidents                    | Reports and notifies the incidents occurring during the trip                      | 1      |
| EP-007    | Interface & Personalization    | Personalizes the color theme and the interface language                         | 2      |

## 7.2 User Stories by Epic

### 7.2.1 Epic EP-001 — Pickup and Drop-off via QR

**Table 57**. _User Stories of epic EP-001_

| **HU** | **Title** | **Role** | **Platform** | **Priority** | **Acceptance Criteria (summarized)** |
| --- | --- | --- | --- | --- | --- |
| HU-QR-001 | The driver scans the student's QR code to register their pickup | `DRIVER` | Mobile | High | With the student assigned to the route and a registered stop, the scan registers the pickup with time and stop and publishes the pickup event; if the student belongs to another route, it is rejected with the message "student not assigned to this route". |
| HU-QR-002 | The driver scans the student's QR code to register their drop-off | `DRIVER` | Mobile | High | With an active pickup boarding on the trip, the scan registers the drop-off with time and location and publishes the drop-off event; without an active boarding, it is rejected with the message "there is no active boarding for the student". |
| HU-QR-004 | The student consults their personal QR code in the mobile application | `STUDENT` | Mobile | Medium | With an active account, the section shows the code with a unique identifier and the student's name for verification; if the code is expired or the account is deactivated, it reports that the code is not available. |

### 7.2.2 Epic EP-002 — Notifications

**Table 58**. _User Stories of epic EP-002_

| **HU** | **Title** | **Role** | **Platform** | **Priority** | **Acceptance Criteria (summarized)** |
| --- | --- | --- | --- | --- | --- |
| HU-NOTIF-001 | Real-time pickup and drop-off notifications to the guardian | `PARENT` | Mobile | High | When the pickup is scanned, the guardian immediately receives the time and the stop; when the drop-off is scanned, they receive the time and the location of the drop-off. |
| HU-NOTIF-005 | Alert when the bus remains stopped longer than allowed | `PARENT` | Mobile | Medium | When the configured stop threshold is exceeded, the alert is sent with the bus location; if the bus resumes movement within the threshold, no alert is sent. |

### 7.2.3 Epic EP-003 — Route & Fleet Management

**Table 59**. _User Stories of epic EP-003_

| **HU** | **Title** | **Role** | **Platform** | **Priority** | **Acceptance Criteria (summarized)** |
| --- | --- | --- | --- | --- | --- |
| HU-ROUTE-001 | The administrator creates and manages school routes with stops and schedules | `ADMIN` | Web | High | The route is registered with ordered stops and schedules and appears in the listing; a duplicate name is rejected with the message "the route already exists". |
| HU-ROUTE-002 | The driver consults their assigned route with stops and schedules | `DRIVER` | Mobile | High | With an assigned and active route, the view shows ordered stops, schedules and the bus; without an assigned route, it reports "no assigned route". |
| HU-ROUTE-003 | The driver starts and finishes their route's trip | `DRIVER` | Mobile | Medium | The start marks the trip active and publishes the corresponding event; the finish validates that there are no pending drop-offs; a second concurrent start is rejected with the message "a trip is already active for this bus". |
| HU-ROUTE-004 | The guardian consults the details of their child's route | `PARENT` | Mobile | Medium | With a linked student and an assigned route, the view shows bus, driver, stops and schedules; without an assigned route, it reports "a route has not yet been assigned". |
| HU-ROUTE-005 | The administrator assigns a bus to a route | `ADMIN` | Web | Medium | The bus is linked to the route and the bus status changes to "en route"; if the route already has an active bus, it is rejected with the message "the route already has an assigned bus". |
| HU-ROUTE-008 | The administrator assigns students to a route with their pickup and drop-off stop | `ADMIN` | Web | High | The student is linked to the route and the guardian can consult it; a stop unrelated to the route is rejected with the message "the stop does not belong to this route". |
| HU-FLEET-001 | The administrator manages the bus fleet | `ADMIN` | Web | Medium | The bus is registered with available status and appears in the listing; a repeated license plate is rejected with the message "the license plate already exists". |
| HU-BUS-002 | The administrator assigns and unassigns a driver to a bus | `ADMIN` | Web | High | The driver is registered as responsible for the bus and sees their bus's routes in the application; if they are already responsible for another bus, it is rejected with the message "driver already assigned to a bus". |

### 7.2.4 Epic EP-004 — User Management and family

**Table 60**. _User Stories of epic EP-004_

| **HU** | **Title** | **Role** | **Platform** | **Priority** | **Acceptance Criteria (summarized)** |
| --- | --- | --- | --- | --- | --- |
| HU-IAM-001 | Sign-in, accounts and roles (role-based access control) | All roles | Web and mobile | High | With valid credentials and an active account, the system issues the token and grants access limited to the role's functions; with incorrect credentials or a deactivated account, it rejects with the message "invalid credentials" and does not issue a token. |
| HU-USER-003 | The administrator links students to a guardian's account | `ADMIN` | Web | High | The student is linked and the guardian consults their information and notifications; a student already linked to another guardian is rejected with the message "student already associated". |
| HU-USER-004 | The administrator manages the accounts of the school ecosystem | `ADMIN` | Web | High | The account is created with the corresponding role and appears in the listing; when an account is deactivated, it no longer accesses the system and the user becomes unavailable for assignments. |

### 7.2.5 Epic EP-005 — Live Tracking

**Table 61**. _User Stories of epic EP-005_

| **HU** | **Title** | **Role** | **Platform** | **Priority** | **Acceptance Criteria (summarized)** |
| --- | --- | --- | --- | --- | --- |
| HU-GPS-002 | Bus location on the map in real time | `PARENT` | Mobile | Medium | With a student assigned to a route, the view shows the updated location without manual refresh; on GPS signal loss, it shows the last known position with a "no signal" marker. |

### 7.2.6 Epic EP-006 — Incidents

**Table 62**. _User Stories of epic EP-006_

| **HU** | **Title** | **Role** | **Platform** | **Priority** | **Acceptance Criteria (summarized)** |
| --- | --- | --- | --- | --- | --- |
| HU-INCIDENT-001 | The driver reports an incident or emergency during the trip | `DRIVER` | Mobile | Medium | With an active trip, the report records type, optional description, trip and location, and notifies the institution and the guardians of the students on board; without an active trip, the trip field is mandatory and the report is rejected. |

### 7.2.7 Epic EP-007 — Interface & Personalization

**Table 63**. _User Stories of epic EP-007_

| **HU**    | **Title**                                                      | **Role**         | **Platform** | **Priority** | **Acceptance Criteria (summarized)**                                                                                                          |
| --------- | --------------------------------------------------------------- | --------------- | -------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| HU-UI-001 | The user personalizes the color theme of the application    | All roles | Web and mobile    | Low          | The color change is applied to all screens and is preserved when closing and reopening the application.                                              |
| HU-UI-002 | The user chooses the interface language from at least four | All roles | Web and mobile    | Low          | When choosing a supported language, the labels are loaded from its dictionary; without a saved preference, the interface uses the default language. |

# 8. General Acceptance Criteria and Verification Strategy

## 8.1 General Acceptance Criteria

A software increment is considered accepted when it jointly meets the following criteria:

**Table 64**. _General acceptance criteria_

| **#** | **Criterion**                                                                                                                                           |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | Every delivered function is traced to at least one functional requirement in section 3 and to one user story in the appendix.                           |
| 2     | The user story's acceptance criteria are verified in Given/When/Then format, with an observable and reproducible result.                |
| 3     | The business rules of each requirement are also met in error scenarios, not only on the happy path.                                     |
| 4     | Input validations reject invalid data with a clear message and without losing the information entered.                                           |
| 5     | Every private endpoint requires a valid token and applies role-based access control; without explicit permission, the operation is denied.                        |
| 6     | Critical actions are recorded in the audit trail with who, what, when, IP address and application.                                               |
| 7     | The interface preserves the contrast, keyboard navigation and responsive presentation required by the design.                                      |
| 8     | Tests do not use real personal data: test records use fictitious data and credentials marked as &lt;PASSWORD_DE_PRUEBA&gt;. |
| 9     | Existing test coverage is not reduced: added code increases or preserves overall coverage.                                           |
| 10    | The interface and data documentation is updated in the same delivery when the contract or the meaning of a field changes.                       |

## 8.2 Verification Strategy

Verification is organized into complementary levels, with greater volume at the base of the pyramid, where tests are faster and cheaper:

**Table 65**. _Levels of the verification strategy_

| **Level**            | **Objective**                                                 | **Typical scope**                                                               | **Tools**                                                                                                     |
| -------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Unit             | Test business logic in isolation                 | Aggregates, value objects, domain services and use cases                 | JUnit 5 with Mockito in Java; xUnit in .NET; Jest and Vitest in Angular; Jest with React Native Testing Library on mobile |
| Integration       | Verify adapters against real systems                 | Persistence layer against the database, publishers against the event bus | Containerized test environments with base and API integration tests                                      |
| Contract          | Confirm that the implementation meets the published interface | HTTP interface against the service contract                                     | Contract validators and verification of the published schema                                                         |
| End-to-end | Validate complete flows as experienced by the user     | Sign-in, route management, boarding and guardian inquiry             | Playwright and Cypress for the web interface; k6 for end-to-end interface tests                             |
| Performance          | Confirm the objectives of section 4.1                    | Critical endpoints under sustained load                                          | k6, failing if P95 latency exceeds 300 ms or the error rate exceeds 1%                                      |
| Regression and smoke     | Protect what already works after each change                  | Critical flows and service health checks                         | Regression suite on every delivery and smoke test in the test environment                                      |

Unit tests run on every code delivery; integration and contract tests in the pipeline; end-to-end and performance tests in the test environment, before publication to production. Defects are logged with severity and traceability to the functional requirement that originates them, and their closure requires the test that reproduces them.

## 8.3 Quality Thresholds and Metrics

**Table 66**. _Quality thresholds and measurement points_

| **Metric**                        | **Threshold**                                  | **Measurement point**                                          |
| ---------------------------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| Test line coverage      | ≥ 80% overall; ≥ 90% in the domain layer | On every delivery; a delivery that reduces coverage is rejected |
| Branch coverage                 | ≥ 75% overall; ≥ 85% in the domain layer | On every delivery                                                  |
| P95 latency of critical endpoints | < 300 ms                                    | Load test in the test environment                         |
| Error rate under load           | < 1%                                       | Load test in the test environment                         |
| Average build time       | < 5 minutes                                 | Integration pipeline                                      |
| Cyclomatic complexity            | ≤ 10 per function                            | Static analysis                                                |
| OWASP Top 10 review              | No pending critical or high findings  | Every release                                                  |
| Availability                     | SLO 99.9% monthly                          | Production environment monitoring                              |

# 9. Glossary

## 9.1 Domain Glossary

**Table 67**. _Domain Glossary_

| **Term**            | **Definition**                                                                                                            |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Guardian              | Adult responsible for one or more students, authorized to consult their transport information.                      |
| Alert                 | High-priority communication generated by an operational event or condition, with type, urgency level and recipients. |
| Route assignment     | Relationship that links a route with a student, a bus, a driver or a stop.                                       |
| Boarding                | Record of a student's pickup or drop-off event on the bus, with its date and time.                                |
| Bus                    | Vehicle authorized for school transport, identified by its license plate and capacity.                                 |
| Route deviation     | Condition in which the vehicle departs from the planned route; a security event subject to monitoring.                  |
| Driver              | Person responsible for operating an assigned bus and route.                                                                 |
| Delay                | Condition in which the vehicle runs behind schedule.                                             |
| School                | Educational institution that owns the transport service, with campuses and grades.                                             |
| Student             | Minor transported, associated with a family, a route and a stop; their data requires enhanced protection.              |
| Family                | Linking structure between students and their guardian or responsible adults.                                        |
| Incident              | Operational or security event that affects a vehicle, route, driver, student or stop.                                |
| Monitoring              | Observation of the route's progress, the vehicle's status, its location and operational events.                       |
| Notification           | Message sent to the user about a relevant event of the trip.                                                       |
| Stop                 | Physical location where students board or alight from the bus.                                                             |
| Route                   | Planned school journey that links the institution with its stops, with assigned vehicles and students.               |
| Campus                   | Physical location of an institution, the basis for the ownership of routes, buses and profiles.                                    |
| Traceability           | Ability to know what happened, when and in relation to which user, vehicle, route or event.                            |
| Vehicle location | Current or last known geographic position of a bus.                                                                   |
| Trip                  | Actual execution of a route, with bus, driver, start time and end time.                                             |

## 9.2 Technical Glossary

**Table 68**. _Technical Glossary_

| **Term**             | **Definition**                                                                                            |
| ----------------------- | --------------------------------------------------------------------------------------------------------- |
| API Gateway             | Single entry point that applies TLS, rate limiting, CORS, authentication and routing to the services.   |
| Bearer token            | Credential carried in the Authorization header of every private request.                         |
| Soft delete             | Deactivation of a record by means of a status, without physical deletion.                                     |
| Correlation identifier  | Technical datum: unique identifier propagated through the logs and traces of the same transaction.         |
| Schema per service    | Pattern in which each service has its own schema within a shared database.            |
| Domain event       | Relevant business fact published for processing by other components.                           |
| gRPC                    | Low-latency synchronous communication protocol used for positioning telemetry.        |
| Password hash      | Irreversible representation of the password; the original value is never stored or logged.         |
| Circuit breaker | Mechanism that isolates a failed dependency and prevents its continued invocation.                                 |
| Kafka (KRaft)           | Event bus in native operating mode, without an external coordination service.                         |
| Liquibase               | Database migration tool with versioned changes and deterministic order.                   |
| Least privilege       | Principle whereby each role receives only the permissions it needs.                             |
| Payload                 | Set of data carried by a request or a message.                                            |
| Health endpoint          | Endpoint that reports whether the service is alive and able to receive traffic.                                 |
| Rate limit              | Limit of requests per interval applied at the entry point.                                      |
| RS256                   | Token signing algorithm with a key pair; only the identity service holds the private key. |
| SLO                     | Service level objective, such as the 99.9% monthly availability.                                 |
| Refresh token  | Long-lived credential that allows renewing the access token without repeating credentials.             |
| Topic                  | Logical channel of the event bus that groups messages of the same type.                                   |
| Stop threshold     | Configured maximum time the bus may remain stationary before generating an alert.                          |

# 10. Appendix: US↔FR Traceability Matrix

## 10.1 User Story to Functional Requirement Matrix

**Table 69**. _User story to functional requirement traceability matrix_

| **#** | **HU**          | **Epic** | **Implemented FRs**               | **Functional group** |
| ----- | --------------- | --------- | ---------------------------------- | ------------------- |
| 1     | HU-IAM-001      | EP-004    | RF-02.1, RF-02.2                   | RF-02               |
| 2     | HU-USER-004     | EP-004    | RF-01.1, RF-01.2, RF-01.3          | RF-01               |
| 3     | HU-USER-003     | EP-004    | RF-01.4                            | RF-01               |
| 4     | HU-ROUTE-001    | EP-003    | RF-03.1, RF-03.2, RF-03.3, RF-03.4 | RF-03               |
| 5     | HU-FLEET-001    | EP-003    | RF-03.6                            | RF-03               |
| 6     | HU-ROUTE-005    | EP-003    | RF-03.5                            | RF-03               |
| 7     | HU-BUS-002      | EP-003    | RF-03.7                            | RF-03               |
| 8     | HU-ROUTE-008    | EP-003    | RF-03.8                            | RF-03               |
| 9     | HU-QR-004       | EP-001    | RF-04.1                            | RF-04               |
| 10    | HU-QR-001       | EP-001    | RF-04.2, RF-05.1                   | RF-04, RF-05        |
| 11    | HU-QR-002       | EP-001    | RF-04.3                            | RF-04               |
| 12    | HU-NOTIF-001    | EP-002    | RF-05.2, RF-05.3                   | RF-05               |
| 13    | HU-NOTIF-005    | EP-002    | RF-05.4                            | RF-05               |
| 14    | HU-GPS-002      | EP-005    | RF-06.1, RF-06.2, RF-06.3          | RF-06               |
| 15    | HU-ROUTE-003    | EP-003    | RF-07.1                            | RF-07               |
| 16    | HU-INCIDENT-001 | EP-006    | RF-08.1                            | RF-08               |
| 17    | HU-ROUTE-004    | EP-003    | RF-09.1                            | RF-09               |
| 18    | HU-ROUTE-002    | EP-003    | RF-09.2                            | RF-09               |
| 19    | HU-UI-001       | EP-007    | RF-10.1                            | RF-10               |
| 20    | HU-UI-002       | EP-007    | RF-10.2                            | RF-10               |

## 10.2 Functional Requirements Coverage

**Table 70**. _Functional Requirements Coverage by group_

| **Group**                         | **FRs in the group** | **Traced FRs** | **Stories covering them**                                       |
| --------------------------------- | ---------------- | --------------- | ------------------------------------------------------------------ |
| RF-01 User Management         | 4                | 4               | HU-USER-004, HU-USER-003                                           |
| RF-02 Authentication               | 2                | 2               | HU-IAM-001                                                         |
| RF-03 Routes, fleet and assignments | 8                | 8               | HU-ROUTE-001, HU-ROUTE-005, HU-BUS-002, HU-ROUTE-008, HU-FLEET-001 |
| RF-04 QR scanning                  | 3                | 3               | HU-QR-004, HU-QR-001, HU-QR-002                                    |
| RF-05 Boarding Notifications   | 4                | 4               | HU-QR-001, HU-NOTIF-001, HU-NOTIF-005                              |
| RF-06 Real-time tracking      | 3                | 3               | HU-GPS-002                                                         |
| RF-07 Trip Management           | 1                | 1               | HU-ROUTE-003                                                       |
| RF-08 Incidents                  | 1                | 1               | HU-INCIDENT-001                                                    |
| RF-09 Route Inquiry           | 2                | 2               | HU-ROUTE-004, HU-ROUTE-002                                         |
| RF-10 Personalization             | 2                | 2               | HU-UI-001, HU-UI-002                                               |
| **Total**                         | **30**           | **30**          | **20 user stories across 7 epics**                            |

Each of the 20 user stories is verified through acceptance criteria in Given/When/Then format and at least one associated test case; any functional requirement without a corresponding story or test case is considered a traceability gap and is managed before its closure.

# 11. References

- Congress of the Republic of Colombia. (2012). *Ley 1581 de 2012: Por la cual se dictan disposiciones generales para la protección de datos personales* [Law 1581 of 2012 on the protection of personal data]. Diario Oficial.
- European Parliament and Council of the European Union. (2016). *Regulation (EU) 2016/679 on the protection of natural persons with regard to the processing of personal data and on the free movement of such data (General Data Protection Regulation)*. Official Journal of the European Union.
- IEEE. (1998). *IEEE recommended practice for software requirements specifications* (IEEE Std 830-1998). Institute of Electrical and Electronics Engineers.
- International Organization for Standardization. (2011). *Systems and software engineering — Systems and software quality requirements and evaluation (SQuaRE) — System and software quality models* (ISO/IEC 25010:2011). ISO.
- International Organization for Standardization. (2017). *Systems and software engineering — Software life cycle processes* (ISO/IEC/IEEE 12207:2017). ISO.
- International Organization for Standardization. (2022). *Software and systems engineering — Software testing* (ISO/IEC/IEEE 29119:2022). ISO.
- OWASP Foundation. (2021). *OWASP Top 10: 2021 — The ten most critical web application security risks*. https://owasp.org/Top10/
- Presidency of the Republic of Colombia. (2013). *Decreto 1377 de 2013, por el cual se reglamenta la Ley 1581 de 2012* [Decree 1377 of 2013]. Diario Oficial.
