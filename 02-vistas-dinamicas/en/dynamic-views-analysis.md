# Dynamic Views Analysis — Guardian Escolar

---

## Table of Contents

- [1. Introduction](#1-introduction)
  - [1.1 Purpose](#11-purpose)
  - [1.2 Scope](#12-scope)
  - [1.3 UML Notation Used](#13-uml-notation-used)
  - [1.4 Diagram and Message Conventions](#14-diagram-and-message-conventions)
  - [1.5 Recurring Participants](#15-recurring-participants)
- [2. General Use Case Diagram](#2-general-use-case-diagram)
  - [2.1 General Description](#21-general-description)
  - [2.2 Use Case Matrix](#22-use-case-matrix)
  - [2.3 Use Case Relationships](#23-use-case-relationships)
- [3. Sequence Diagrams](#3-sequence-diagrams)
  - [3.1 SEQ-01 — Login and Token Renewal](#31-seq-01--login-and-token-renewal)
  - [3.2 SEQ-02 — User Creation with *.created Event](#32-seq-02--user-creation-with-created-event)
  - [3.3 SEQ-03 — Branch and Course Creation](#33-seq-03--branch-and-course-creation)
  - [3.4 SEQ-04 — Driver–Bus–Route Assignment](#34-seq-04--driver-bus-route-assignment)
  - [3.5 SEQ-05 — Route Planning and Scheduling](#35-seq-05--route-planning-and-scheduling)
  - [3.6 SEQ-06 — Route Execution Start and Completion](#36-seq-06--route-execution-start-and-completion)
  - [3.7 SEQ-07 — Pickup/Drop-off QR Scanning (student.scanned)](#37-seq-07--pickupper-drop-off-qr-scanning-studentscanned)
  - [3.8 SEQ-08 — GPS Position Reception via gRPC](#38-seq-08--gps-position-reception-via-grpc)
  - [3.9 SEQ-09 — Alert Generation and Push Notification Delivery](#39-seq-09--alert-generation-and-push-notification-delivery)
- [4. Activity Diagrams](#4-activity-diagrams)
  - [4.1 ACT-01 — Student Pickup/Drop-off Process with Validations](#41-act-01--student-pickupper-drop-off-process-with-validations)
  - [4.2 ACT-02 — Alert Handling Process](#42-act-02--alert-handling-process)
  - [4.3 ACT-03 — User Creation and Edition with Events](#43-act-03--user-creation-and-edition-with-events)
  - [4.4 ACT-04 — Exceptional Route or Bus Usage](#44-act-04--exceptional-route-or-bus-usage)
  - [4.5 ACT-05 — Critical Actions Audit Procedure](#45-act-05--critical-actions-audit-procedure)
- [5. State Machine Diagrams](#5-state-machine-diagrams)
  - [5.1 EST-01 — User Session](#51-est-01--user-session)
  - [5.2 EST-02 — Route Execution](#52-est-02--route-execution)
  - [5.3 EST-03 — Alert](#53-est-03--alert)
  - [5.4 EST-04 — Student Trip (Boarding Event)](#54-est-04--student-trip-boarding-event)
  - [5.5 EST-05 — GPS Device](#55-est-05--gps-device)
  - [5.6 EST-06 — Access Profile (ACTIVE/INACTIVE)](#56-est-06--access-profile-activeinactive)
- [6. Dynamic Traceability ↔ FR/US](#6-dynamic-traceability--frus)
  - [6.1 Coverage by Requirements Group](#61-coverage-by-requirements-group)
  - [6.2 Traceability Observations](#62-traceability-observations)

---

## 1. Introduction

### 1.1 Purpose

This document specifies the **dynamic views** of the Guardian Escolar software — those system descriptions representing behavior over time through message exchanges between participants, action sequences with decision points, and persistent element lifecycles. Guardian Escolar is a web and mobile platform for school transportation monitoring and safety, featuring real-time GPS tracking, QR code-based boarding control, and push notifications and alerts.

The dynamic views complement the static views (requirements, data model, architecture) by answering three analysis questions: what messages do participants exchange and in what order; what rules and decisions govern a business process; and in what states can each entity be found and what transitions trigger them.

### 1.2 Scope

Dynamic modeling scope includes:

- The **general use cases** of the platform with their actors, functional requirements, and linked user stories.
- **Sequence diagrams** for critical flows: authentication, user onboarding with events, institutional administration, transport assignments, route planning, trip execution, QR boarding control, GPS telemetry, and alert delivery.
- **Activity diagrams** for business processes with validation rules, including exceptional usage flows and the audit procedure for critical actions.
- **State machine diagrams** for entities with relevant lifecycle: user session, route execution, alert, student trip, GPS device, and access profile.

Static views (class structure, detailed data model, graphical interfaces), installation procedures, and operational guides are treated separately.

### 1.3 UML Notation Used

Notation is limited to the three types of behavioral diagrams defined in UML 2.5:

| Diagram Type | Elements Used | Purpose in Document |
| --- | --- | --- |
| Sequence | Participants, lifelines, numbered message arrows and responses | Client–gateway–services–broker message flow |
| Activity | Swimlanes, activity nodes, decision nodes, forks/joins, initial/final nodes, guard conditions | Business processes with validation rules |
| State Machine | States, transitions, guards, entry/exit actions, internal activities | Entity lifecycle management |

### 1.4 Diagram and Message Conventions

Numbering convention:

| Prefix | Meaning |
| --- | --- |
| UC-* | Use case identifier |
| SEQ-* | Sequence diagram identifier |
| ACT-* | Activity diagram identifier |
| EST-* | State machine identifier |

Message numbering within sequence diagrams uses decimal notation (e.g., `1.`, `1.1`, `1.2`) reflecting nesting depth for alt/opt/ref fragments.

### 1.5 Recurring Participants

| Actor/System | Role | Interface |
| --- | --- | --- |
| Parent/Guardian App | Mobile client initiating user requests | React + Expo |
| Admin Web App | Web client for administrative tasks | Angular 21 + Material 3 |
| Driver App | Mobile client for route execution | React + Expo |
| Kong API Gateway | Single entry point handling auth, routing, rate limiting | HTTPS :8000 |
| IAM Service | Identification, authentication, authorization, JWT issuance | REST :8080 |
| User Management Service | Account data CRUD operations | REST :8080 |
| Routes Service | Route configuration, assignments, trip lifecycle | REST :8080 |
| Fleet Service | Bus and driver assignment management | REST :8080 |
| Notifications Service | Event processing, notification dispatch, QR validation | REST :8080 |
| Kafka Broker | Event bus for domain event publishing/subscribing | TCP :9092 |
| gRPC Telemetry Channel | Low-latency GPS position streaming | gRPC :5001 |
| SQL Server | Relational persistence | TDS :1433 |
| GPS Device | On-board hardware emitting positions | Cellular → gRPC |
| Expo Push Service | External push notification provider | HTTPS callbacks |

---

## 2. General Use Case Diagram

### 2.1 General Description

The Guardian Escolar system has four authenticated actors and one external hardware actor. Each actor interacts with specific subsystems:

```
          ┌──────────┐         ┌──────────────────────┐
          │  ADMIN   │         │    SUPER_ADMIN       │
          │  School  │         │   Platform Operator  │
          └────┬─────┘         └──────────┬───────────┘
               │                          │
               ▼                          ▼
          ┌──────────────────────────────────────┐
          │           Kong API Gateway            │
          │        TLS · Auth · Rate Limit        │
          └──────────┬───────────────────────────┘
                     │
        ┌────────────┼────────────────────────────┐
        ▼            ▼                            ▼
  ┌──────────┐  ┌──────────┐            ┌───────────────┐
  │   WEB    │  │  MOBILE  │            │  EXTERNAL     │
  │  CLIENTS │  │  CLIENTS │            │  SYSTEMS      │
  └──────────┘  └──────────┘            └───────────────┘
```

Primary stakeholders:
- **SUPER_ADMIN**: Manages all institutions, global settings, and platform-wide audit.
- **ADMIN**: Manages their institution's schools, courses, people, fleet, routes, and reports.
- **DRIVER**: Executes assigned routes, confirms boardings via QR scanning, receives alerts.
- **PARENT**: Follows routes, receives boarding/drop-off notifications, views boarding history.
- **STUDENT**: Views their QR identification for boarding purposes.
- **GPS Device** (external): Emits position data during active trips.

### 2.2 Use Case Matrix

| ID | Name | Actor(s) | Related RF | Priority |
| --- | --- | --- | --- | --- |
| UC-01 | Create Account | ADMIN, SUPER_ADMIN | RF-01.1 | High |
| UC-02 | Update Account Data | ADMIN | RF-01.2 | Medium |
| UC-03 | Activate/Deactivate Account | ADMIN | RF-01.3 | Medium |
| UC-04 | Link Students to Guardian | ADMIN | RF-01.4 | Medium |
| UC-05 | Login & Get Token | All authenticated users | RF-02.1 | High |
| UC-06 | Refresh Token | All authenticated users | RF-02.2 | Medium |
| UC-07 | Create Route | ADMIN | RF-03.1 | High |
| UC-08 | Update Route | ADMIN | RF-03.3 | Medium |
| UC-09 | Delete Route | ADMIN | RF-03.4 | Medium |
| UC-10 | Assign Bus to Route | ADMIN | RF-03.5 | High |
| UC-11 | Manage Buses | ADMIN | RF-03.6 | High |
| UC-12 | Assign Driver to Bus | ADMIN | RF-03.7 | High |
| UC-13 | Assign Students to Route | ADMIN | RF-03.8 | High |
| UC-14 | Scan QR – Pickup | DRIVER | RF-04.2 | High |
| UC-15 | Scan QR – Drop-off | DRIVER | RF-04.3 | High |
| UC-16 | Generate QR Code | System (auto) | RF-04.1 | Medium |
| UC-17 | View Route Details | PARENT, DRIVER | RF-09.1, RF-09.2 | Medium |
| UC-18 | Track Bus Live | PARENT | RF-06.2 | High |
| UC-19 | Report Incident | DRIVER, ADMIN | RF-08.1 | High |
| UC-20 | Personalize Interface | All users | RF-10.1, RF-10.2 | Low |

### 2.3 Use Case Relationships

| Primary UC | Extension Point | Extended UC | Condition |
| --- | --- | --- | --- |
| UC-14 | Scan QR – Pickup | UC-16 | QR must exist before scan |
| UC-14 | Scan QR – Pickup | UC-17 | After successful scan, show route info |
| UC-15 | Scan QR – Drop-off | UC-16 | Same prerequisite as pickup |
| UC-15 | Scan QR – Drop-off | UC-18 | Drop-off triggers location update |
| UC-01 | Create Account | UC-05 | New account immediately can login |
| UC-19 | Report Incident | UC-17 | Incident report references current route |

Include relationships:

| Base Use Case | Included Use Case | Rationale |
| --- | --- | --- |
| UC-05, UC-06 | Authentication validation | Every logged-in session requires valid credentials |
| UC-14, UC-15 | QR validation | Every scan validates against database |
| UC-07 through UC-13 | RBAC check | Every admin action requires role verification |
| UC-18 | Real-time position lookup | Map display requires live GPS stream subscription |

---

## 3. Sequence Diagrams

### 3.1 SEQ-01 — Login and Token Renewal

```
Parent App          Admin Web       Kong GW         IAM Svc         DB (IAM)
     │                  │              │               │               │
     │  POST /auth/login│              │               │               │
     ├─────────────────►│──────────────►               │               │
     │                  │              │  POST /internal/auth       │
     │                  │              ├───────────────►               │
     │                  │              │               │  SELECT * FROM │
     │                  │              │               ├──────────────►│
     │                  │              │               │  WHERE email=? │
     │                  │              │               │◄──────────────┤
     │                  │              │               │  hash check    │
     │                  │              │               │  (bcrypt)      │
     │                  │              │               │---------------│
     │                  │              │  { "token": jwt, │             │
     │                  │              │    "refresh": rf }│           │
     │                  │              │◄───────────────┤               │
     │                  │              │               │               │
     │  { token, refresh }              │               │               │
     │◄─────────────────┴───────────────┤               │               │
     │                                  │               │               │
     │  ── 15 min later ────────────────────────────────────────────────│
     │                                  │               │               │
     │  POST /auth/refresh              │               │               │
     ├─────────────────►│──────────────►               │               │
     │                  │              │  POST /internal/refresh      │
     │                  │              ├───────────────►               │
     │                  │              │  validate refresh token      │
     │                  │              │  sign new access token       │
     │                  │              │◄───────────────┤               │
     │                  │              │  { "token": jwt_new }        │
     │                  │              │◄───────────────┤               │
     │  { new token }                               │               │               │
     │◄─────────────────┴───────────────┤               │               │
```

**Preconditions:** User account exists, ACTIVE state, password set.
**Postconditions:** JWT issued with role claim; refresh token stored server-side.
**Exceptions:**
- Invalid credentials → HTTP 401 No token issued.
- Deactivated account → HTTP 403 Account suspended.
- Expired refresh token → HTTP 401 Please log in again.

### 3.2 SEQ-02 — User Creation with *.created Event

```
Admin App         Kong GW         UserMgr Svc    IAM Svc      Kafka       DB
     │              │                │             │           │           │
     │  POST /users │                │             │           │           │
     ├─────────────►│───────────────►│             │           │           │
     │              │                │  INSERT person,        │           │
     │              │                │  INSERT academic       │           │
     │              │                ├───────────────────────►           │
     │              │                │  CREATE profile,       │           │
     │              │                │  generate pw hash      │           │
     │              │                │               │ publish "profile.created"  │
     │              │                │               ├──────────────────────►│
     │              │                │               │           │ emit event │
     │              │                │◄───────────────┤◄──────────────────────┤
     │              │                │  profile.id,     │                       │
     │              │                │  initial pw ref  │                       │
     │              │                │◄───────────────┤                       │
     │  { account } │                │               │                       │
     │◄─────────────┴────────────────┤               │                       │
```

**Preconditions:** ADMIN/SUPER_ADMIN authenticated; unique email/document confirmed.
**Postconditions:** Person record created in UserManagement; profile in IAM with auto-generated password. Event published to Kafka.
**Validation:** Email uniqueness, document uniqueness, required fields present, phone format valid.

### 3.3 SEQ-03 — Branch and Course Creation

```
Admin App         Kong GW         SchoolSvc      DB (School)
     │              │                │                │
     │  POST /schools/:id/branches  │                │
     ├─────────────►│───────────────►                │
     │              │                │  INSERT branch │
     │              │                ├───────────────►│
     │              │                │  RETURN branch │
     │              │                │◄───────────────│
     │              │                │                │
     │  POST /courses                 │                │
     ├─────────────►│───────────────►                │
     │              │                │  INSERT course │
     │              │                ├───────────────►│
     │              │                │  RETURN course │
     │              │                │◄───────────────│
     │  { data }    │                │                │
     │◄─────────────┴────────────────┤                │
```

**Preconditions:** Institution branch context established; ADMIN authenticated.
**Postconditions:** Branch/course persisted; visible in school hierarchy.

### 3.4 SEQ-04 — Driver–Bus–Route Assignment

```
Admin App         Kong GW         FleetSvc    RoutesSvc  DB
     │              │                │           │         │
     │  Assign bus  │                │           │         │
     ├─────────────►│───────────────►│           │         │
     │              │                │  UPDATE bus.route_id = ? │
     │              │                ├──────────────────────►│
     │              │                │           │           │
     │  Assign driver│               │           │         │
     ├─────────────►│───────────────►│           │         │
     │              │                │  INSERT driver_bus_assignment │
     │              │                ├──────────────────────►│
     │              │                │           │           │
     │  Publish assignment event    │           │         │
     │              │                ├──────────────────────►│
     │  Confirmation            │           │           │
     │◄─────────────┴───────────────┴───────────┴───────────┘
```

**Preconditions:** Active route exists; bus ACTIVE with no competing assignment; driver ACTIVE.
**Business Rules Enforced:** One bus per route; one driver per bus; both entities ACTIVE.
**Postconditions:** Bus shows on driver app; driver sees route coverage derived from bus assignment.

### 3.5 SEQ-05 — Route Planning and Scheduling

```
Admin App         Kong GW         RoutesSvc   DB    Kafka
     │              │                │          │      │
     │  Create route│                │          │      │
     ├─────────────►│───────────────►│          │      │
     │              │                │  Validate uniqueness │
     │              │                │  INSERT route + stops │
     │              │                ├──────────┤      │
     │              │                │  Return route│      │
     │              │                │◄──────────┤      │
     │  Schedule route instance   │          │      │
     │  for specific day/time    │          │      │
     │              │                ├────────────────────►│
     │              │                │           │      │ publish "route.scheduled"
     │              │                │◄────────────────────┤
     │  Confirmed   │                │           │      │
     │◄─────────────┴────────────────┤           │      │
```

**Preconditions:** Stops ordered; valid schedule times; institution context.
**Postconditions:** Route active in calendar; drivers notified of scheduled instances.

### 3.6 SEQ-06 — Route Execution Start and Completion

```
Driver App        Kong GW         RoutesSvc   DB     Notifications Svc
     │              │                │          │                 │
     │  Start trip  │                │          │                 │
     ├─────────────►│───────────────►│          │                 │
     │              │                │  TRANSITION trip: PLANNED→IN_PROGRESS │
     │              │                ├──────────┤                 │
     │              │                │  SET gps_streaming=true │                 │
     │              │                │◄──────────┤                 │
     │  Trip started│                │          │                 │
     │◄─────────────┴────────────────┤          │                 │
     │              │                │          │                 │
     │  ─── Trip execution (scans, GPS streaming) ────────────────│
     │              │                │          │                 │
     │  Complete trip│              │          │                 │
     ├─────────────►│───────────────►│          │                 │
     │              │                │  TRANSITION trip: IN_PROGRESS→COMPLETED │
     │              │                ├──────────┤                 │
     │              │                │  Generate summary        │                 │
     │              │                ├──────────────────────────►│
     │              │                │  Emit trip.completed event│                 │
     │              │                │◄──────────────────────────┤
     │  Completed   │                │          │                 │
     │◄─────────────┴────────────────┤          │                 │
```

**Preconditions:** Driver authenticated; route assigned for today; bus and GPS available.
**Postconditions:** Trip archived with full event log; all students confirmed dropped off or flagged missing.
**Exception:** Missing student detection at completion → flag for administrator follow-up.

### 3.7 SEQ-07 — Pickup/Drop-off QR Scanning (student.scanned)

```
Driver App        Kong GW         NotifySvc     RoutesSvc   DB      Kafka
     │              │                │             │           │         │
     │  Scan QR     │                │             │           │         │
     ├─────────────►│───────────────►│             │           │         │
     │              │                │  Validate:   │           │         │
     │              │                │  - QR exists │           │         │
     │              │                │  - Student   │           │         │
     │              │                │    assigned  │           │         │
     │              │                │    to route  │           │         │
     │              │                │  - Trip in   │           │         │
     │              │                │    progress  │           │         │
     │              │                │  - Previous  │           │         │
     │              │                │    scan not  │           │         │
     │              │                │    already   │           │         │
     │              │                │    recorded  │           │         │
     │              │                ├──────────────┤           │         │
     │              │                │  INSERT boarding_event│   │
     │              │                ├──────────────┤           │
     │              │                │  Emit student.scanned event │
     │              │                ├────────────────────────────►│
     │              │                │             │           │    │
     │              │                │  Queue notification    │   │
     │              │                ├──────────────────────────►│
     │  Boarded ✓   │             │             │           │    │
     │◄─────────────┴───────────────┴─────────────┴───────────┴────┘
```

**Preconditions:** Valid QR; active trip; student properly assigned.
**Postconditions:** Boarding event persisted; guardian notification queued.
**Exception:** Invalid QR → show clear error on driver app; unassigned student → alert guardian and driver.

### 3.8 SEQ-08 — GPS Position Reception via gRPC

```
GPS Device        Gateway       FleetSvc     gRPC Streamer   DB
     │              │              │             │             │
     │  lat,lng,spd │              │             │             │
     ├──────────────┼──────────────┼────────────►│             │
     │  (every 10s) │              │             │  INSERT position_event │
     │              │              │             ├─────────────┤
     │              │              │             │  RETURN ack  │
     │              │              │             │◄─────────────│
     │              │              │             │             │
     │              │              │  Broadcast to│             │
     │              │              │  subscribers │             │
     │              │              │◄─────────────┤             │
```

**Preconditions:** Driver started trip; GPS transmitting; gRPC channel open.
**Postconditions:** Position persisted; live map updated for subscribed clients.
**Exception:** Position gap > 2min → SignalLostEvent emitted → alert triggered.

### 3.9 SEQ-09 — Alert Generation and Push Notification Delivery

```
NotifySvc         Kafka           Expo          Guardians Apps
     │               │               │                │
     │  Listen for    │               │                │
     │  stationarity  │               │                │
     │  threshold     │               │                │
     │               │               │                │
     │  Threshold     │               │                │
     │  exceeded      │               │                │
     ├───────────────►│               │                │
     │  Emit alert.critical   │               │                │
     │               │               │                │
     │  Build payload:  │               │                │
     │  type, severity, │               │                │
     │  bus_id, route,  │               │                │
     │  timestamp, loc  │               │                │
     ├──────────────────►│                │
     │  Call Expo send    │                │
     │  POST /push/send   │                │
     ├────────────────────────────────────►│                │
     │                                      │  Push received   │
     │                                      │◄─────────────────┤
     │  Delivery status: DELIVERED          │                │
     │◄─────────────────────────────────────│                │
```

**Preconditions:** Stationary detection logic ran; threshold breached; guardians registered with Expo tokens.
**Postconditions:** Alert appears in apps; acknowledged by system.
**Exception:** Delivery failure → retry up to 3x with exponential backoff; after max retries → fallback SMS sent.

---

## 4. Activity Diagrams

### 4.1 ACT-01 — Student Pickup/Drop-off Process with Validations

```
[Start]
   │
   ▼
┌─────────────┐
│ Driver starts│
│ trip execution│
└──────┬───────┘
       │
       ▼
┌──────────────────────────────┐
│ For each stop on route:      │
│   Wait for student arrival   │
└──────┬───────────────────────┘
       │
       ▼
┌─────────────────────┐
│  Student presents   │
│  QR code            │
└──────┬──────────────┘
       │
       ▼
┌──────────────────────────┐
│ System validates:         │
│ • QR exists?              │
│ • Student assigned here?  │
│ • Trip in progress?       │
│ • Previously scanned?     │
└──────┬───────────────────┘
       │
       ├── NO ──► Show error to driver
       │              │
       ▼ YES
┌──────────────────────────┐
│ Record boarding event     │
│ (pickup or drop-off)      │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Send notification to      │
│ linked guardians          │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Continue to next stop?    │
└──────┬───────────────────┘
       │
       ├── YES ──► Loop to next stop
       │
       ▼ NO
┌──────────────────────────┐
│ All students processed?   │
└──────┬───────────────────┘
       │
       ├── SOME MISSING ──► Flag for admin follow-up
       │
       ▼ ALL CONFIRMED
┌──────────────────────────┐
│ Trip complete             │
└──────────────────────────┘
```

**Guard Conditions:**
- Stop-specific: only students assigned to this stop trigger the flow.
- Time window: scans outside ±3 min of schedule may require admin override.
- Duplicate prevention: cooldown period prevents rapid re-scans.

### 4.2 ACT-02 — Alert Handling Process

```
[Alert Triggered]
       │
       ▼
┌──────────────────────────┐
│ Determine alert type &    │
│ severity level            │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Identify affected parties │
│ (guardians, admins, ops)  │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Dispatch notifications    │
│ based on severity         │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Log alert in incident     │
│ tracking system           │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Await resolution          │
│ (manual or automatic)     │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Resolution confirmed?     │
└──────┬───────────────────┘
       │
       ├── NO ──► Escalate per severity tree
       │
       ▼ YES
┌──────────────────────────┐
│ Close alert & record      │
│ resolution details        │
└──────────────────────────┘
```

**Severity Escalation:**
- Low → Admin dashboard notification only.
- Medium → Admin + guardian push notification.
- High → Admin + guardian + operations team.
- Critical → All stakeholders immediately; emergency protocol activated.

### 4.3 ACT-03 — User Creation and Edition with Events

```
[Admin initiates action]
       │
       ▼
┌──────────────────────────┐
│ Action: Create or Edit    │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Input: personal data,     │
│ role, academics (if stu.) │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Validate:                 │
│ • Required fields present │
│ • Email unique            │
│ • Document unique         │
│ • Phone format valid      │
└──────┬───────────────────┘
       │
       ├── FAIL ──► Show errors, retain data
       │
       ▼ PASS
┌──────────────────────────┐
│ Persist person data       │
│ (UserManagement DB)       │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ If create: persist        │
│ profile+role+hash         │
│ (IAM DB); generate        │
│ initial password          │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Publish events:           │
│ • profile.created OR      │
│ • profile.updated         │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Return confirmation +     │
│ initial password ref      │
│ (if creation)             │
└──────────────────────────┘
```

### 4.4 ACT-04 — Exceptional Route or Bus Usage

```
[Situation detected]
(e.g., wrong bus, missed student,
route deviation)
       │
       ▼
┌──────────────────────────┐
│ Driver/admin reports      │
│ via app                   │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ System captures:          │
│ • Type of exception       │
│ • Time, location          │
│ • Affected students       │
│ • Driver/bus identifiers  │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Notify relevant parties   │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Admin investigates        │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Determine outcome:        │
│ • Corrected during trip   │
│ • Requires follow-up      │
│ • Incidents filed         │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Log in exceptional/events │
│ table                     │
└──────────────────────────┘
```

### 4.5 ACT-05 — Critical Actions Audit Procedure

```
[Action performed in system]
(e.g., delete route, deactivate
account, modify critical config)
       │
       ▼
┌──────────────────────────┐
│ System intercepts         │
│ request                   │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Pre-check:                │
│ • Actor authenticated?    │
│ • Role permits action?    │
│ • Data integrity preserved?│
└──────┬───────────────────┘
       │
       ├── DENY ──► Log denial reason, return HTTP 403
       │
       ▼ ALLOW
┌──────────────────────────┐
│ Execute action            │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Audit log entry:          │
│ actor, timestamp, IP,     │
│ application, action,      │
│ target, result            │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Store in immutable audit  │
│ table                     │
└──────────────────────────┘
```

---

## 5. State Machine Diagrams

### 5.1 EST-01 — User Session

```
[Unauthenticated]
       │
       ▼
┌──────────────────────────┐
│ LOGGED_IN                │
│ ─ Token active            │
│ ─ Role claim validated    │
│ ─ Permissions checked     │
└──────┬───────────────────┘
       │
       ├─ Token expires (15m) ─► REFRESHING
       │                             │
       │                    ┌────────┴───────┐
       │                    │ Refresh OK?     │
       │                    ├────────────────┤
       │                    │ YES ─► LOGGED_IN│
       │                    │ NO  ─► UNAUTH  │
       │                    └────────────────┘
       │
       ├─ Logout ─► LOGGED_OUT
       │
       └─ Session invalidated ─► UNAUTHENTICATED

REFRESHING ──► Token renewed ──► LOGGED_IN
       │
       └──► Refresh failed ────► EXPIRED (requires re-login)
```

### 5.2 EST-02 — Route Execution

```
[PLANNED]
   │
   ▼ (driver starts trip)
┌──────────────────────────┐
│ IN_PROGRESS              │
│ ─ GPS streaming active   │
│ ─ Boarding enabled       │
│ ─ Notifications flowing  │
└──────┬───────────────────┘
       │
       ├── All students boarded ──► Monitoring mode
       │                              (tracking + notifications)
       │
       ├── Last student dropped ──► FINALIZING
       │
       └── Emergency/abnormal ──► ABNORMAL_TERMINATION
                                     │
                                     ▼
                               ┌──────────────────────────┐
                               │ COMPLETED                 │
                               │ ─ Summary generated       │
                               │ ─ All events logged       │
                               └──────────────────────────┘

ABNORMAL_TERMINATION ──► Manual review ──► Marked as COMPLETED_WITH_EXCEPTION
```

### 5.3 EST-03 — Alert

```
[Generated]
   │
   ▼
┌──────────────────────────┐
│ DISPATCHED               │
│ ─ Push sent to parties   │
│ ─ Dashboard populated    │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ ACKNOWLEDGED             │
│ ─ Party opened/did read  │
│ ─ Or manual mark          │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ RESOLVED                 │
│ ─ Root cause identified  │
│ ─ Action taken            │
└──────────────────────────┘
```

### 5.4 EST-04 — Student Trip (Boarding Event)

```
[PENDING]
(Scheduled trip for today)
   │
   ▼ (trip starts)
┌──────────────────────────┐
│ BOARDING_WINDOW_OPEN      │
│ ─ Pickups possible       │
│ ─ Drop-offs not allowed  │
└──────┬───────────────────┘
       │
       ▼ (all pickups done, route in progress)
┌──────────────────────────┐
│ IN_ROUTE                 │
│ ─ GPS streaming          │
│ ─ Location tracked       │
│ ─ Notifications active   │
└──────┬───────────────────┘
       │
       ▼ (arrival at final stop)
┌──────────────────────────┐
│ BOARDING_WINDOW_CLOSE     │
│ ─ Drop-offs possible     │
│ ─ Pickups closed          │
└──────┬───────────────────┘
       │
       ▼ (last student alighted)
┌──────────────────────────┐
│ TRIP_COMPLETE             │
│ ─ Full event log exists  │
│ ─ History immutable       │
└──────────────────────────┘
```

### 5.5 EST-05 — GPS Device

```
[OFFLINE]
   │
   ▼ (powered on, connected)
┌──────────────────────────┐
│ CONNECTED                │
│ ─ Signal acquired        │
│ ─ Positions flowing      │
└──────┬───────────────────┘
       │
       ├── Signal lost (>2min) ──► OFFLINE
       │                              │
       │                        ┌─────┴──────┐
       │                        │ Retry conn? │
       │                        ├────────────┤
       │                        │ YES ─► CONN │
       │                        │ NO  ─► OFFL │
       │                        └────────────┘
       │
       └── Battery low ──► LOW_POWER
                               │
                               ▼ LOW_POWER ──► CRITICAL_BATTERY ──► OFFLINE
```

### 5.6 EST-06 — Access Profile (ACTIVE / INACTIVE)

```
[AWAITING_ACTIVATION]
(Account being set up)
   │
   ▼ (activated by admin or auto)
┌──────────────────────────┐
│ ACTIVE                   │
│ ─ Can authenticate       │
│ ─ Can access permitted   │
│   resources              │
│ ─ Can receive            │
│   notifications          │
└──────┬───────────────────┘
       │
       └── Deactivated by admin ──► INACTIVE
                                         │
                                         ├── Reactivated ──► ACTIVE
                                         │
                                         └── Permanently disabled ──► TERMINATED
                                                                                    │
                                                                                    ▼
                                                                             [Cannot reactivate;
                                                                              historical data preserved]
```

---

## 6. Dynamic Traceability ↔ FR/US

### 6.1 Coverage by Requirements Group

| FR Group | Related SEQs | Related ACTs | Related ESTs | Notes |
| --- | --- | --- | --- | --- |
| RF-01 (User Mgmt) | SEQ-02, SEQ-03, SEQ-04 | ACT-03 | EST-06 | Account lifecycle fully modeled |
| RF-02 (Auth) | SEQ-01 | — | EST-01 | Session management captured |
| RF-03 (Routes/Fleet) | SEQ-04, SEQ-05, SEQ-06 | ACT-04 | EST-02 | Trip lifecycle end-to-end |
| RF-04 (QR Scanning) | SEQ-07 | ACT-01 | EST-04 | Boarding flow fully specified |
| RF-05 (Notifications) | SEQ-09 | ACT-02 | EST-03 | Alert lifecycle complete |
| RF-06 (GPS Tracking) | SEQ-08 | ACT-05 | EST-05 | Telemetry chain verified |
| RF-07 (Trip Mgmt) | SEQ-06 | ACT-01, ACT-04 | EST-02, EST-04 | Trip states covered |
| RF-08 (Incidents) | SEQ-09 | ACT-02, ACT-04 | EST-03 | Incident handling traced |
| RF-09 (Route Inquiry) | SEQ-07 (read side) | — | — | Read-only query paths implicit |
| RF-10 (Personalization) | — | — | — | Affects UI layer only; no backend dynamics |

### 6.2 Traceability Observations

| Observation | Impact | Action |
| --- | --- | --- |
| RF-10 does not appear in any sequence diagram | Personalization affects presentation only; backend contracts unchanged. | Ensure UI components implement theme/locale switching without affecting service calls. |
| SEQ-01 covers both initial login and token refresh | Single diagram handles both authn and authz renewal paths. | Verify JWT RS256 signing key rotation strategy documented separately. |
| SEQ-07 is the most critical path (real-time safety) | Any failure impacts parent trust and student safety. | Implement idempotency keys on QR scan endpoint to prevent double-charging or duplicate notifications. |
| EST-04 Boarding window enforces temporal ordering | Ensures pickups complete before drop-offs begin. | Test boundary conditions: early arrivals, late departures, and mid-route student additions. |
| SEQ-08 gRPC stream has no explicit termination | Implies continuous connection until trip ends or device disconnects. | Add heartbeat mechanism; detect stale connections and clean up resources. |
| ACT-05 Audit trail is separate from business flows | Audit acts as cross-cutting concern independent of primary transactions. | Verify audit records cannot be modified or deleted (WORM storage recommended). |

---

*Document end — Dynamic Views Analysis for Guardian Escolar.*
