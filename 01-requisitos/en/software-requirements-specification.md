# Software Requirements Specification — Guardian Escolar

---

## Table of Contents

- [1. Introduction](#1-introduction)
  - [1.1 Purpose](#11-purpose)
  - [1.2 Product Scope](#12-product-scope)
  - [1.3 Definitions and Acronyms](#13-definitions-and-acronyms)
  - [1.4 Normative References](#14-normative-references)
- [2. General Description](#2-general-description)
  - [2.1 Product Perspective and Characteristics](#21-product-perspective-and-characteristics)
  - [2.2 User Classes](#22-user-classes)
  - [2.3 Constraints](#23-constraints)
  - [2.4 Assumptions and Dependencies](#24-assumptions-and-dependencies)
- [3. Functional Requirements](#3-functional-requirements)
- [4. Non-Functional Requirements](#4-non-functional-requirements)
- [5. Interface Requirements](#5-interface-requirements)
- [6. Data Requirements](#6-data-requirements)
- [7. User Stories](#7-user-stories)
- [8. General Acceptance Criteria and Verification Strategy](#8-general-acceptance-criteria-and-verification-strategy)
- [9. Glossary](#9-glossary)
- [10. Appendix: HU↔RF Traceability Matrix](#10-appendix-hurfr-traceability-matrix)

---

## 1. Introduction

### 1.1 Purpose

This document specifies the software requirements for **Guardian Escolar**, a web and mobile platform for school transportation monitoring and safety. The specification establishes, in a verifiable manner, what the system must do, with which qualities it must operate, how it communicates with its components and users, and what data it manages.

This specification follows the structure recommended by **IEEE 830** for software requirements specifications (SRS) and adopts the usage characteristics model from **ISO/IEC 25010** as its product quality reference model. Its audience includes the design, construction, and verification team, the project instructor, and the educational institution receiving the solution.

Each requirement in this document is **necessary and sufficient** to guide subsequent implementation and verification: functional requirements describe observable functions and their business rules; non-functional requirements establish measurable metrics; and the traceability appendix links each user story to the requirements it implements.

### 1.2 Product Scope

**Guardian Escolar** is a centralized platform that integrates GPS tracking, boarding control via QR code scanning, real-time notifications and alerts, a web administrative panel, and mobile applications for drivers and guardians. The software comprises:

- A **web application** built with Angular 21 and Angular Material 3, aimed at administrative and platform operation roles.
- A **mobile application** built with React and Expo, aimed at drivers, guardians, and students.
- **Nine domain microservices**: identification and access (IAM), user management, school management, fleet, routes, notifications, exceptional uses, audit, and configuration.
- An **API Gateway (Kong OSS)** as the single entry point, handling TLS termination, rate limiting, CORS, and routing versioned /api/v1 paths.
- An **event bus** with Apache Kafka in KRaft mode and a **persistent low-latency gRPC channel** for geolocation telemetry.
- A **single relational database** (SQL Server 2022) with per-service schema and versioned migrations via Liquibase.

The following elements are deliberately excluded from this specification's scope:

| Element out of scope | Reason |
| --- | --- |
| Service billing or payment for school transport | Outside the safety and monitoring purpose |
| Fleet mechanical maintenance, fuel, or repair shop | Not a confirmed user flow |
| Academic management (grades, attendance, learning) | The platform manages transport, not academics |
| Public transit and non-school routes | Domain limited to school transport |
| Physical purchase, administration, or installation of GPS devices | External responsibility |
| Autonomous emergency response or automatic dispatch | Human and institutional response is external |
| Biometric identification or facial recognition | Not required for current scope and raises privacy obligations |
| Multinational regulatory geolocation | Initial context is Neiva (Huila), Colombia |
| Autonomous driver navigation and AI-optimal route generation | Outside current product scope |

The system does **not allow self-registration**: all user accounts are created by the institution administrator. There is no desktop application; access is via web browser and mobile application only.

### 1.3 Definitions and Acronyms

| Term | Definition |
| --- | --- |
| SRS | _Software Requirements Specification_ |
| FR | Functional Requirement: function or behavior the system must exhibit |
| NFR | Non-Functional Requirement: measurable system quality or constraint |
| US | User Story: brief description of a need from the role's perspective |
| Epic | Grouping of related user stories by shared business value |
| API | Application Programming Interface |
| REST | Architectural style for resource exchange via HTTP |
| gRPC | High-efficiency remote procedure call protocol |
| JWT | _JSON Web Token_: signed token used for authentication and authorization |
| RBAC | Role-Based Access Control |
| QR | Quick Response two-dimensional barcode |
| Boarding Event | Student pickup or drop-off event registered on the bus |
| GPS | Global Positioning System |
| Parent/Guardian | Responsible adult authorized to view their student's transport information |
| SLO | Service Level Objective |
| P95 / P99 | 95th or 99th percentile of a measurement distribution |
| RTO / RPO | Recovery Time Objective / Recovery Point Objective |
| PII | Personally Identifiable Information |
| Error Rate | Percentage of requests ending in server error responses |

### 1.4 Normative References

The following standards and regulatory frameworks apply as mandatory or reference:

| Reference | Subject |
| --- | --- |
| IEEE 830-1998 | Guide for writing software requirements specifications |
| ISO/IEC 25010:2011 | Software product quality characteristics model |
| ISO/IEC/IEEE 12207 | Software life cycle processes |
| ISO/IEC 29119 | Software testing standards |
| OWASP Top 10 | Critical security risks for web applications |
| Law 1581 of 2012 and Decree 1377 of 2013 (Colombia) | Personal data protection and habeas data, special treatment for minors' data |
| General Data Protection Regulation (GDPR) | Applicable if the product is offered in the European Union |

---

## 2. General Description

### 2.1 Product Perspective and Characteristics

Guardian Escolar addresses the lack of reliable, timely visibility into school bus routes: parents don't know if students boarded or alighted, drivers record information manually, and there is no traceability or timely alerts in incidents. The solution replaces those manual processes with a single platform connecting families, students, drivers, administrators, and platform operators around school transport operations.

The product relies on the following structural decisions:

- **Event-oriented microservices architecture** behind an API Gateway (Kong OSS) on port **8000**; business services handle requests on port **8080**.
- **Single relational database** (SQL Server 2022, port **1433**) with per-service schema and Liquibase migrations; no service accesses another service's schema directly.
- **Apache Kafka 4.1.0** in KRaft mode (port **9092**) for high-level domain events and **gRPC** on port **5001** for low-latency geolocation telemetry streaming.
- **JWT RS256** authentication: 15-minute access token and 30-day refresh token; only the IAM service holds the private key.
- Clients: **Angular 21 + Angular Material 3** web app and **React + Expo** mobile app (Expo push notifications and camera for QR scanning).

```
   Parent / Student / Driver              Admin / Operator
    (React + Expo mobile app)                (Angular web app)
                  │                                        │
                  └───────────────┬────────────────────────┘
                                  │ HTTPS · /api/v1
                                  ▼
                    ┌─────────────────────────────┐
                    │  API Gateway (Kong) :8000   │
                    │  TLS · CORS · Rate Limiting │
                    │  JWT RS256 · Routing        │
                    └──────────────┬──────────────┘
                                   │ REST/JSON :8080
       ┌──────────┬──────────┬──────┼──────┬──────────┬──────────┐
       ▼          ▼          ▼      ▼      ▼          ▼          ▼
     IAM     User Mgmt   School   Fleet   Routes  Notifications  …
                              Mgmt
       └──────────┴──────────┴──────┼──────┴──────────┴──────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              ▼                       ▼                       ▼
      SQL Server :1433          Kafka :9092              gRPC :5001
     (per-service schema)  (domain events)         (GPS positions)
```

**Figure 1.** _General architecture view of the platform_

The table below summarizes the product's quality characteristics and their expected manifestation, according to ISO/IEC 25010:

| Quality Characteristic | Manifestation in the Product |
| --- | --- |
| Functional Suitability | Coverage of 30 functional requirements in Section 3 |
| Performance Efficiency | Latency and delivery times set in Section 4.1 |
| Compatibility | Integration via REST, gRPC, events, and mobile push notifications |
| Usability | Role-based interfaces, side navigation, validated forms, immediate feedback |
| Reliability | 99.9% SLO availability, failure recovery, and GPS signal loss alerts |
| Security | JWT RS256 authentication, role-based authorization, audit trail, personal data protection |
| Maintainability | Test coverage, controlled complexity, versioned migrations |
| Portability | Containerized deployment independent of host operating system |

The product pursues three objectives: (1) guarantee real-time traceability of every school trip; (2) notify guardians of their students' boarding and alighting status; (3) provide the institution with a management panel for fleet, routes, and role-based users.

### 2.2 User Classes

The system defines five roles, coded as enumeration values. Each role authenticates and accesses only the functions and resources authorized by its role and permissions:

| Role | Title | Profile and Main Functions | Interface |
| --- | --- | --- | --- |
| SUPER_ADMIN | Platform Operator | Manages institutions, global configuration, and platform audit. | Web |
| ADMIN | School Administrator | Manages branches, courses, people, fleet, routes, and reports for their institution. | Web |
| DRIVER | Driver | Executes routes, confirms boardings via QR scanning, and receives alerts on mobile app. | Mobile |
| PARENT | Guardian | Follows the route, receives boarding/drop-off notifications, and views boarding history. | Mobile |
| STUDENT | Student | Views their QR identification linked to profile for boarding. | Mobile |

External actors interacting with the system (not authenticated users) include the onboard GPS device emitting positions and the push notification provider. The educational institution, families, and emergency services remain outside the system as parties acting upon the information it provides.

### 2.3 Constraints

| Type | Constraint |
| --- | --- |
| Academic | Project developed under SENA's Software Analysis and Development program, study code 3145556, Neiva (Huila) training center. |
| Geographical | Initial operational context is Neiva (Huila), Colombia. |
| Technological | Web frontend in Angular 21 with Angular Material 3; mobile app in React with Expo; services in Java 21 with Spring Boot (identification and access) and C# with .NET 10 (remaining domain services). |
| Architectural | Microservices style with event orientation behind an API Gateway; single relational database with per-service schema; mandatory Liquibase migrations on every startup. |
| Communications | Versioned synchronous interface /api/v1 over REST/JSON; gRPC on port 5001 for telemetry; events on Apache Kafka in KRaft mode. |
| Security | JWT RS256 authentication with 15-minute access token and 30-day refresh token; private key exclusive to IAM service; least privilege by role; secrets only in environment variables. |
| Regulatory | Compliance with Law 1581 of 2012 and Decree 1377 of 2013, special treatment and consent for minors' data; GDPR if product offered in EU. |
| Operational | Docker containerized deployment orchestrated via Docker Compose; development, test, and production environments. |
| Language | Interface supporting at least four languages (Spanish, English, French, Portuguese) with Spanish as primary language in v1. |
| Process | Specification, design, construction, testing, and deployment conducted with agile methodology; any scope change documented and approved before construction. |

### 2.4 Assumptions and Dependencies

| # | Assumption or Dependency | Type | Consequence if Not Met |
| --- | --- | --- | --- |
| 1 | Institutions and families provide correct data for students, guardians, drivers, vehicles, routes, and stops. | Assumption | Route assignments and security notifications would be incorrect. |
| 2 | Users have a compatible web browser or a mobile device with a camera. | Assumption | An assisted or offline channel would be required. |
| 3 | SQL Server is available for project persistence. | Assumption | Persistence architecture and deployment plan would change. |
| 4 | The programming interface exposes the data required by web and mobile clients. | Assumption | Contracts, flows, and frontend delivery dates would change. |
| 5 | A positioning provider exposes vehicle location and status when monitoring is active. | Assumption | Real-time location functions would remain unavailable. |
| 6 | Notification sending is available via the configured provider or service. | Assumption | Notifications would be limited to the app or simulated data. |
| 7 | Roles and permissions are consistently defined between frontends and services. | Assumption | Unauthorized access and inconsistent dashboards could occur. |
| 8 | Initial deployment targets Neiva (Huila), Colombia. | Assumption | Data, language, regulation, and operation assumptions should be reviewed. |
| 9 | SQL Server environment available before integration tests. | Dependency | Integrated verification would not be possible. |
| 10 | GPS provider definition and contract before live monitoring. | Dependency | Real-time location display would remain inactive. |
| 11 | Push notification service defined before production. | Dependency | Notifications would not reach devices. |
| 12 | Valid school and transport data before acceptance tests. | Dependency | Real business flows could not be validated. |
| 13 | Role and authorization contract aligned between services and frontend teams. | Dependency | Publication with access control would be blocked. |

---

## 3. Functional Requirements

This section specifies **30 functional requirements** organized into **10 functional groups**. Each requirement indicates its identifier, title, detailed description, inputs and outputs, preconditions and postconditions, and the business rules governing it. Priority uses the scale High, Medium, and Low based on business value and operational dependency.

| Group | Title | Functional Owner | FR | Dominant Priority |
| --- | --- | --- | --- | --- |
| RF-01 | User Management | User management service and IAM | 4 | High |
| RF-02 | Authentication | Identification and Access Management (IAM) | 2 | High |
| RF-03 | School Route, Fleet & Assignments Management | Routes service and Fleet service | 8 | High |
| RF-04 | QR Scanning Management: Boarding Control | IAM service and Notifications service | 3 | High |
| RF-05 | Boarding Notifications | Notifications service | 4 | High |
| RF-06 | Real-Time Bus Tracking | Geolocation module and gRPC channel | 3 | Medium |
| RF-07 | Trip Management | Routes service | 1 | Medium |
| RF-08 | Incidents | Notifications service | 1 | High |
| RF-09 | Route Inquiry | Routes service | 2 | Medium |
| RF-10 | Personalization | Web and mobile client apps | 2 | Low |

### 3.1 Group RF-01 — User Management

This group covers the lifecycle of ecosystem accounts: creation, update, activation/deactivation, and linking students to their guardian. The account is split across two responsibilities: person data belongs to the user management service; the authentication profile, with its role and password hash, belongs to IAM.

#### 3.1.1 FR-01.1 — Account Creation with Assigned Role

The administrator creates accounts for students, drivers, guardians, and administrators, assigning the role that will define their permissions in the same act. The initial password is generated by the system and delivered to the user through a secure channel; the role is stored with the profile and enables corresponding functions from the first login.

| Field | Detail |
| --- | --- |
| Inputs | Document type and number, full name, email, phone, address, date of birth, and role to assign (ADMIN, DRIVER, PARENT, STUDENT); student academic data and student–guardian link where applicable. |
| Outputs | Created and active account with assigned role; error message if email or document already exist; audit record of creation. |
| Preconditions | Applicant authenticated with ADMIN or SUPER_ADMIN role and possessing user management permission. |
| Postconditions | Account exists, is active, and can log in with initial password; the user creation event is published for system dissemination. |

**Business Rules**

- No self-registration: all accounts are created by the institution administrator; the administrator account is created directly by the deployment team.
- Email and document number are unique in the system; if they already exist, the system informs that the account is registered and does not duplicate the record.
- A student is linked to only one active guardian account at a time.
- Any missing required field prevents creation and the system preserves entered data for correction.

#### 3.1.2 FR-01.2 — Account Data Update

The administrator updates account personal data (name, document, email, phone, address, date of birth) and student academic data, without altering boarding history or established relationships.

| Field | Detail |
| --- | --- |
| Inputs | Account identifier and set of fields to modify. |
| Outputs | Account updated with last modification date; error message on uniqueness conflict. |
| Preconditions | Existing account and authenticated applicant with user management permission. |
| Postconditions | Current data reflects the last modification; the update event is published. |

**Business Rules**

- Creation date is immutable; last modification date updates automatically.
- If the modified email or modulated document already belong to another account, the operation is rejected without altering the record.
- Changes to persons are recorded in the critical actions audit.

#### 3.1.3 FR-01.3 — Account Activation and Deactivation

The administrator activates or deactivates accounts. Deactivation prevents login and blocks any future assignment of the user to routes, buses, or trips, while preserving history.

| Field | Detail |
| --- | --- |
| Inputs | Account identifier and target state (ACTIVE / INACTIVE). |
| Outputs | Account state updated; login rejection if account is deactivated. |
| Preconditions | Existing account; authenticated applicant with corresponding permission. |
| Postconditions | An INACTIVE account cannot authenticate nor be assigned; its history remains queryable. |

**Business Rules**

- Entity lifecycle is governed by ACTIVE and INACTIVE states, i.e., logical disablement, not physical deletion.
- Deactivation blocks access without removing user history.
- An INACTIVE profile cannot start a new authenticated session.

#### 3.1.4 FR-01.4 — Linking Students to Guardian Account

The administrator links one or more students to a guardian account so they can follow all their children from a single account.

| Field | Detail |
| --- | --- |
| Inputs | Guardian account identifier and identifiers of students to link. |
| Outputs | Guardian–student relationship recorded; error if student is already associated. |
| Preconditions | Existing and active guardian and student accounts; applicant with user management permission. |
| Postconditions | Guardian sees the linked student's information and receives their notifications. |

**Business Rules**

- A student can only be linked to one active guardian account at a time.
- The guardian account must exist before receiving links.
- Linking enables viewing the student's route and receiving their notifications.

### 3.2 Group RF-02 — Authentication

This group specifies system access. The IAM service issues RS256-signed JWT tokens and applies role-based access control; the access token expires after 15 minutes and the refresh token after 30 days.

#### 3.2.1 FR-02.1 — Credential Authentication and Token Issuance

The system authenticates the user with credentials per the current password policy, and if valid and the account is active, issues an RS256-signed access token limiting the session to functions permitted by their role.

| Field | Detail |
| --- | --- |
| Inputs | Email and password per policy (length, uppercase, numbers, symbols, and day expiration). |
| Outputs | RS256 JWT access token with 15-minute validity and role claim; "invalid credentials" error without token issuance. |
| Preconditions | Account created by administrator and in ACTIVE state. |
| Postconditions | Session started and registered with IP address, dates, and application; profile creation and session events published. |

**Business Rules**

- Invalid credentials or deactivated account produce rejection and do not issue a token.
- Signing algorithm is RS256 and only IAM holds the private key.
- Role-based access control applies by default: without explicit role and permission, no access is granted.

#### 3.2.2 FR-02.2 — Logout and Token Renewal

The user logs out, and while holding a valid refresh token, renews the access token without re-entering credentials.

---
*Translation continues in next sections... → [Section 4 onwards](#4-non-functional-requirements)*

### 3.3 Group RF-03 — School Route, Fleet & Assignments Management

This group covers operational transport configuration: route definition with stops and schedules, fleet bus administration, and establishment of bus, driver, and student assignments. The routes service manages route configuration and assignments; the fleet service manages buses and driver accountability per bus.

#### 3.3.1 FR-03.1 — Route Creation with Data, Ordered Stops & Schedules

The administrator creates a school route entering its data, its stops in itinerary order, and its schedules, so that drivers, students, and families have a defined and consistent itinerary.

| Field | Detail |
| --- | --- |
| Inputs | Route name and sector, home branch, stops with address and coordinates, stop order, day of week, trip direction, and scheduled time. |
| Outputs | Route registered with ordered stops and schedules; route visible in listing. |
| Preconditions | Institution branch and master data available; applicant authenticated as administrator. |
| Postconditions | Route exists in usable state and its stops preserve itinerary order. |

**Business Rules**

- A route must belong to a branch of an institution.
- Stops are ordered by itinerary, never by creation order, and cannot repeat at the same position.
- A route cannot be activated without complete required configuration.

#### 3.3.2 FR-03.2 — Route Uniqueness Validation

Before persisting a route, the system validates no other route exists with the same name in the corresponding scope.

| Field | Detail |
| --- | --- |
| Inputs | Proposed route name. |
| Outputs | Route persisted, or "route already exists" message without duplicate record. |
| Preconditions | Route creation request received. |
| Postconditions | No two routes share the same name. |

**Business Rules**

- Uniqueness is evaluated by route name before any write.
- Duplicate attempt is reported to the user and preserves entered data.

#### 3.3.3 FR-03.3 — Route Data Update

The administrator modifies existing route data, its stops, and schedules, maintaining consistency of current assignments.

| Field | Detail |
| --- | --- |
| Inputs | Route identifier and fields to modify. |
| Outputs | Route updated; rejection if modification violates uniqueness or integrity. |
| Preconditions | Existing route and applicant with permission. |
| Postconditions | Changes take effect and appear in driver and guardian queries. |

**Business Rules**

- Update cannot introduce duplicate stops at the same position or break itinerary order.
- All route configuration changes are recorded in audit.

#### 3.3.4 FR-03.4 — Route Data Deletion

The administrator deletes a route when it is no longer operating, unless there is an active trip depending on it.

| Field | Detail |
| --- | --- |
| Inputs | Identifier of the route to delete. |
| Outputs | Route deleted, or rejection message if an active trip depends on it. |
| Preconditions | Existing route; applicant with permission. |
| Postconditions | Route removed from operation without orphan references. |

**Business Rules**

- Cannot delete a route with an active trip.
- Associated assignments are removed with the route to preserve integrity.

#### 3.3.5 FR-03.5 — Bus Assignment and Removal from Route

The administrator assigns a bus to a route and can remove it, so each route operates with an available vehicle.

| Field | Detail |
| --- | --- |
| Inputs | Route and bus identifiers; assignment or removal action. |
| Outputs | Route–bus relationship registered or removed; bus status updated to "on route" when applicable. |
| Preconditions | Active route and active bus without incompatible ongoing assignment. |
| Postconditions | Route has at most one assigned bus and the bus serves at most one route. |

**Business Rules**

- A bus is assigned to only one route at a time, and a route receives only one bus at a time.
- Buses in INACTIVE state are not assigned.
- Driver route coverage derives from the bus assigned to the route.

#### 3.3.6 FR-03.6 — Bus Creation, Edition, Activation & Deactivation

The administrator creates, edits, activates, or deactivates fleet buses, keeping the vehicle's physical identity up to date for route assignment.

| Field | Detail |
| --- | --- |
| Inputs | License plate, capacity, model and brand, SOAT (insurance) data, associated GPS device, and initial status. |
| Outputs | Bus registered with available status; error message if plate already exists. |
| Preconditions | Applicant authenticated with fleet management permission. |
| Postconditions | Bus appears in listing and is eligible for assignment when active. |

**Business Rules**

- Plate is unique in the system and capacity must be greater than zero.
- A bus belongs to one institution and a GPS device cannot be assigned to multiple buses simultaneously.
- An INACTIVE bus cannot be assigned to a route.

#### 3.3.7 FR-03.7 — Driver Assignment and Removal from Bus

The administrator assigns a driver as responsible for a bus and can reassign or remove them; routes covered by that bus are covered by that driver.

| Field | Detail |
| --- | --- |
| Inputs | Bus and driver identifiers; assignment or removal action. |
| Outputs | Driver–bus relationship registered with validity period; assignment event published. |
| Preconditions | Driver with active account and existing bus; applicant with permission. |
| Postconditions | The responsible driver can consult route coverage in the mobile app for their bus. |

**Business Rules**

- A driver is responsible for only one bus at a time until reassigned or removed.
- No INACTIVE drivers are eligible for assignment.
- Drivers reach their routes transversely through the bus assigned to each route; no direct driver–route assignment exists.

#### 3.3.8 FR-03.8 — Student Assignment to Route with Pickup & Drop-off Stops

The administrator assigns students to a route indicating the pickup stop and drop-off stop, so the system knows who waits, at which stop, and at what time.

| Field | Detail |
| --- | --- |
| Inputs | Route identifier, student identifier, and pickup/drop-off stops. |
| Outputs | Route–student relationship registered; assignment event published. |
| Preconditions | Active route, student with active account, and selected stops belonging to the route. |
| Postconditions | Guardian can query their student's route and QR boarding flow is enabled. |

**Business Rules**

- Selected stops must belong to the route; otherwise rejected with "stop does not belong to this route."
- A student maintains only one active route at a time.
- Assigned student must belong to the school context of the route.

### 3.4 Group RF-04 — QR Scanning Management: Boarding Control

This group specifies student identification via QR code and recording of boarding events. The individual QR code is generated by IAM; pickup and drop-off registration is processed by the notifications service, which validates operational conditions of the trip.

#### 3.4.1 FR-04.1 — Generation of Individual Student QR Code

The system generates a unique QR code for each enrolled student, containing the encrypted data needed for verification at boarding and alighting points.

| Field | Detail |
| --- | --- |
| Inputs | Student identifier and account activation status. |
| Outputs | Unique QR code string/link generated and stored; visible to guardian and student apps. |
| Preconditions | Student account created, active, and linked to guardian. |
| Postconditions | QR code available for scanning; boarding events can reference this code. |

**Business Rules**

- Each student has exactly one QR code active at a time.
- The QR code is regenerated if compromised or lost (with invalidation of previous).
- The code contains enough information for validation but prevents impersonation.

#### 3.4.2 FR-04.2 — QR Code Reading to Register Pickup

The driver scans the student's QR code at the pickup stop. The system validates that the student belongs to the active trip on the route, that the timing is within expected window, and records the boarding event.

| Field | Detail |
| --- | --- |
| Inputs | Scanned QR code data, driver identity, bus location, and timestamp. |
| Outputs | Boarding event registered; push notification sent to guardian ("Student picked up"). |
| Preconditions | Student assigned to active route; driver logged in; GPS signal active. |
| Postconditions | Boarding event persists; guardián receives notification; trip progress advances. |

**Business Rules**

- Scanner accepts codes only from students currently assigned to the active trip.
- Duplicate scan within short time window is prevented (debounce period).
- If GPS is unavailable during scan, the event is queued and synced when connectivity resumes.
- Invalid or unassigned QR codes produce a clear error shown to the driver.

#### 3.4.3 FR-04.3 — QR Code Reading to Register Drop-off

Same flow as pickup but at the destination stop. System confirms the student boarded the route and was previously marked as picked up before allowing drop-off.

| Field | Detail |
| --- | --- |
| Inputs | Scanned QR code data, driver identity, bus location, and timestamp. |
| Outputs | Drop-off event registered; push notification sent to guardian ("Student dropped off"). |
| Preconditions | Student assigned to active route; prior pickup event exists; GPS active. |
| Postconditions | Full journey recorded end-to-end; guardian notified of successful drop-off. |

**Business Rules**

- A drop-off cannot occur without a prior pickup event.
- Scan only valid at designated drop-off stop for that student.
- Missed student (no scan by end of route) triggers an alert to guardian and administrator.

### 3.5 Group RF-05 — Boarding Notifications

This group specifies automatic notifications sent to guardians regarding boarding and alighting events, plus alerts for prolonged stops.

#### 3.5.1 FR-05.1 — Student Pickup Event Detection

The system detects the pickup event triggered by QR scanning or proximity-based methods when the student boards the assigned bus.

| Field | Detail |
| --- | --- |
| Inputs | Validated boarding event from scanner or sensor. |
| Outputs | Pickup event confirmed and routed to notification engine. |
| Preconditions | Student actively boarding; route execution in progress. |
| Postconditions | Notification queued for delivery to guardian(s). |

**Business Rules**

- Event detection requires valid QR scan or confirmed GPS proximity match.
- Only one pickup event per student per trip.

#### 3.5.2 FR-05.2 — Automatic Pickup Notification to Guardian

Upon confirming pickup, the system sends an immediate push notification to all linked guardians.

| Field | Detail |
| --- | --- |
| Inputs | Confirmed pickup event with student name, time, bus ID, and route info. |
| Outputs | Push notification delivered via Expo service. |
| Preconditions | Pickup event validated; guardian notification preferences configured. |
| Postconditions | Guardian receives real-time alert; delivery status tracked. |

**Business Rules**

- Notification includes: student name, pickup time, route number, bus plate.
- If delivery fails, the event retries up to 3 times with exponential backoff.
- Notification preferences per guardian control channel (push/email/SMS).

#### 3.5.3 FR-05.3 — Automatic Drop-off Notification to Guardian

Upon confirming drop-off, the system sends an immediate push notification to all linked guardians.

| Field | Detail |
| --- | --- |
| Inputs | Confirmed drop-off event with student name, time, bus ID, and route info. |
| Outputs | Push notification delivered via Expo service. |
| Preconditions | Drop-off event validated; guardian notification preferences configured. |
| Postconditions | Guardian receives real-time alert; delivery status tracked. |

**Business Rules**

- Same format rules as pickup notification.
- Trip completion summary generated after all students dropped off.

#### 3.5.4 FR-05.4 — Alert for Prolonged Bus Stop

If the bus remains stationary beyond the configured threshold (default: 5 minutes) at any point along the route, the system sends an alert to guardians and administrators.

| Field | Detail |
| --- | --- |
| Inputs | GPS positions showing extended stationary period exceeding threshold. |
| Outputs | Delay/prolonged stop alert pushed to relevant guardians and admin dashboard. |
| Preconditions | Active route execution; GPS signal stable and reporting correctly. |
| Postconditions | Alert delivered; administrator informed of possible delay. |

**Business Rules**

- Threshold configurable per institution (default 5 min).
- Traffic-related delays excluded via external traffic API integration.
- Repeated prolonged stops trigger escalation to operations team.

### 3.6 Group RF-06 — Real-Time Bus Tracking

This group covers continuous GPS capture and interactive map visualization of bus location during active trips.

#### 3.6.1 FR-06.1 — Continuous GPS Location Capture

The system continuously captures the bus's GPS position via gRPC stream during active route execution.

| Field | Detail |
| --- | --- |
| Inputs | GPS position data streamed from device on the bus (lat/lng, speed, heading). |
| Outputs | Position persisted and forwarded to real-time map subscribers. |
| Preconditions | Route execution started; GPS device transmitting; gRPC channel open. |
| Postconditions | Real-time position available for display; position history logged. |

**Business Rules**

- Minimum update interval: 10 seconds (configurable).
- Missing positions (> 2 min gap) trigger "signal loss" state.
- Positions filtered for accuracy (GPS noise reduction).

#### 3.6.2 FR-06.2 — Interactive Map Location Visualization

Guardians and administrators view the bus position in real-time on an interactive map within their applications.

| Field | Detail |
| --- | --- |
| Inputs | Real-time GPS positions for active trips. |
| Outputs | Live map rendering with bus marker, route path, and nearby stops. |
| Preconditions | Active trip with GPS tracking; user authorized to view the trip. |
| Postconditions | User sees moving marker synchronized with actual bus position. |

**Business Rules**

- Only users with child/student assignment to the route see the marker.
- Map shows current position, upcoming stops, and historical trace.
- Auto-refreshes every 10 seconds; manual refresh button available.

#### 3.6.3 FR-06.3 — GPS Signal Loss Alert

If the bus loses GPS signal for more than 2 consecutive minutes, the system alerts the administrator and all affected guardians.

| Field | Detail |
| --- | --- |
| Inputs | Gap in GPS position data exceeding timeout threshold. |
| Outputs | Signal loss alert sent to admin and guardians of affected students. |
| Preconditions | Active route execution; expected GPS signal absent. |
| Postconditions | Alert logged and delivered; auto-clears when signal restored. |

**Business Rules**

- Signal restoration automatically clears the alert.
- Alert persists only while signal remains missing during active trip.
- Repeated signal losses flagged for fleet maintenance review.

### 3.7 Group RF-07 — Trip Management

#### 3.7.1 FR-07.1 — Trip Lifecycle: Start and Completion

The trip lifecycle covers initiation of a route execution through completion, including intermediate states and checkpoints.

| Field | Detail |
| --- | --- |
| Inputs | Trip start command from driver app; completion confirmation at final stop. |
| Outputs | Trip state transitions recorded; completion summary generated. |
| Preconditions | Route assigned, bus and driver ready, all students boarded. |
| Postconditions | Trip archived with full event log; guardian notifications completed. |

**Business Rules**

- Trip starts only when driver confirms readiness via app.
- Intermediate checkpoints: departure from depot, first pickup, last pickup, midpoint, last drop-off.
- Trip completes only when final student is dropped off and driver confirms.
- Incomplete trips require reason entry for follow-up.

### 3.8 Group RF-08 — Incidents

#### 3.8.1 FR-08.1 — Incident Report and Automatic Notification

Any participant can report an incident during trip execution. The system notifies relevant parties immediately based on incident severity.

| Field | Detail |
| --- | --- |
| Inputs | Incident type, description, severity level, optional photo/evidence. |
| Outputs | Incident record created; automatic notifications dispatched. |
| Preconditions | Active user with authentication and role-appropriate permissions. |
| Postconditions | Incident visible to admins; guardians notified based on severity; resolution workflow initiated. |

**Business Rules**

- Incident types: mechanical failure, accident, student injury, medical emergency, security issue, weather-related.
- Severity levels: Low, Medium, High, Critical — determining notification scope.
- Critical incidents notify all stakeholders immediately and trigger emergency protocols.
- Resolution follows predefined escalation tree.

### 3.9 Group RF-09 — Route Inquiry

#### 3.9.1 FR-09.1 — Guardian Route Inquiry for Their Student

Guardians view the assigned route details including stops, schedules, bus info, and driver name for their student(s).

| Field | Detail |
| --- | --- |
| Inputs | Guardian authentication and linked student identifiers. |
| Outputs | Route details page: stops list, pickup/drop-off times, bus plate, driver name. |
| Preconditions | Guardian account active; student linked to guardian. |
| Postconditions | Guardian sees real-time status and planned schedule. |

**Business Rules**

- Shows route for ALL linked students consolidated into single view.
- Updates automatically when route changes are made by admin.
- Historical routes available for past dates (reporting period).

#### 3.9.2 FR-09.2 — Driver Assigned Route Inquiry

Drivers view their assigned route, stop sequence, pickup/drop-off schedule, and list of assigned students.

| Field | Detail |
| --- | --- |
| Inputs | Driver authentication and current date/route execution context. |
| Outputs | Route details with student list, stop order, schedule, and contact info for guardians. |
| Preconditions | Driver assigned to active route for selected date. |
| Postconditions | Driver sees operational plan for the day. |

**Business Rules**

- Route info locked once trip execution starts (read-only mode).
- Guardian contact info accessible for emergencies only.
- Schedule adjustments reflected in real-time if admin updates mid-trip.

### 3.10 Group RF-10 — Personalization

#### 3.10.1 FR-10.1 — System Color Theme Customization

Users can customize the color theme of the interface according to institutional branding or personal preference.

| Field | Detail |
| --- | --- |
| Inputs | Color palette selection (primary, secondary, accent). |
| Outputs | Interface rendered with chosen colors across web and mobile clients. |
| Preconditions | User authenticated; institution may provide default theme. |
| Postconditions | Theme applied persistently per user/account. |

**Business Rules**

- Institutional themes set by admin apply globally unless overridden at user level.
- WCAG contrast requirements enforced — no theme violates minimum contrast ratios.
- Available palettes include: institutional (branded), neutral dark, neutral light.

#### 3.10.2 FR-10.2 — Multi-language Interface Configuration

The interface supports switching between at least four languages (Spanish, English, French, Portuguese), with Spanish as primary in v1.

| Field | Detail |
| --- | --- |
| Inputs | Language preference selection in settings. |
| Outputs | All UI strings rendered in selected language. |
| Preconditions | User authenticated; supported language pack available. |
| Postconditions | Language preference saved and applied to all screens. |

**Business Rules**

- All system-generated messages and notifications honor language preference.
- New locale keys added with translation pipeline.
- Fallback to English if translation for selected language missing.

---

## 4. Non-Functional Requirements

### 4.1 NFR-01 Performance

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-01.1 | API Gateway response latency (p95) | ≤ 200ms under normal load |
| NFR-01.2 | API Gateway response latency (p99) | ≤ 500ms under peak load |
| NFR-01.3 | Login/authentication response time | ≤ 1 second |
| NFR-01.4 | Mobile app initial load time | ≤ 3 seconds on 4G |
| NFR-01.5 | Map position update latency | ≤ 5 seconds from GPS emission to display |
| NFR-01.6 | Query performance for route/student data | ≤ 500ms for standard queries |
| NFR-01.7 | QR scan processing time | ≤ 2 seconds end-to-end |
| NFR-01.8 | Push notification delivery time | ≤ 10 seconds after event generation |

**Test Method:** Load testing with JMeter/gatling simulating realistic concurrent user patterns (simultaneous login, route queries, map updates). Monitor p50, p95, p99 percentiles.

### 4.2 NFR-02 Availability

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-02.1 | Platform availability (SLO) | ≥ 99.9% monthly uptime |
| NFR-02.2 | Planned maintenance window | Maximum 4 hours/month, announced 7 days in advance |
| NFR-02.3 | API Gateway redundancy | Zero-downtime deployments; rolling updates |
| NFR-02.4 | Database failover | Automatic failover within 30 seconds |

**Test Method:** Chaos engineering tests, deployment rollback drills, and simulated infrastructure failures. Verify no user-facing errors exceed 0.1% of total requests during planned maintenance.

### 4.3 NFR-03 Scalability

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-03.1 | Concurrent authenticated users | ≥ 500 simultaneous sessions |
| NFR-03.2 | Simultaneous GPS streams | ≥ 100 concurrent devices pushing position data |
| NFR-03.3 | Kafka throughput | ≥ 10,000 events/second without backlog |
| NFR-03.4 | Database connections | Support connection pool scaling up to 200 |
| NFR-03.5 | Horizontal scaling | Stateless services scale horizontally without config changes |

**Test Method:** Capacity planning benchmarks. Increase load incrementally and measure degradation points. Document maximum sustainable capacity per environment tier.

### 4.4 NFR-04 Security

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-04.1 | Authentication protocol | JWT RS256, tokens rotated as defined |
| NFR-04.2 | Authorization enforcement | RBAC checked on every API request |
| NFR-04.3 | Secrets management | No secrets in source code; env vars or vault only |
| NFR-04.4 | Input validation | OWASP Top 10 mitigations implemented (XSS, SQLi, CSRF, injection) |
| NFR-04.5 | Encryption at rest | Sensitive PII encrypted using AES-256 |
| NFR-04.6 | Encryption in transit | TLS 1.3 for all external communications; internal mTLS where applicable |
| NFR-04.7 | Audit logging | All critical actions logged with actor, timestamp, IP, app, and outcome |
| NFR-04.8 | Minimize privilege | Principle of least privilege enforced across all roles and services |

**Test Method:** Penetration testing twice per year by qualified third party. Automated dependency vulnerability scanning in CI pipeline. Static analysis (SAST) + dynamic analysis (DAST) gates before release.

### 4.5 NFR-05 Observability

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-05.1 | Centralized logging | Structured JSON logs aggregated via ELK/Loki |
| NFR-05.2 | Metrics collection | Prometheus exporters per service; Grafana dashboards |
| NFR-05.3 | Distributed tracing | OpenTelemetry instrumentation across all services |
| NFR-05.4 | Health check endpoints | /health endpoint on every service reporting status and dependencies |
| NFR-05.5 | Alerting | PagerDuty/integration for critical thresholds exceeded |

**Test Method:** Verify all services expose metrics, traces, and health checks. Confirm dashboards populated and alert rules fire against synthetic thresholds.

### 4.6 NFR-06 Maintainability

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-06.1 | Test coverage (unit + integration) | ≥ 80% lines, ≥ 70% branches per service |
| NFR-06.2 | Cyclomatic complexity | ≤ 10 per method/function |
| NFR-06.3 | Migration strategy | Liquibase scripts forward-only; rollback scripts provided |
| NFR-06.4 | Service startup time | ≤ 30 seconds cold start in container |
| NFR-06.5 | Documentation freshness | Docs updated with every PR modifying behavior |

**Test Method:** Coverage reports from CI (JaCoCo for Java, Istanbul for C#). SonarQube quality gate blocking merge below threshold. Manual audit of migration rollback scripts per sprint.

### 4.7 NFR-07 Portability

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-07.1 | Deployment target | Docker containers running on Linux host (Debian 12+) |
| NFR-07.2 | Orchestration | Docker Compose for local/dev; Kubernetes for production (future) |
| NFR-07.3 | Dependency pinning | Every image tagged with exact version hash |
| NFR-07.4 | Build reproducibility | `docker build` produces identical images on any compliant host |

**Test Method:** Build and deploy on clean machines without cached layers. Verify functional parity across dev, staging, and prod.

### 4.8 NFR-08 Disaster Recovery

| Requirement | Metric | Target |
| --- | --- | --- |
| NFR-08.1 | RTO (Recovery Time Objective) | ≤ 4 hours for full platform restoration |
| NFR-08.2 | RPO (Recovery Point Objective) | ≤ 1 hour data loss tolerance for transactional tables |
| NFR-08.3 | Backup frequency | Daily full backups + hourly incremental for critical DBs |
| NFR-08.4 | DR drill | Tabletop exercise conducted quarterly; full restore test annually |

**Test Method:** Execute backup and restore procedure from scratch documenting actual elapsed time. Compare against RTO/RPO targets. Record discrepancies for remediation planning.

---

## 5. Interface Requirements

### 5.1 User Interface

Requirements governing layout, navigation, form validation, accessibility compliance, and responsiveness of the web and mobile interfaces. Detailed specifications maintained in **04-diseno/es/diseno-software.md**.

Key principles:

- Role-based navigation menu (sidebar for web, bottom nav bar for mobile).
- Form validation with inline feedback; disable submit when invalid.
- Accessible color contrast (WCAG AA level); screen reader compatible labels.
- Responsive layout adapting across tablet and desktop breakpoints.

### 5.2 Hardware Interface

Requirements for interaction with GPS devices mounted on buses.

- GPS device transmits over cellular network (4G/LTE preferred).
- Device emits latitude, longitude, altitude, speed, heading, and timestamps via gRPC or MQTT.
- Minimum positioning update rate: every 10 seconds while route is active.
- Battery/power supplied by vehicle electrical system with UPS backup for ≥ 2 hours.

### 5.3 Software Interface

Inter-service contracts maintained under API documentation. Key interfaces:

| Consumer | Provider | Interface Type | Protocol |
| --- | --- | --- | --- |
| Frontends (Web/Mobile) | Kong API Gateway | RESTful APIs | HTTPS / JSON |
| All domain services | IAM service | AuthN/AuthZ | Internal API |
| Routes, Fleet, Exceptional | Apache Kafka | Event streaming | TCP/Kafka protocol |
| Fleet (gps) | gRPC server | Bi-streaming telemetry | gRPC/TLS |
| Notifications | SMS/Push providers | HTTP callbacks | HTTPS |

Contract versioning: `/api/v1` prefix on all exposed paths. Breaking changes require new version (`/api/v2`) with coexistence period ≥ 90 days.

### 5.4 Communications Interface

Protocols and ports used for inter-component communication:

| Component | Port | Protocol | Purpose |
| --- | --- | --- | --- |
| Kong API Gateway | 8000 | HTTPS/TLS 1.3 | Public client facing entry |
| Domain services | 8080 | REST/JSON | Internal service-to-service |
| Kafka brokers | 9092 | Kafka binary protocol | Event bus |
| gRPC telemetry | 5001 | gRPC/TLS | Low-latency position streaming |
| SQL Server | 1433 | TDS/TLS | Data persistence |
| Redis cache | 6379 | RESP | Session caching, rate limit state |

---

## 6. Data Requirements

### 6.1 Logical Data Model (Summary)

Per-service database schemas maintain strong cohesion:

| Service | Primary Entities |
| --- | --- |
| IAM | Profile, Role, Credential, Session, RefreshToken |
| UserManagement | Person, AcademicRecord, GuardianLink |
| School | Institution, Branch, Course, Grade |
| Fleet | Bus, VehicleMaintenanceLog, GpsDevice |
| Route | Route, Stop, Assignment, DriverAssignment |
| Geolocation | PositionEvent, RouteExecution |
| Notifications | NotificationQueue, PushDelivery, DeliveryStatus |
| Exceptional | IncidentReport, RemediationAction |
| Audit | ActionLog, IpAddress, ApplicationTrace |

Full entity-relationship diagrams in **04-diseno/es/diseno-software.md → Section 5: Data Model**.

### 6.2 Integrity and Validation Rules

| Rule | Description |
| --- | --- |
| Referential integrity | Foreign keys cascade UPDATE SET NULL on dependent records deletion |
| Temporal consistency | Dates cannot be in the future for historical data; birthdates cannot indicate minors below legal age for adult roles |
| Uniqueness constraints | Email and document number unique across Person table; license plate unique in Fleet; route name unique within branch |
| Value constraints | Phone numbers 10+ digits; GPS coordinates within Colombia bounds (± 5° latitude, ± 80° longitude) |
| Soft-delete pattern | ACTIVE / INACTIVE flags on core entities instead of hard deletes; audit trail preserved |

### 6.3 Data Lifecycle and Retention

| Data Category | Retention Period | Disposition |
| --- | --- | --- |
| Session data | 30 days | Purge |
| Refresh tokens | 30 days | Invalidate at expiry |
| Position events | 1 year | Aggregate to daily summaries, purge raw data |
| Boarding events | Indefinite | Retain permanently for safety audit trail |
| Incident reports | 5 years | Retain per regulatory requirement |
| Audit logs | 5 years | Retain per compliance |
| User personal data | Until account deletion + 3 years post-deletion | Secure erase following Colombian habeas data law |

### 6.4 Personal Data Protection

Compliance with Colombian Law 1581 of 2012 and Decree 1377 of 2013 (habeas data for minors):

- Explicit parental consent required before processing minor's personal data.
- Data subject rights: access, rectification, deletion, portability exercisable through admin portal.
- Data processing purpose limitation: transport safety only — no marketing use.
- Cross-border data transfer prohibited without institutional approval and equivalent protection assessment.
- Privacy impact assessment conducted before introducing new data categories.

---

## 7. User Stories

### 7.1 Epics

| Epic | Title | Scope |
| --- | --- | --- |
| EP-001 | Boarding via QR | QR code generation, scanning, event recording |
| EP-002 | Notifications | Push notifications, alerts, delivery tracking |
| EP-003 | Route & Fleet Management | Route setup, bus/administration, assignments |
| EP-004 | User & Family Management | Account lifecycle, guardian links |
| EP-005 | Live Tracking | GPS streaming, map visualization |
| EP-006 | Incidents | Reporting, escalation, resolution |
| EP-007 | Interface & Personalization | Theming, localization, accessibility |

### 7.2 User Stories by Epic

#### EP-001 — Boarding via QR

| ID | Story | Acceptance Criteria |
| --- | --- | --- |
| US-EP001-01 | As a driver, I want to scan a student's QR code to confirm they have boarded the bus, so that I know who is on board. | Given route execution started, when driver scans valid QR, then boarding event recorded and guardian notified. Reject invalid/mismatched QR codes clearly. |
| US-EP001-02 | As a driver, I want to see a list of students assigned to today's route, so that I can verify attendance visually alongside QR scans. | List shows student name, stop number, pickup/drop-off indicator. Sorted by stop order. Refreshable manually. |

#### EP-002 — Notifications

| ID | Story | Acceptance Criteria |
| --- | --- | --- |
| US-EP002-01 | As a guardian, I want to receive a push notification when my child boards the bus, so that I know they are safe. | Notification delivered within 10s of boarding event. Contains student name, time, route info. Deep link to trip detail opens app. |
| US-EP002-02 | As a guardian, I want to receive a push notification when my child is dropped off safely, so that I know the trip is complete. | Same delivery SLA as pickup. Includes arrival time and drop-off stop name. |

#### EP-003 — Route & Fleet Management

| ID | Story | Acceptance Criteria |
| --- | --- | --- |
| US-EP003-01 | As an admin, I want to create a route with ordered stops and schedules, so that transport operations follow a defined itinerary. | Route saved with stops in specified order. Visible in routes list. Can activate/deactivate. |
| US-EP003-02 | As an admin, I want to assign a bus to a route and a driver to a bus, so that every route has a vehicle and responsible operator. | One-to-one relationships enforced. Visual confirmation upon assignment. Unassignment clears relationship. |

#### EP-004 — User & Family Management

| ID | Story | Acceptance Criteria |
| --- | --- | --- |
| US-EP004-01 | As an admin, I want to create accounts for students, drivers, and guardians with assigned roles, so that each person accesses appropriate functions. | Role assigned at creation. Initial password generated and communicated securely. Account active immediately. |
| US-EP004-02 | As an admin, I want to link multiple students to one guardian account, so that the guardian can manage and track all children from one place. | All linked students visible in guardian dashboard. Unified notification routing per linked student. |

#### EP-005 — Live Tracking

| ID | Story | Acceptance Criteria |
| --- | --- | --- |
| US-EP005-01 | As a guardian, I want to see my child's bus moving in real-time on a map, so that I can estimate arrival time. | Moving marker on map. Marker position updates every ~10s. Shows next stops ahead on route. |
| US-EP005-02 | As a guardian, I want to receive an alert if the bus stops for an unusually long time, so that I know something might be wrong. | Alert sent after 5-minute stationary threshold. Includes location and duration. Clear when signal restored or movement resumes. |

#### EP-006 — Incidents

| ID | Story | Acceptance Criteria |
| --- | --- | --- |
| US-EP006-01 | As a driver or admin, I want to report an incident during trip execution, so that relevant parties are notified immediately. | Incident form with type, severity, description, and photo attachment. Immediate push to guardians based on severity. Admin alerted. |

#### EP-007 — Interface & Personalization

| ID | Story | Acceptance Criteria |
| --- | --- | --- |
| US-EP007-01 | As a user, I want to switch the interface language between supported options, so that I can use the app in my preferred language. | Settings dialog lists available languages. Selection persists across sessions. All text translated. |
| US-EP007-02 | As a user, I want to choose a color theme for the application, so that it matches my preferences or institutional branding. | Theme selector offers predefined palettes. Preview applies instantly. Institution-administered themes take precedence. |

---

## 8. General Acceptance Criteria and Verification Strategy

### 8.1 General Acceptance Criteria

The solution is considered ready for production acceptance when all of the following are satisfied:

1. All 30 functional requirements implemented and tested with green test results.
2. All non-functional requirements verified against measurable thresholds (Section 4).
3. Zero critical or high severity defects open at GA (General Availability).
4. Acceptance test suite passes on staging environment replicating production topology.
5. Performance benchmark results documented and within targets (Section 4.1).
6. Security penetration test report reviewed and all findings remediated.
7. Operational runbooks authored and reviewed by ops team.
8. User acceptance testing signed off by product owner and institution representative.

### 8.2 Verification Strategy

| Verification Level | Scope | Tooling | Frequency |
| --- | --- | --- | --- |
| Unit Tests | Single function/method correctness | JUnit 5 (Java), xUnit (C#) | On every commit |
| Integration Tests | Service ↔ database, service ↔ Kafka | Testcontainers, embedded Kafka | On every PR |
| Contract Tests | API consumer/provider compatibility | Pact / Spring Cloud Contract | Before deploy |
| End-to-End Tests | Complete user flows across frontends and backends | Playwright (web), Detox (mobile) | Nightly + before release |
| Load Tests | Performance under realistic concurrency | JMeter, Gatling | Sprint milestone |
| Security Tests | Vulnerability scanning, penetration testing | OWASP ZAP, Burp Suite, Qualys | Quarterly + pre-release |
| UAT | Business acceptance by stakeholders | Manual test scripts executed by PO/institution reps | Before GA |

### 8.3 Quality Thresholds and Metrics

| Metric | Target | Measurement |
| --- | --- | --- |
| Code coverage | ≥ 80% lines, ≥ 70% branches | JaCoCo / Istanbul reports |
| Defect density | ≤ 0.5 defects/KLOC | Defects found / thousands lines of code |
| Mean time to detect (MTTD) | ≤ 15 min | From incident occurrence to alert firing |
| Mean time to resolve (MTTR) | ≤ 4 hours P1/P2 | From incident occurrence to resolution |
| Change failure rate | ≤ 5% | Failed deployments / total deployments |
| Lead time for changes | ≤ 5 days | From commit to production deployment |

---

## 9. Glossary

### 9.1 Domain Glossary

| Term | Definition |
| --- | --- |
| Bordada (Boarding Event) | Registration of a student getting on (pickup) or off (drop-off) the bus |
| Ruta (Route) | Defined path with ordered stops connecting pickup points to school |
| Sede (Branch) | Physical campus/facility of an educational institution |
| Conductor (Driver) | Person responsible for operating the school bus |
| Acudiente (Guardian/Parent) | Responsible adult authorized to receive transport information about their student |
| Flota (Fleet) | Collection of buses assigned to an institution |
| Trayecto (Trip/Journey) | Single execution of a route from depot to final drop-off and return |
| Parada (Stop) | Specific location where students board or alight the bus |
| Instancia de ruta (Route Instance) | Scheduled execution of a route on a specific day with specific bus and driver |

### 9.2 Technical Glossary

| Term | Definition |
| --- | --- |
| API Gateway | Kong OSS — reverse proxy handling routing, auth, rate limiting |
| Microservicio | Independently deployable service owning a bounded context |
| Bounded Context | DDD concept defining explicit boundaries of responsibility |
| gRPC | Google Remote Procedure Call — efficient RPC framework |
| Kafka | Apache Kafka — distributed event streaming platform |
| KRaft | Kafka's ZooKeeper-less controller architecture mode |
| Liquibase | Database migration tool managing schema versioning |
| JWT | JSON Web Token — standardized token format for assertions |
| RS256 | RSA Signature with SHA-256 — asymmetric signing algorithm |
| Docker Compose | Container orchestration tool for multi-container local environments |
| SLO | Service Level Objective — measurable reliability target |
| P95 / P99 | Percentile measurement capturing tail latency behavior |
| CI/CD | Continuous Integration / Continuous Deployment pipeline |
| RBAC | Role-Based Access Control |

---

## 10. Appendix: HU↔RF Traceability Matrix

### 10.1 User Story to Functional Requirements Mapping

| US ID | Epic | Linked RF(s) | Notes |
| --- | --- | --- | --- |
| US-EP001-01 | EP-001 | RF-04.1, RF-04.2, RF-05.2 | QR scan → boarding event → notification |
| US-EP001-02 | EP-001 | RF-04.3, RF-05.3 | Drop-off scan → event → guardian notification |
| US-EP002-01 | EP-002 | RF-05.1, RF-05.2, RF-05.3 | Event detection → notification delivery |
| US-EP002-02 | EP-002 | RF-05.3 | Drop-off notification flow |
| US-EP003-01 | EP-003 | RF-03.1, RF-03.2, RF-03.3, RF-03.4 | Route CRUD operations |
| US-EP003-02 | EP-003 | RF-03.5, RF-03.6, RF-03.7, RF-03.8 | Fleet and assignment operations |
| US-EP004-01 | EP-004 | RF-01.1, RF-01.2, RF-01.3, RF-01.4 | Account lifecycle |
| US-EP004-02 | EP-004 | RF-01.4 | Guardian-student linking |
| US-EP005-01 | EP-005 | RF-06.1, RF-06.2, RF-09.1, RF-09.2 | Real-time map tracking |
| US-EP005-02 | EP-005 | RF-05.4, RF-06.3 | Prolonged stop and GPS loss alerts |
| US-EP006-01 | EP-006 | RF-08.1 | Incident reporting flow |
| US-EP007-01 | EP-007 | RF-10.2 | Language switching |
| US-EP007-02 | EP-007 | RF-10.1 | Theme customization |

### 10.2 Functional Requirements Coverage

| RF ID | Covered By US(s) | Status |
| --- | --- | --- |
| RF-01.1 | US-EP004-01 | ✅ Implemented |
| RF-01.2 | US-EP004-01 | ✅ Implemented |
| RF-01.3 | US-EP004-01 | ✅ Implemented |
| RF-01.4 | US-EP004-01, US-EP004-02 | ✅ Implemented |
| RF-02.1 | — | ✅ Auth module handles credential verification |
| RF-02.2 | — | ✅ Token renewal handled by gateway/IAM |
| RF-03.1 | US-EP003-01 | ✅ Route creation flow |
| RF-03.2 | US-EP003-01 | ✅ Validation layer in route service |
| RF-03.3 | US-EP003-01 | ✅ Edit route flow |
| RF-03.4 | US-EP003-01 | ✅ Delete route flow with dependency check |
| RF-03.5 | US-EP003-02 | ✅ Bus-route assignment |
| RF-03.6 | US-EP003-02 | ✅ Bus CRUD operations |
| RF-03.7 | US-EP003-02 | ✅ Driver-bus assignment |
| RF-03.8 | US-EP003-02 | ✅ Student-route-stop assignment |
| RF-04.1 | US-EP001-01 | ✅ QR generation service |
| RF-04.2 | US-EP001-01 | ✅ QR scanner + boarding validation |
| RF-04.3 | US-EP001-02 | ✅ Drop-off QR validation |
| RF-05.1 | US-EP002-01 | ✅ Event detection pipeline |
| RF-05.2 | US-EP002-01 | ✅ Push notification dispatch |
| RF-05.3 | US-EP002-02 | ✅ Drop-off notification dispatch |
| RF-05.4 | US-EP005-02 | ✅ Stationary detection + alert |
| RF-06.1 | US-EP005-01 | ✅ GPS position ingestion |
| RF-06.2 | US-EP005-01 | ✅ Map rendering with live markers |
| RF-06.3 | US-EP005-02 | ✅ Signal loss monitoring |
| RF-07.1 | — | ✅ Trip state machine in route service |
| RF-08.1 | US-EP006-01 | ✅ Incident report form + escalation |
| RF-09.1 | US-EP005-01 | ✅ Guardian route inquiry |
| RF-09.2 | US-EP005-01 | ✅ Driver route inquiry |
| RF-10.1 | US-EP007-02 | ✅ Theme CSS variables + picker |
| RF-10.2 | US-EP007-01 | ✅ i18n framework with locale files |

---

*Document end — Software Requirements Specification for Guardian Escolar.*
