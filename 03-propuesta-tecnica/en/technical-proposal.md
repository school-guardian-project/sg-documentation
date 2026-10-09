# Software Technical Proposal — Guardian Escolar

---

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
  - [1.1 The Proposal in Brief](#11-the-proposal-in-brief)
  - [1.2 Benefits for the Institution](#12-benefits-for-the-institution)
  - [1.3 Key Proposal Figures](#13-key-proposal-figures)
- [2. Diagnosis and Problem](#2-diagnosis-and-problem)
  - [2.1 Current State of School Transportation](#21-current-state-of-school-transportation)
  - [2.2 Diagnostic Findings](#22-diagnostic-findings)
  - [2.3 Limitations of Current Solution](#23-limitations-of-current-solution)
  - [2.4 Collected Evidence](#24-collected-evidence)
  - [2.5 Contextual Constraints](#25-contextual-constraints)
- [3. Proposal Objectives](#3-proposal-objectives)
  - [3.1 General Objective](#31-general-objective)
  - [3.2 Specific Objectives](#32-specific-objectives)
  - [3.3 External Assumptions and Dependencies](#33-external-assumptions-and-dependencies)
- [4. Solution and Scope](#4-solution-and-scope)
  - [4.1 Solution Description](#41-solution-description)
  - [4.2 Guiding Principles](#42-guiding-principles)
  - [4.3 Included Scope](#43-included-scope)
  - [4.4 Excluded Scope](#44-excluded-scope)
  - [4.5 Actors and Roles](#45-actors-and-roles)
- [5. Architectural Model and Stack](#5-architectural-model-and-stack)
  - [5.1 Architectural Style and Technical Principles](#51-architectural-style-and-technical-principles)
  - [5.2 Proposed Model Overview](#52-proposed-model-overview)
  - [5.3 Components and Ports](#53-components-and-ports)
  - [5.4 Communication Patterns](#54-communication-patterns)
  - [5.5 Summary Data Model](#55-summary-data-model)
  - [5.6 Stack Justification by Layer](#56-stack-justification-by-layer)
- [6. Key Functional Capabilities](#6-key-functional-capabilities)
  - [6.1 Requirements Structure](#61-requirements-structure)
  - [6.2 Capabilities by Group](#62-capabilities-by-group)
  - [6.3 Epics and User Stories](#63-epics-and-user-stories)
- [7. Security and Compliance](#7-security-and-compliance)
  - [7.1 Identification, Authorization, and Sessions](#71-identification-authorization-and-sessions)
  - [7.2 Infrastructure Security](#72-infrastructure-security)
  - [7.3 Regulatory Compliance](#73-regulatory-compliance)
- [8. Implementation Plan](#8-implementation-plan)
  - [8.1 Phases and Milestones](#81-phases-and-milestones)
  - [8.2 Delivery Conditions](#82-delivery-conditions)
  - [8.3 Acceptance Criteria](#83-acceptance-criteria)
- [9. Quality Standards and Testing](#9-quality-standards-and-testing)
  - [9.1 Code Quality Gates](#91-code-quality-gates)
  - [9.2 Test Strategy](#92-test-strategy)
  - [9.3 Performance Targets](#93-performance-targets)
- [10. Deployment Architecture](#10-deployment-architecture)
  - [10.1 Environments](#101-environments)
  - [10.2 Containerization and Orchestration](#102-containerization-and-orchestration)
  - [10.3 Database and Migration Strategy](#103-database-and-migration-strategy)
  - [10.4 Monitoring and Observability](#104-monitoring-and-observability)
- [11. Risks and Mitigations](#11-risks-and-mitigations)
- [12. Team and Schedule](#12-team-and-schedule)
- [13. Glossary](#13-glossary)
- [14. References](#14-references)

---

## 1. Executive Summary

### 1.1 The Proposal in Brief

**Guardian Escolar** is a web and mobile platform for monitoring and securing school transportation. It integrates real-time GPS tracking, student QR code-based boarding control, automatic notifications and alerts, a web administrative panel, and mobile applications for drivers, guardians, and students. This technical proposal explains how the platform is built, secured, and deployed, with emphasis on the value delivered to the educational institution and verifiable delivery conditions.

The proposed model separates the domain into **nine domain services** behind a **single API gateway**, with **asynchronous event communication** for real-time operation and a **relational database with per-service schema** to preserve data integrity. Clients are an **Angular 21 web application** with Angular Material 3 and a **React + Expo mobile application**. Delivery is organized into **six phases** and **five verifiable milestones** across **twenty-eight weeks** (preliminary estimate).

### 1.2 Benefits for the Institution

| Benefit | How It Materializes |
| --- | --- |
| Permanent route visibility | Bus position tracked live on a map updating without manual refresh, with "no signal" indicator when connection drops. |
| Certainty about pickup/drop-off | Each boarding recorded via QR scan, validated that the student belongs to the route, immediate guardian notification. |
| Timely communication | Automatic warnings for boarding, drop-off, extended stops, incidents, and signal loss, directed to correct recipients. |
| Centralized operational control | Administration of branches, courses, people, fleet, routes, schedules, and assignments from a role-based panel. |
| Incident traceability | Boarding history, route executions, alerts, and administrative actions with who-what-when-IP-app records. |

### 1.3 Key Proposal Figures

| Metric | Value | Source |
| --- | --- | --- |
| Total functional requirements | 30 | Section 6 below; SRS document |
| Domain microservices | 9 | ms-iam, ms-user-management, ms-school, ms-fleet, ms-route, ms-notification, ms-exceptional, ms-audit, ms-settings |
| Authentication protocol | JWT RS256 | Section 7.1 |
| Event bus | Apache Kafka KRaft mode | Section 5.4 |
| Telemetry channel | gRPC persistent stream | Section 5.4 |
| Database | SQL Server 2022, single instance, per-schema | Section 10.3 |
| Web client | Angular 21 + Angular Material 3 | Section 5.6 |
| Mobile client | React Native (Expo Go) + Expo Notifications | Section 5.6 |
| API gateway | Kong OSS v3.8 | Section 5.2 |
| Target availability | 99.9% monthly uptime | Section 9.3 |

---

## 2. Diagnosis and Problem

### 2.1 Current State of School Transportation

School transport in Neiva (Huila), Colombia, relies primarily on manual processes: paper lists for roll call, WhatsApp groups for ad-hoc communication, phone calls for status updates, and scattered record-keeping. Drivers carry printed manifests; administrators compile them manually; parents have no visibility until their child arrives home or misses transportation.

The absence of automated tracking creates uncertainty, inefficiency, and safety gaps. Incidents go unreported until families discover issues independently. Route optimization relies entirely on driver knowledge rather than systematic planning. Insurance and liability documentation lacks timestamped proof of student boardings.

### 2.2 Diagnostic Findings

| Finding | Impact | Evidence |
| --- | --- | --- |
| No real-time location visibility | Parents cannot verify if the bus is en route or delayed | Interviews with 3 families and 2 administrators |
| Manual roll call error-prone | Human error in counting students leads to missed pickups or drop-offs | Observation of 2 route executions |
| Communication fragmentation | Information scattered across WhatsApp, phone calls, and paper notes | Survey: 100% of respondents use ≥2 communication channels |
| No incident documentation | Accidents or near-misses lack timestamped evidence for insurance/liability | Institutional policy review |
| No route analytics | Cannot measure punctuality, identify problematic stops, or optimize schedules | Absence of historical data collection |

### 2.3 Limitations of Current Solution

Current approaches fail at scale because they rely on human memory and multiple parallel information channels. When five buses operate simultaneously across three branches, coordinating through WhatsApp groups becomes impossible. Paper manifests are lost, damaged, or duplicated. Without digitization, there is no audit trail, no recovery path, and no basis for continuous improvement.

### 2.4 Collected Evidence

Evidence gathered during the diagnostic phase includes: structured interviews with school administrators (n=3), parent surveys (n=15), driver observations (n=2 route executions), institutional policy documents (SISAC compliance requirements), regulatory references (Law 1581/2012 – data protection; Resolution 2525/2017 – SISAC standards), and competitive analysis of existing solutions in the Colombian market.

### 2.5 Contextual Constraints

| Constraint | Detail |
| --- | --- |
| Geographic scope | Initial deployment: Neiva, Huila, Colombia. Urban area ~22 km². Routes within municipal boundaries. |
| Network conditions | Intermittent cellular coverage in some routes; offline-first capability required for critical flows. |
| Device diversity | Mix of Android mid-range phones (drivers), iOS and Android (guardians/students), tablets/desktops (admins). |
| Language | Primary: Spanish. Interface must support English, French, Portuguese as future localizations. |
| Regulatory | Colombian data protection law (Ley 1581/2012); minor data special treatment required. |
| Budget | Academic project budget constraints influence technology choices toward open-source where possible. |

---

## 3. Proposal Objectives

### 3.1 General Objective

Design and implement a digital platform enabling educational institutions in Neiva (Huila) to monitor school transportation in real time, notify guardians automatically of student boarding and alighting events, manage fleet and route assignments centrally, maintain complete operational audit trails, and provide incident reporting capabilities — all while complying with Colombian data protection regulations and achieving measurable reliability targets.

### 3.2 Specific Objectives

| # | Objective | Measurable Criterion |
| --- | --- | --- |
| SO-01 | Provide real-time GPS visualization on interactive maps | Position update latency ≤ 5 seconds end-to-end |
| SO-02 | Automate boarding and drop-off confirmation via QR scanning | 100% of trips completed with electronic boarding records |
| SO-03 | Deliver instant push notifications to guardians | Notification delivery within 10 seconds of event generation |
| SO-04 | Enable central administration of users, routes, fleet, and assignments | All CRUD operations accessible through admin dashboard |
| SO-05 | Maintain immutable audit trail of all critical actions | Every state change logged with actor, timestamp, IP, app |
| SO-06 | Support role-based access control for five distinct user types | RBAC enforced across all API endpoints |
| SO-07 | Achieve 99.9% platform availability target | Monthly downtime < 43 minutes per month |
| SO-08 | Comply with Colombian data protection regulations (Ley 1581/2012) | Privacy impact assessment passed; data subject rights implemented |
| SO-09 | Enable multi-language interface (minimum 4 languages) | i18n framework supports es, en, fr, pt with English as fallback |
| SO-10 | Provide offline capability for critical mobile flows | QR scanning and boarding event recording work with intermittent connectivity |

### 3.3 External Assumptions and Dependencies

| ID | Assumption / Dependency | Risk if Not Met | Mitigation |
| --- | --- | --- | --- |
| A-01 | Institutions provide accurate student, guardian, driver, and route data | Incorrect assignments, failed notifications | Data validation layer in admin portal; bulk import templates |
| A-02 | Users have compatible browsers or mobile devices with cameras | Requires assisted/manual fallback channel | Progressive enhancement; print-QR fallback |
| D-01 | SQL Server available for persistence | Would require architecture/persistence redesign | PostgreSQL alternative containerized evaluation pre-decision |
| D-02 | GPS device definition and contract before live monitoring | Real-time tracking unavailable | Start with simulated GPS for testing; integrate provider once contracted |
| D-03 | Push notification service defined before production | Notifications not reaching devices | Expo Push tested early; SMS/email fallback configured |

---

## 4. Solution and Scope

### 4.1 Solution Description

Guardian Escolar delivers a centralized platform integrating the following components:

1. **Web Application (Admin)**: Angular 21 + Angular Material 3 dashboard for managing schools, users, routes, fleet, and reports. Role-based views tailored to SUPER_ADMIN and ADMIN roles.

2. **Mobile Applications**:
   - **Driver App**: React + Expo application for route execution, QR scanning, incident reporting, and receiving alerts.
   - **Parent/Guardian App**: React + Expo application for live bus tracking, boarding notifications, route inquiry, and incident viewing.
   - **Student App** (basic): QR code display for identification during boarding.

3. **Microservices Backend**: Nine domain-specific services communicating via REST API, Apache Kafka events, and gRPC telemetry streaming.

4. **API Gateway**: Kong OSS handling TLS termination, authentication, rate limiting, CORS, and request routing.

5. **Infrastructure**: Docker containers orchestrated with Docker Compose for development; cloud-hosted containers planned for staging and production.

### 4.2 Guiding Principles

| Principle | Description |
| --- | --- |
| Design Before Code (SDD) | Approved design documentation precedes implementation |
| Test-Driven Development | Tests written before implementation; green tests gate merges |
| Contract-First APIs | OpenAPI contracts published before service implementation begins |
| Bounded Context Ownership | Each microservice owns its data model; cross-service access via events |
| Least Privilege Access | RBAC applied at every layer; minimal permissions per role |
| Immutable Audit Trail | Critical actions logged permanently with full context |
| Offline-First Mobile | Core flows functional with intermittent connectivity |
| Open Source Preference | Prefer OSS tools where capabilities match requirements |

### 4.3 Included Scope

| Component | Inclusion Details |
| --- | --- |
| User Management | Account creation, role assignment, activation/deactivation, guardian–student linking |
| Authentication & Authorization | JWT RS256, login/logout, token refresh, RBAC enforcement |
| Route Management | Route creation/edit/delete, stop ordering, scheduling |
| Fleet Management | Bus CRUD, driver assignment, vehicle identity management |
| Trip Execution | Trip start/completion, intermediate checkpoints, summary generation |
| QR Boarding Control | QR generation, scan validation, boarding event recording |
| Push Notifications | Pickup alert, drop-off alert, extended-stop alert, incident alert |
| Real-Time Tracking | GPS ingestion, position broadcasting, live map rendering |
| Incident Reporting | Type classification, severity levels, escalation, resolution workflow |
| Admin Dashboard | Role-specific views, data management, reporting, configuration |
| Multi-Language UI | Spanish primary; English, French, Portuguese scaffolding |
| Audit Logging | All critical actions captured with contextual metadata |

### 4.4 Excluded Scope

Elements deliberately excluded from this proposal (see also SRS Section 1.2 exclusions):

| Excluded Element | Reason |
| --- | --- |
| Payments, billing, invoicing for transport service | Outside safety and monitoring purpose |
| Fleet mechanical maintenance, fuel, repair shop management | Not a confirmed user flow |
| Academic management (grades, attendance, learning) | Platform manages transport only |
| Public transit and non-school routes | Domain limited to school transport |
| Physical purchase/administration/installation of GPS hardware | External responsibility |
| Autonomous emergency response or automatic dispatch | Human/institutional response is external |
| Biometric identification or facial recognition | Not required for current scope; raises privacy obligations |
| Multinational regulatory geolocation | Initial context is Neiva (Huila), Colombia |
| Autonomous driver navigation and AI-optimal route generation | Outside current product scope |

### 4.5 Actors and Roles

| Role | Title | Responsibilities | Interface |
| --- | --- | --- | --- |
| SUPER_ADMIN | Platform Operator | Manages all institutions, global configuration, platform-wide audit | Web |
| ADMIN | School Administrator | Manages their institution's schools, courses, people, fleet, routes, reports | Web |
| DRIVER | Driver | Executes routes, scans QR codes for boarding, receives alerts | Mobile |
| PARENT | Guardian/Parent | Tracks routes, receives notifications, views boarding history | Mobile |
| STUDENT | Student | Displays QR code for identification | Mobile |

External actors: GPS hardware device (emits positions), push notification provider (Expo/Firebase Cloud Messaging).

---

## 5. Architectural Model and Stack

### 5.1 Architectural Style and Technical Principles

**Architecture style:** Event-oriented microservices behind a single API Gateway. Services communicate synchronously via REST/JSON for request-response and asynchronously via Apache Kafka events for domain-level coordination. Low-latency telemetry uses a dedicated gRPC bidirectional stream.

**Technical principles:**
- Services are independently deployable containers with bounded-context ownership.
- Each service has its own database schema; no cross-service direct DB access.
- API contracts defined via OpenAPI/Swagger before implementation.
- All secrets stored in environment variables or vault; never in source code.
- Health checks and metrics exposed on every service.
- CI/CD pipeline with quality gates: lint → build → test → security scan → package → deploy.

### 5.2 Proposed Model Overview

```
         Parent / Student / Driver Apps      Admin Web App
               (React + Expo)                     (Angular)
                           │                          │
                           └───────────┬──────────────┘
                                       │ HTTPS :8000
                                       ▼
                         ┌─────────────────────────────┐
                         │  API Gateway (Kong) :8000   │
                         │  TLS · CORS · Rate Limit    │
                         │  JWT RS256 · Routing        │
                         └──────────────┬──────────────┘
                                        │ REST/JSON :8080
     ┌─────────┬─────────┬─────────┬───┼───┬─────────┬─────────┐
     ▼         ▼         ▼         ▼   ▼   ▼         ▼         ▼
    IAM     User Mgmt  School   Fleet Routes Notif Exception  Audit Settings
                              Mgmt
     └─────────┴─────────┴─────────┴───┼───┴─────────┴─────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                  ▼                  ▼
           SQL Server :1433     Kafka :9092        gRPC :5001
         (per-service schemas)  (domain events)  (GPS positions)
```

### 5.3 Components and Ports

| Component | Protocol | Port(s) | Purpose |
| --- | --- | --- | --- |
| Kong API Gateway | HTTPS/TLS 1.3 | 8000 | Public entry point |
| Microservices (internal) | REST/JSON | 8080 | Business logic |
| Apache Kafka | TCP binary | 9092 | Event bus |
| gRPC telemetry | gRPC/TLS | 5001 | GPS streaming |
| SQL Server | TDS/TLS | 1433 | Persistence |
| Redis cache | RESP | 6379 | Session/rate-limit caching |

### 5.4 Communication Patterns

#### 5.4.1 Domain Events

| Event | Publisher | Subscribers | Payload Summary |
| --- | --- | --- | --- |
| `profile.created` | IAM Service | Audit, Notifications | profileId, roleId, timestamp |
| `profile.updated` | IAM Service | Audit | profileId, fieldsChanged, timestamp |
| `route.scheduled` | Routes Service | Fleet, Notifications | routeId, date, time, branch |
| `trip.started` | Routes Service | Fleet, Notifications, GPS | tripId, routeId, driverId |
| `trip.completed` | Routes Service | Fleet, Notifications, Audit | tripId, summary, duration |
| `student.scanned` | Notifications Service | Audit, Notifications | studentId, eventId, timestamp, type(pickup/dropoff) |
| `alert.critical` | Notifications Service | Guardians, Admin | alertType, severity, routeId, timestamp |
| `position.emitted` | Fleet Service (gRPC) | Live Map Subscribers | deviceId, lat, lng, speed, heading, timestamp |

#### 5.4.2 Boarding Flow Sequence

```
Driver scans QR → Notifications validates (assigned? active trip?)
  → Boarding event persisted → student.scanned event published
  → Guardian notification queued → pushed via Expo
  → Guardián receives "Student picked up" on mobile device
```

#### 5.4.3 Representative Resources

| Resource | HTTP Method | Path | Auth Required | Response |
| --- | --- | --- | --- | --- |
| Create account | POST | /api/v1/users | ADMIN+ | 201 Created |
| Login | POST | /api/v1/auth/login | No (credentials) | 200 {token, refresh} |
| Refresh token | POST | /api/v1/auth/refresh | Yes (refresh) | 200 {newToken} |
| Get route | GET | /api/v1/routes/:id | Yes (role) | 200 {route, stops} |
| Assign bus | PUT | /api/v1/routes/:id/bus | ADMIN+ | 200 {assignment} |
| Scan QR | POST | /api/v1/boardings/scan | DRIVER+ | 200 {event} or 400 Error |

### 5.5 Summary Data Model

| Service | Core Tables | Relationship Notes |
| --- | --- | --- |
| IAM | profiles, roles, credentials, sessions, refresh_tokens | One profile per person; hashed passwords stored here |
| User Management | persons, academic_records, guardian_links | Persons may be students, drivers, parents; academics for students |
| School | institutions, branches, courses, grades | Hierarchical: institution → branches → courses → grades |
| Fleet | buses, gps_devices, vehicle_maintenance | One GPS per bus; maintenance logs per bus |
| Route | routes, stops, route_assignments, driver_assignments | Stops ordered within route; assignments link drivers to buses to routes |
| Geolocation | position_events, route_executions | position_events linked to route_execution; append-only table |
| Notifications | notification_queue, push_delivery, delivery_status | Queue managed by worker; status tracked for delivery confirmation |
| Exceptional | incident_reports, remediation_actions | Incidents linked to route execution; actions track resolution |
| Audit | action_logs | Append-only cross-cutting concern; records all critical system actions |

### 5.6 Stack Justification by Layer

#### Backend Services

| Technology | Choice Rationale |
| --- | --- |
| Java 21 + Spring Boot (IAM) | Mature ecosystem, strong auth/security libraries, team experience |
| C# + .NET 10 (other services) | Strong typing, performance, excellent ORM (EF Core), broad ecosystem |
| Kafka (event bus) | Proven high-throughput messaging; KRaft mode eliminates ZooKeeper dependency |
| gRPC (telemetry) | Binary protocol efficiency; streaming support ideal for continuous GPS feeds |
| SQL Server 2022 (DB) | ACID guarantees, mature tooling, good JSON support, preferred by organization |

#### Data Layer

| Technology | Choice Rationale |
| --- | --- |
| SQL Server 2022 | Relational integrity; per-schema isolation provides logical separation |
| Liquibase (migrations) | Version-controlled, idempotent migrations with rollback scripts |
| Redis (cache) | In-memory speed for session caching, rate-limit counters, temporary state |

#### Client Applications

| Technology | Choice Rationale |
| --- | --- |
| Angular 21 + Material 3 (web admin) | TypeScript strictness, component library, enterprise maturity |
| React + Expo (mobile) | Cross-platform single codebase; Expo ecosystem for notifications, camera, QR |
| HTML/CSS/JS for static assets | Standard web technologies for CDN-hosted resources |

#### Infrastructure

| Technology | Choice Rationale |
| --- | --- |
| Docker (containerization) | Process isolation; reproducible builds across environments |
| Docker Compose (local orchestration) | Simple multi-service setup for development |
| Kong OSS (API gateway) | Open source, Lua scripting extensibility, well-documented plugins |

---

## 6. Key Functional Capabilities

### 6.1 Requirements Structure

The platform implements **30 functional requirements** organized into **10 functional groups**:

| Group | Count | Priority | Owner |
| --- | --- | --- | --- |
| RF-01 User Management | 4 | High | User Management + IAM |
| RF-02 Authentication | 2 | High | IAM |
| RF-03 Routes/Fleet/Assignments | 8 | High | Routes + Fleet |
| RF-04 QR Scanning | 3 | High | IAM + Notifications |
| RF-05 Boarding Notifications | 4 | High | Notifications |
| RF-06 Real-Time Tracking | 3 | Medium | Geolocation + gRPC |
| RF-07 Trip Management | 1 | Medium | Routes |
| RF-08 Incidents | 1 | High | Notifications |
| RF-09 Route Inquiry | 2 | Medium | Routes |
| RF-10 Personalization | 2 | Low | Clients |

### 6.2 Capabilities by Group

**RF-01 User Management (4 RF):** Complete account lifecycle: creation with role assignment, data update, activation/deactivation, and guardian-student linking. No self-registration; all accounts created by admins.

**RF-02 Authentication (2 RF):** JWT RS256 credential verification, token issuance with 15-minute expiry, refresh token mechanism with 30-day validity, logout capability.

**RF-03 Routes, Fleet & Assignments (8 RF):** Route CRUD with ordered stops and schedules, uniqueness validation, bus fleet management, driver-bus-route assignment chains with one-to-one relationships.

**RF-04 QR Scanning (3 RF):** Individual QR code generation per student, pickup scan with validation against active trip, drop-off scan ensuring prior pickup exists.

**RF-05 Notifications (4 RF):** Automatic pickup/drop-off push notifications to guardians, extended-stop alerts, incident alerts with severity-based routing.

**RF-06 Real-Time Tracking (3 RF):** Continuous GPS capture via gRPC, interactive map visualization, signal-loss alerting after 2-minute gap threshold.

**RF-07 Trip Management (1 RF):** Trip lifecycle state machine: PLANNED → IN_PROGRESS → COMPLETED, with abnormal termination path.

**RF-08 Incidents (1 RF):** Incident reporting with type/severity classification, automatic notification to affected parties, escalation workflow.

**RF-09 Route Inquiry (2 RF):** Guardian view of student's assigned route with stops/times/schedule; driver view of daily route plan.

**RF-10 Personalization (2 RF):** Color theme customization respecting WCAG contrast; multi-language interface supporting four languages with English fallback.

### 6.3 Epics and User Stories

| Epic | Focus Area | User Stories | Status |
| --- | --- | --- | --- |
| EP-001 QR Boarding | QR code lifecycle and scanning | US-EP001-01 (driver scan pickup), US-EP001-02 (driver see route list) | Design → Implementation |
| EP-002 Notifications | Push notification delivery | US-EP002-01 (pickup alert), US-EP002-02 (drop-off alert) | Design → Implementation |
| EP-003 Routes & Fleet | Route and fleet administration | US-EP003-01 (create route), US-EP003-02 (assign bus/driver) | Design → Implementation |
| EP-004 Users & Families | Account and guardian management | US-EP004-01 (create accounts), US-EP004-02 (link students) | Design → Implementation |
| EP-005 Live Tracking | GPS and map functionality | US-EP005-01 (live map), US-EP005-02 (prolonged stop alert) | Design → Implementation |
| EP-006 Incidents | Incident reporting | US-EP006-01 (report incident) | Design → Implementation |
| EP-007 UI Customization | Theming and localization | US-EP007-01 (language switch), US-EP007-02 (theme picker) | Design → Implementation |

---

## 7. Security and Compliance

### 7.1 Identification, Authorization, and Sessions

| Aspect | Configuration |
| --- | --- |
| Authentication | JWT RS256 signed tokens; private key held exclusively by IAM service |
| Access Token Lifetime | 15 minutes (configurable, default 900 seconds) |
| Refresh Token Lifetime | 30 days (configurable; rotated on each use for forward-secrecy) |
| Password Policy | Minimum 12 characters; uppercase, lowercase, numbers, symbols; bcrypt salt rounds ≥ 12 |
| Session Management | Stateless JWT validation; server-side refresh token store enables revocation |
| Authorization | RBAC: SUPER_ADMIN, ADMIN, DRIVER, PARENT, STUDENT; permission checked on every request |
| Least Privilege | Each role granted minimum required permissions; admin actions require explicit permission flags |

### 7.2 Infrastructure Security

| Control | Implementation |
| --- | --- |
| Encryption in transit | TLS 1.3 for all external traffic; internal mTLS between services |
| Secrets management | Environment variables; no hardcoded secrets; Kubernetes secrets for production |
| Input validation | OWASP Top 10 mitigations: parameterized queries, output encoding, CSRF tokens, XSS sanitization |
| Rate limiting | Kong rate-limit plugin: per-client configurable limits; burst handling |
| CORS | Explicitly listed origins; no wildcard (*) in production |
| Audit logging | All critical actions (user creation/modification/deletion, config changes, security events) logged with actor, timestamp, IP, application, and outcome |
| Vulnerability scanning | Automated dependency scanning (npm audit, OWASP Dependency-Check) in CI pipeline |

### 7.3 Regulatory Compliance

| Regulation | Compliance Measures |
| --- | --- |
| Ley 1581 of 2012 (Colombia) | Data processing purpose limitation; consent capture; right of access/rectification/deletion; breach notification procedures |
| Decreto 1377 of 2013 | Special treatment for minors' data: explicit parental consent required before processing any minor's personal data |
| GDPR (if offered in EU) | Data portability, right to erasure, DPIA procedures, data processing agreements with subprocessors |
| PCI-DSS | Not applicable — payment processing is out of scope for this project |

---

## 8. Implementation Plan

### 8.1 Phases and Milestones

| Phase | Duration | Deliverables | Milestone |
| --- | --- | --- | --- |
| P1: Setup & Foundation | Weeks 1–2 | Repository structure, CI/CD pipeline, Dev environment, First microservice skeleton | MS-1: Dev environment operational |
| P2: Core Services | Weeks 3–6 | IAM + auth, User Management, basic routes | MS-2: Accounts can be created, users authenticate |
| P3: Route & Fleet | Weeks 7–10 | Routes CRUD, Fleet management, Driver-bus assignments | MS-3: Admin can configure routes and assign fleet |
| P4: QR & Notifications | Weeks 11–14 | QR code generation/scanning, Push notification delivery | MS-4: Boarding events trigger guardian notifications |
| P5: GPS & Live Map | Weeks 15–18 | GPS ingestion, gRPC streaming, Live map rendering | MS-5: Real-time bus tracking functional |
| P6: Polish & Launch | Weeks 19–28 | Incidents, personalization, testing, hardening, documentation | MS-6: Platform ready for production acceptance |

### 8.2 Delivery Conditions

| Condition | Requirement | Verification |
| --- | --- | --- |
| Code quality | Linting clean; cyclomatic complexity ≤ 10 per method | SonarQube gate check |
| Test coverage | ≥ 80% lines, ≥ 70% branches per service | JaCoCo/Istanbul report in CI |
| Security | Zero critical/high vulnerabilities | OWASP ZAP scan results |
| Performance | API p95 ≤ 200ms; GPS update ≤ 5s | Load test report with percentiles |
| Documentation | API docs updated; runbooks authored | Peer review sign-off |
| UAT | Product owner acceptance | Signed UAT report |

### 8.3 Acceptance Criteria

The platform achieves production readiness when:

1. All 30 functional requirements implemented and verified.
2. Non-functional targets met (Section 9.3).
3. Zero critical or high-severity defects open.
4. UAT signed off by product owner and institution representative.
5. Operational documentation complete and reviewed.
6. Penetration test completed with findings remediated.
7. Backup and restore procedure documented and tested.

---

## 9. Quality Standards and Testing

### 9.1 Code Quality Gates

| Gate | Tool | Threshold | Action if Failed |
| --- | --- | --- | --- |
| Lint check | ESLint (JS), SpotBugs/PMD (Java), Roslyn (C#) | Zero warnings/errors | Block merge |
| Build | npm/yarn/maven/dotnet | Clean compilation | Block merge |
| Unit tests | Jest/Vitest, JUnit, xUnit | ≥ 80% line, ≥ 70% branch coverage | Block merge |
| Integration tests | Testcontainers, embedded Kafka | All pass | Block merge |
| Contract tests | Pact, Spring Cloud Contract | Consumer/provider match | Block merge |
| Static security analysis | OWASP Dependency-Check, Bandit | No CVEs with CVSS ≥ 7 | Block merge |
| Docker image scan | Trivy, Snyk | No HIGH/CRITICAL vulns | Warn; require justification |

### 9.2 Test Strategy

| Level | Scope | Tooling | Frequency |
| --- | --- | --- | --- |
| Unit | Single function/method behavior | Jest (clients), JUnit (Java), xUnit (C#) | Every commit |
| Integration | Service ↔ DB, service ↔ Kafka, service ↔ gRPC | Testcontainers, embedded Kafka, mock gRPC | Every PR |
| Contract | API consumer ↔ provider compatibility | Pact | Before deploy |
| End-to-End | Complete user workflows | Playwright (web), Detox (mobile) | Nightly + pre-release |
| Load/Performance | Concurrent user patterns, peak load | JMeter, Gatling | Sprint milestone |
| Security | Penetration testing, vulnerability scanning | OWASP ZAP, Burp Suite | Quarterly + pre-release |

### 9.3 Performance Targets

| Metric | Target | Measurement Method |
| --- | --- | --- |
| API Gateway p95 latency | ≤ 200ms | JMeter concurrent load test |
| API Gateway p99 latency | ≤ 500ms | Same test, 99th percentile |
| Authentication response | ≤ 1 second | Load test simulating login spikes |
| GPS update to display | ≤ 5 seconds end-to-end | Latency tracer instrumenting full chain |
| QR scan processing | ≤ 2 seconds | Transaction timer in scan endpoint |
| Push notification delivery | ≤ 10 seconds | Timestamp comparison: event generated vs. received |
| Page/app load time | ≤ 3 seconds on 4G | Lighthouse (web), expo diagnostics (mobile) |
| ConcurrentUser capacity | ≥ 500 sessions | Load test with ramp-up to 500 concurrent users |

---

## 10. Deployment Architecture

### 10.1 Environments

| Environment | Purpose | Deployed To | Access |
| --- | --- | --- | --- |
| Local (dev) | Developer feature development | Developer machines via Docker Compose | Localhost only |
| Test (staging) | QA testing, UAT, performance benchmarks | Staging VM/cluster mimicking production topology | Authorized testers |
| Production | Live operation serving institution users | Production infrastructure (cloud or on-premises) | Restricted to authenticated users |

Each environment isolated with separate databases, Kafka clusters, and secret stores. No shared state across environments.

### 10.2 Containerization and Orchestration

All services packaged as Docker images built from multi-stage Dockerfiles minimizing final image size:

| Image Tag Pattern | Example | Purpose |
| --- | --- | --- |
| `{service}:latest` | `iam-service:latest` | Development reference |
| `{service}:{commit-sha}` | `iam-service:a3f2b1c` | Reproducible builds tied to git commit |
| `{service}:{semver}` | `iam-service:1.0.0` | Release-tagged artifacts |

Docker Compose orchestrates local development:

```yaml
version: '3.9'
services:
  kong:
    image: kong:3.8
    ports: ['8000:8000', '8443:8443']
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    ports: ['1433:1433']
  kafka:
    image: confluentinc/cp-kafka:7.6.1
    ports: ['9092:9092']
  redis:
    image: redis:7-alpine
    ports: ['6379:6379']
  iam-service:
    build: ./services/iam
    ports: ['8081:8080']
  user-mgmt-service:
    build: ./services/user-mgmt
    ports: ['8082:8080']
  ... other services ...
```

Production deployment to Kubernetes planned for scaling needs beyond current project scope.

### 10.3 Database and Migration Strategy

- Single SQL Server instance with per-service schema (`Iam`, `UserManagement`, `School`, `Fleet`, `Route`, `Geolocation`, `Notifications`, `Exceptional`, `Audit`).
- Liquibase manages all schema changes through versioned changelogs (`changelog-master.yaml`).
- Each changelog file adds tables, columns, indexes, or constraints in idempotent fashion.
- Rollback scripts provided for every migration.
- Migration executed on service startup (first-run detection prevents duplicate application).
- Connection pooling maintained per service; max pool size configurable via environment variable.

### 10.4 Monitoring and Observability

| Capability | Tool | Data Captured |
| --- | --- | --- |
| Metrics | Prometheus exporters per service | Request rates, latencies, errors, business counters |
| Dashboards | Grafana | Pre-built dashboards per service + aggregate view |
| Logs | Structured JSON logs aggregated via ELK or Loki | Application logs, audit events, error traces |
| Distributed Tracing | OpenTelemetry SDK per service | Request spans across gateway → service → DB/events |
| Alerting | PagerDuty integration | Threshold breaches triggering pages to on-call engineer |

Health endpoints: `/health` on every service returns status OK/DEGRADED/UNHEALTHY with dependency health summary.

---

## 11. Risks and Mitigations

| Risk | Probability | Impact | Mitigation Strategy |
| --- | --- | --- | --- |
| Intermittent GPS signal on routes | High | Real-time tracking degrades | Store-and-forward buffering; signal-loss alerts; graceful degradation to "last known" position |
| Cell network outage during trip | Medium | GPS stream and push notifications interrupted | Offline QR scanning works without connectivity; events batch-sync when reconnected |
| Parent/guardian does not have smartphone | Low | Cannot receive app notifications | SMS fallback configured for critical alerts; physical QR cards available |
| Driver forgets to charge GPS device | Medium | Position data gap for entire shift | Battery life ≥ 12 hours; low-battery alert sent to driver; redundant power adapter in bus |
| Team delivery delays due to scope growth | Medium | Timeline extension required | Strict scope control; change request process; prioritize features per MoSCoW method |
| Data protection compliance violation | Low | Legal/regulatory consequences | Privacy Impact Assessment conducted early; data minimization principle; consent flows designed first |
| Third-party service outage (Expo push, SMS provider) | Low | Notification delivery failure | Fallback channel (email/SMS alternation); retry with exponential backoff; queue persistence |

---

## 12. Team and Schedule

| Role | Responsibility | Allocation |
| --- | --- | --- |
| Project Lead / Tech Lead | Architecture decisions, code reviews, sprint planning | Full-time equivalent |
| Backend Developer (Java/Spring) | IAM service, authentication, authorization | 1.5 FTE |
| Backend Developer (C#/.NET) | Remaining 8 microservices | 1.0 FTE |
| Frontend Developer (Angular) | Web admin dashboard | 0.5 FTE |
| Mobile Developer (React/Expo) | Driver app, Guardian app, Student QR display | 1.0 FTE |
| QA/Test Engineer | Test strategy, automation, UAT coordination | 0.5 FTE |
| Instructor/Mentor | Guidance, milestone reviews, quality oversight | Part-time |

Total estimated effort: ~6.0 FTE-equivalents over 28 weeks.

---

## 13. Glossary

| Term | Definition |
| --- | --- |
| Boarding Event (Bordada) | Student pickup or drop-off registration on the bus |
| API Gateway | Single entry point (Kong) handling routing, auth, rate limiting |
| Microservice | Independently deployable unit owning a bounded context |
| JWT | JSON Web Token for stateless authentication assertions |
| gRPC | High-efficiency RPC protocol for telemetry streaming |
| Kafka | Distributed event streaming platform |
| Liquibase | Database migration and versioning tool |
| SLO | Service Level Objective — reliability target |
| RBAC | Role-Based Access Control |
| UAT | User Acceptance Testing |
| SISAC | Superintendencia de Transporte Colombiano — transport regulator |
| Expo | React Native development platform with unified tooling |
| Docker Compose | Multi-container orchestration for local development |

---

## 14. References

1. IEEE Std 830-1998 — Guide to Software Requirements Specification
2. ISO/IEC 25010:2011 — Systems and software engineering — Quality model
3. ISO/IEC/IEEE 12207 — Software life cycle processes
4. OWASP Top 10 2021 — Top ten web application risks
5. Law 1581 of 2012 (Colombia) — Personal data protection
6. Decree 1377 of 2013 (Colombia) — Regulations for Law 1581
7. Kong API Gateway documentation — https://docs.konghq.com/
8. Apache Kafka documentation — https://kafka.apache.org/documentation/
9. JWT specification — https://jwt.io/introduction/
10. Google Firebase Cloud Messaging documentation — Expo Push Notifications guide
11. Angular documentation — https://angular.dev/
12. React Native / Expo documentation — https://docs.expo.dev/

---

*Document end — Technical Proposal for Guardian Escolar.*
