# Software Design Document — Guardian Escolar

---

## Table of Contents

- [1. Introduction and Design Principles](#1-introduction-and-design-principles)
  - [1.1 Purpose and Scope](#11-purpose-and-scope)
  - [1.2 Design Principles](#12-design-principles)
  - [1.3 Design Conventions](#13-design-conventions)
- [2. Architectural Model](#2-architectural-model)
  - [2.1 Context View](#21-context-view)
  - [2.2 Container View](#22-container-view)
  - [2.3 Deployment View](#23-deployment-view)
  - [2.4 Internal Layer View of a Service](#24-internal-layer-view-of-a-service)
  - [2.5 Adopted Architectural Styles and Justification](#25-adopted-architectural-styles-and-justification)
  - [2.6 Cross-Cutting Concerns](#26-cross-cutting-concerns)
- [3. Applied Design Patterns](#3-applied-design-patterns)
  - [3.1 Verified Patterns](#31-verified-patterns)
  - [3.2 Evaluated Patterns Not Adopted](#32-evaluated-patterns-not-adopted)
- [4. User Interfaces](#4-user-interfaces)
  - [4.1 Interface Principles](#41-interface-principles)
  - [4.2 Web Interfaces by Role](#42-web-interfaces-by-role)
  - [4.3 Mobile Interfaces](#43-mobile-interfaces)
  - [4.4 Navigation Map](#44-navigation-map)
  - [4.5 Design Tokens](#45-design-tokens)
  - [4.6 Interface States](#46-interface-states)
  - [4.7 Accessibility and Adaptation](#47-accessibility-and-adaptation)
- [5. Data Model](#5-data-model)
  - [5.1 Global ER Diagram](#51-global-er-diagram)
  - [5.2 IAM Schema](#52-iam-schema)
  - [5.3 UserManagement Schema](#53-usermanagement-schema)
  - [5.4 School Schema](#54-school-schema)
  - [5.5 Fleet Schema](#55-fleet-schema)
  - [5.6 Route Schema](#56-route-schema)
  - [5.7 GPS Schema](#57-gps-schema)
  - [5.8 Notification Schema](#58-notification-schema)
  - [5.9 Exceptional Schema](#59-exceptional-schema)
  - [5.10 Settings Schema](#510-settings-schema)

---

## 1. Introduction and Design Principles

### 1.1 Purpose and Scope

This document specifies the technical design decisions for Guardian Escolar, transforming requirements from the SRS into implementable architecture, service boundaries, interface specifications, and data schemas. It serves as the authoritative reference for developers designing components, selecting algorithms, implementing business logic, and ensuring cross-cutting concerns (security, audit, observability) are consistently applied.

**In scope:** System architecture at container level, internal layer patterns per service, API contracts, data schemas with relationships, UI screen specifications, navigation flows, and design token system.

**Out of scope:** Hardware specifications for GPS devices (external procurement), third-party provider integration details (Expo push, SMS providers), detailed network cabling or ISP configuration, legal/contractual documents.

### 1.2 Design Principles

| Principle | Guideline | Enforcement Mechanism |
| --- | --- | --- |
| Bounded Context Ownership | Each microservice owns its database schema; no cross-service direct DB queries | Code review + architecture decision record (ADR) |
| Hexagonal Architecture | Services expose ports (interfaces) implemented by adapters (DB, external APIs) | Dependency rule: domain → application → interfaces ← infrastructure |
| Contract-First APIs | OpenAPI/Swagger spec published before service implementation starts | CI pipeline validates all endpoints match contract |
| Eventual Consistency | Cross-service state changes communicated via Kafka events; compensating actions for failures | Saga pattern for distributed transactions |
| Fail-Fast Validation | Input validation occurs at API entry point; reject invalid requests immediately | Validations in DTO/request validators |
| Immutable Audit Trail | Critical actions logged permanently with full contextual metadata | Audit interceptor on controller layer |
| Offline-First Mobile | Core mobile flows functional with intermittent connectivity; queue-and-sync | Local SQLite storage on device; background sync when connected |
| Least Privilege RBAC | Every role granted minimum permissions required; checked at every boundary | Gateway JWT claims → service permission middleware |

### 1.3 Design Conventions

| Element | Convention | Example |
| --- | --- | --- |
| Package structure (Java) | `com.guardianscolar.{context}.{layer}.{entity}` | `com.guardianscolar.fleet.model.Bus` |
| Package structure (C#) | `GuardianScholar.{Context}.Infrastructure.{Layer}` | `GuardianScholar.Fleet.Persistence.EfCore` |
| API path prefix | `/api/v1/{resource}` | `/api/v1/routes`, `/api/v1/users` |
| HTTP status codes | Standard REST semantics (200, 201, 400, 401, 403, 404, 500) | POST creating → 201 Created |
| Error response format | `{"error":{"code":"VALIDATION_ERROR","message":"...","details":[]}}` | Consistent across all services |
| Date/time serialization | ISO 8601 UTC timestamps everywhere | `"2026-03-15T14:30:00Z"` |
| Database naming | snake_case tables, camelCase columns (per EF conventions) | `bus_assignments`, `licensePlate` |
| Microservice naming | `ms-{context}` | `ms-fleet`, `ms-route`, `ms-notification` |

---

## 2. Architectural Model

### 2.1 Context View

```
┌───────────────────┐         ┌──────────────────────┐
│   Parent App      │         │     Admin Web App    │
│   (React + Expo)  │         │   (Angular 21)       │
└─────────┬─────────┘         └──────────┬───────────┘
          │ HTTPS                         │ HTTPS
          └──────────┬────────────────────┘
                     │
          ┌──────────▼──────────┐
          │ Kong API Gateway    │
          │ :8000               │
          └──────────┬──────────┘
                     │ REST :8080
    ┌────────────────┼───────────────────────┐
    ▼                ▼                       ▼
┌────────┐   ┌────────────┐        ┌────────────────┐
│ IAM    │   │ Routes SVC │        │ Notifications  │
│(Java)  │   │ (.NET)     │        │ SVC (.NET)     │
└───┬────┘   └─────┬──────┘        └────────┬───────┘
    │              │                        │
    ▼              ▼                        ▼
[SQL Server: Iam] [SQL Server: Route]  [Kafka Broker :9092]
                                                    │ gRPC :5001
                                          ┌────────▼────────┐
                                          │ GPS Device      │
                                          │ (on bus)        │
                                          └─────────────────┘
```

External systems: Expo Push Service, SMS Provider (optional), Traffic API (future enhancement).

### 2.2 Container View

```
                    ┌─────────────────────────────────┐
                    │         Docker Compose           │
                    │                                  │
                    │  ┌────┐ ┌────┐ ┌────┐ ┌────┐   │
                    │  │IAM │ │UMgr│ │School│ │Fleet│   │
                    │  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘   │
                    │     │       │       │       │       │
                    │  ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐   │
                    │  │Route│ │Notif│ │Audit│ │Sett.│   │
                    │  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘   │
                    │     │       │       │       │       │
                    │  ┌──┴───────────────────────────┴───┐│
                    │  │         SQL Server :1433          ││
                    │  │   (Iam, UMgr, School, Fleet,      ││
                    │  │    Route, Notif, Audit, Sett.sch)  ││
                    │  └───────────────────────────────────┘│
                    │  ┌───────────────────────────────────┐│
                    │  │         Kafka :9092                ││
                    │  └───────────────────────────────────┘│
                    │  ┌───────────────────────────────────┐│
                    │  │         Redis :6379                ││
                    │  └───────────────────────────────────┘│
                    │  ┌───────────────────────────────────┐│
                    │  │    Kong API Gateway :8000          ││
                    │  └───────────────────────────────────┘│
                    └────────────────────────────────────────┘
```

Web and mobile clients run outside Docker on user devices. Production deployment targets Kubernetes with similar topology but spread across nodes/namespaces.

### 2.3 Deployment View

```
Production / Cloud (planned)
┌────────────────────────────────────────────────────────────┐
│                      Load Balancer                         │
│                           │                                │
│                           ▼                                │
│              ┌───────────────────────┐                     │
│              │   Kong Cluster (HA)    │                     │
│              │   :8000 / :8443        │                     │
│              └───────┬───────────────┘                     │
│                      │ REST :8080                          │
│        ┌─────────────┼─────────────┐                       │
│        ▼             ▼             ▼                       │
│   ┌─────────┐ ┌──────────┐ ┌─────────────┐               │
│   │ IAM Pod │ │Routes Pod│ │NotificationPod│               │
│   │ x2      │ │ x2       │ │ x2           │               │
│   └────┬────┘ └────┬─────┘ └──────┬──────┘               │
│        │           │              │                        │
│   ┌────▼───────────▼──────────────▼─────────────────┐    │
│   │          Managed SQL Server Instance             │    │
│   │          Single instance, per-schema DBs         │    │
│   └─────────────────────────────────────────────────┘    │
│   ┌─────────────────────────────────────────────────┐    │
│   │          Managed Kafka Cluster (3 brokers)       │    │
│   └─────────────────────────────────────────────────┘    │
│   ┌─────────────────────────────────────────────────┐    │
│   │          Managed Redis (primary + replica)       │    │
│   └─────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────┘
```

Local development uses Docker Compose on developer machines with single-instance databases and unencrypted TCP.

### 2.4 Internal Layer View of a Service

Every microservice follows hexagonal architecture with four layers:

```
┌──────────────────────────────────────────────────────────┐
│                   PORTS (Interfaces)                      │
│  • Application Port: Use case interfaces                  │
│  • Driving Port: API controllers, event subscribers       │
│  • Supporting Port: Repository interfaces                 │
├──────────────────────────────────────────────────────────┤
│                  APPLICATION LAYER                         │
│  • Domain logic orchestration                             │
│  • Transaction boundaries                                 │
│  • DTO conversion                                         │
├──────────────────────────────────────────────────────────┤
│                     DOMAIN LAYER                           │
│  • Entities, Value Objects                                │
│  • Business rules / validation                            │
│  • Domain events                                          │
├──────────────────────────────────────────────────────────┤
│                   ADAPTERS (Infrastructure)                │
│  • Entity Framework / JPA (persistence)                   │
│  • Kafka producer/consumer                                │
│  • HTTP client for external APIs                          │
│  • gRPC server/client                                   │
└──────────────────────────────────────────────────────────┘
```

Dependency direction: **Adapters → Ports → Application → Domain**. Domain layer has zero dependencies on frameworks or infrastructure.

### 2.5 Adopted Architectural Styles and Justification

| Style | Adoption | Rationale |
| --- | --- | --- |
| Microservices | Yes | Independent deployability; bounded context alignment with business capabilities |
| Hexagonal (Ports & Adapters) | Yes | Testability; framework independence; clear separation of domain vs infrastructure |
| CQRS (Read/Write Separation) | Partially | Write side uses command pattern; read side currently shares same database. Read replicas planned. |
| Event Sourcing | No | Overcomplicates for current data volume and consistency requirements |
| Serverless | No | Cold start latency unacceptable for real-time flows; fixed cost more economical than pay-per-invocation at scale |
| Monolith (modular) | Considered | Rejected — team size and domain complexity justify service boundaries from day one |
| GraphQL | No | REST sufficient for well-defined CRUD + event-driven updates; simpler tooling |

### 2.6 Cross-Cutting Concerns

| Concern | Implementation | Scope |
| --- | --- | --- |
| Authentication | JWT RS256 validated at Kong gateway; tokens forwarded to services | All API calls |
| Authorization | Role claims extracted from JWT; checked in service middleware | All API calls |
| Logging | Structured JSON logs via Serilog (C#) / Logback (Java) | All services |
| Metrics | Prometheus metrics exposed per service | Health check endpoint |
| Tracing | OpenTelemetry SDK; trace IDs propagated via headers | Cross-service request chains |
| Retry/Failover | Circuit breaker (Resilience4j .NET / Spring Retry Java); exponential backoff | External API calls |
| Configuration | Environment variables; optional config file mounted as volume | All containers |
| Secrets | Vault injection or env vars; never committed to repo | All containers |

---

## 3. Applied Design Patterns

### 3.1 Verified Patterns

| Pattern | Where Used | Purpose |
| --- | --- | --- |
| Repository | Data access layer of each service | Abstraction over persistence technology |
| Factory | Creating aggregates/entities with validation | Ensures consistent creation rules |
| Builder | Complex DTO construction (route creation request) | Readable construction of multi-field objects |
| Strategy | Authentication strategies (password, JWT refresh) | Swappable authentication implementations |
| Observer / Events | Domain event publishing via Kafka | Loose coupling between services |
| Saga | Distributed transaction for route assignment (update route + fleet simultaneously) | Coordinate cross-service operations with compensation |
| Circuit Breaker | Calls to external services (SMS, push notification) | Prevent cascade failure |
| Decorator | Adding authorization/audit wrappers around handlers | Composable cross-cutting behavior |
| Middleware Pipeline | Kong plugins, Spring Interceptors, ASP.NET Middlewares | Request/response processing chain |

### 3.2 Evaluated Patterns Not Adopted

| Pattern | Reason for Rejection | Future Consideration |
| --- | --- | --- |
| CQRS Full | Add maintenance complexity without clear read/write ratio justification yet | If read load significantly exceeds writes, introduce read models |
| Event Sourcing | Current requirement for mutable entities (account updates, profile edits) conflicts with immutable event log principle | Consider for audit-only append-only streams |
| Message Bus (NServiceBus/MassTransit) | Kafka already handles event distribution; adding another abstraction adds unnecessary indirection | Re-evaluate if Kafka migration pain emerges |
| Dependency Injection Containers (beyond built-in) | Built-in DI frameworks (Spring IoC, .NET DI) sufficient for project scale | No additional container needed |
| ORM-less Raw SQL | EF Core / JPA provide adequate performance and safety for current query patterns | Profile queries; use raw SQL only for bottlenecks |

---

## 4. User Interfaces

### 4.1 Interface Principles

1. **Role-Based Views**: Each role sees only relevant screens and actions; no navigation to unauthorized pages.
2. **Progressive Disclosure**: Simple views first; advanced options expandable via accordion/toggle.
3. **Consistent Feedback**: Every action shows immediate result (success toast, error banner, loading spinner).
4. **Accessible by Default**: WCAG AA contrast ratios; keyboard navigable; screen reader labels.
5. **Responsive Layout**: Adapts fluidly from phone (360px width) through tablet (768px) to desktop (≥1200px).
6. **Offline Tolerant**: Mobile app caches critical data locally; retries failed operations when connectivity restored.

### 4.2 Web Interfaces by Role

#### 4.2.1 Public Space (Unauthenticated)

| Screen | Access | Purpose |
| --- | --- | --- |
| Login | Anyone | Enter credentials to authenticate |
| Forgot Password | Anyone | Initiate password reset via email/SMS |
| Registration Redirect | Anyone | Direct to "Contact your administrator" message (no self-registration) |

#### 4.2.2 SUPER_ADMIN Role

| Screen | Purpose | Key Components |
| --- | --- | --- |
| Dashboard | Platform-wide overview | Institution count, active trips, incidents today, system health |
| Institution Management | CRUD institutions | Search table, create/edit modal, activate/deactivate toggle |
| Global Configuration | Platform settings | Theme defaults, rate limit thresholds, notification templates |
| Platform Audit |查看所有审计记录 | Filterable log table: date range, actor, action type, outcome |
| User Management | Create admin accounts | Form with role selection, credential generation |

#### 4.2.3 ADMIN Role (School Administrator)

| Screen | Purpose | Key Components |
| --- | --- | --- |
| Dashboard | Institution overview | Active trips count, today's routes summary, recent incidents |
| Schools/Courses | Manage branches and courses | Tree view: institution → branches → courses → grades |
| People | Person management | Search table: filter by role (students/drivers/parents); CRUD forms |
| Fleet | Bus administration | Grid showing plate, capacity, status; assign/unassign dialogs |
| Routes | Route configuration | Interactive map showing stops; ordered list of stops; schedule editor |
| Assignments | Bus-driver-student assignments | Three-level selector: route → bus → driver → students per stop |
| Reports | Generated reports | Export to PDF/CSV: trip summaries, boarding statistics, incident log |
| Notifications Center | Review sent alerts | Timeline of dispatched notifications with delivery status |

#### 4.2.4 Limited Access Profiles (Driver, Parent, Student)

These profiles primarily use the mobile application. Web fallback screens include:
- Driver: Route plan view, incident report form
- Parent: Live map view, boarding history, notification preferences
- Student: QR code display page (read-only)

#### 4.2.5 Component and Validation Matrix by Form

| Form Field | Type | Required | Validation Rule | Error Message |
| --- | --- | --- | --- | --- |
| Email | Text | Yes | RFC 5322 format; unique | "Email in use" |
| Phone | Text | Yes | 10+ digits; valid Colombian format | "Phone must be 10+ digits" |
| Document Number | Text | Yes | Length/format by type (CC, TI, CE) | "Invalid document format" |
| Name | Text | Yes | Non-empty; ≤ 120 chars | "Name is required" |
| Date of Birth | Date | Yes | Age ≥ 5 years old; ≤ 120 years | "Date out of valid range" |
| Role | Select | Yes | Enum values only | "Select a role" |
| Route Stop Order | Number | Yes | Positive integer; unique within route | "Stop order must be unique" |
| Bus License Plate | Text | Yes | Alphanumeric; unique in fleet | "Plate already exists" |
| Bus Capacity | Number | Yes | Integer > 0 | "Capacity must be positive" |

### 4.3 Mobile Interfaces

| Screen | Role(s) | Purpose | Key Features |
| --- | --- | --- | --- |
| Login | All | Authenticate | Email/password; remember device; biometric (future) |
| Driver – Route Plan | DRIVER | Today's route | Stop list with times, passenger count per stop, scan QR button |
| Driver – QR Scanner | DRIVER | Capture student QR | Camera preview; scan animation; success/error feedback |
| Driver – Trip Start | DRIVER | Begin route execution | Confirm bus assigned; GPS signal check; submit |
| Driver – Alert View | DRIVER | View received alerts | Push notification deep link opens alert detail |
| Driver – Incident Report | DRIVER | Report exceptions | Predefined types, severity selector, free text, photo attachment |
| Parent – Live Map | PARENT | Track bus position | Google Maps/OpenStreetMap view; moving marker; ETA estimate |
| Parent – Notifications | PARENT | Boarding/drop-off alerts | List of recent notifications with timestamps |
| Parent – Route Detail | PARENT | View assigned route | Stops list, pickup/drop-off times, bus info, driver name |
| Parent – Trip History | PARENT | Historical boardings | Calendar view; tap date for daily summary |
| Student – QR Display | STUDENT | Show identification QR | Large QR code; auto-refresh every 60 seconds |

### 4.4 Navigation Map

```
Web (Admin/SuperAdmin)
┌────────────── Sidebar Navigation ───────────────────────────┐
│                                                              │
│  📊 Dashboard                                                │
│  👥 People                                                   │
│      ├ Students                                              │
│      ├ Drivers                                               │
│      └ Parents                                                │
│  🏫 Schools & Courses                                        │
│  🚌 Fleet                                                    │
│  🗺️ Routes                                                  │
│      ├ Schedule Manager                                      │
│      └ Assignment Wizard                                     │
│  ⚙️ Settings                                                 │
│  📋 Audit Log                                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Mobile (Driver)
┌──────────── Bottom Tab Bar ──────────────────────────────┐
│                                                              │
│  🗺️ Route    📷 Scan    🚨 Report    👤 Profile            │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Mobile (Parent/Guardian)
┌──────────── Bottom Tab Bar ──────────────────────────────┐
│                                                              │
│  🗺️ Track    🔔 Alerts    🚌 Routes    👤 Profile           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 4.5 Design Tokens

Color palette (brand-derived):

| Token | HEX | Usage |
| --- | --- | --- |
| `--color-primary` | `#1976D2` | Primary buttons, active states, links |
| `--color-secondary` | `#FFA000` | Secondary actions, warnings, highlights |
| `--color-success` | `#388E3C` | Confirmation messages, success states |
| `--color-warning` | `#F57C00` | Warning banners, caution indicators |
| `--color-danger` | `#D32F2F` | Errors, destructive actions, critical alerts |
| `--color-info` | `#0288D1` | Informational banners, tooltips |
| `--color-background` | `#FAFAFA` | Page backgrounds |
| `--color-surface` | `#FFFFFF` | Card surfaces, modals |
| `--color-text-primary` | `#212121` | Primary text, headings |
| `--color-text-secondary` | `#757575` | Secondary text, captions |
| `--color-border` | `#E0E0E0` | Dividers, input borders |

Typography:

| Token | Font Family | Size | Weight | Usage |
| --- | --- | --- | --- | --- |
| `--font-display` | Roboto | 24–32px | Bold 700 | Page titles |
| `--font-body` | Roboto | 14–16px | Regular 400 | Body text |
| `--font-caption` | Roboto | 12px | Medium 500 | Labels, dates |
| `--font-code` | Fira Mono | 13px | Regular 400 | Code blocks, IDs |

Spacing scale (4px base):

`--spacing-xs: 4px`; `--spacing-sm: 8px`; `--spacing-md: 16px`; `--spacing-lg: 24px`; `--spacing-xl: 32px`; `--spacing-xxl: 48px`.

Border radius: `--radius-sm: 4px`; `--radius-md: 8px`; `--radius-lg: 12px`; `--radius-full: 9999px`.

Shadows: `--shadow-sm: 0 1px 3px rgba(0,0,0,0.12)`; `--shadow-md: 0 4px 6px rgba(0,0,0,0.1)`; `--shadow-lg: 0 10px 25px rgba(0,0,0,0.15)`.

### 4.6 Interface States

| State | Visual Treatment | Examples |
| --- | --- | --- |
| Idle | Standard appearance | Initial page load; form ready |
| Loading | Skeleton screens or spinners | Data fetching; submission in progress |
| Success | Green confirmation banner/toast | Account created; route saved |
| Error | Red banner with specific message | Validation failure; server error |
| Empty | Illustration + instructional text | No routes defined; no notifications |
| Disabled | Grayed out; tooltip explaining reason | Submit button while pending; inactive bus shown dimmed |
| Warning | Amber/yellow banner | Signal lost; approaching threshold |

### 4.7 Accessibility and Adaptation

| Requirement | Implementation |
| --- | --- |
| WCAG AA Contrast | All text meets ≥ 4.5:1 contrast ratio against background |
| Keyboard Navigation | All interactive elements reachable via Tab; visible focus indicator |
| Screen Reader | ARIA labels on icons; semantic HTML landmarks (`<main>`, `<nav>`, `<header>`) |
| Reduced Motion | prefers-reduced-motion media query respected; animations disabled |
| High Contrast Mode | OS-level high contrast theme detected; adapted styles applied |
| Responsive Breakpoints | 360px (phone portrait), 768px (tablet), 1024px (small desktop), 1440px (large desktop) |

---

## 5. Data Model

### 5.1 Global ER Diagram

The database uses a single SQL Server instance with nine schemas, each owned by its corresponding microservice:

```
Schema:Iam ────FK───► Schema:UserManagement (profileId references persons.id)
Schema:Fleet ──FK───► Schema:Route (busId in route_assignments)
Schema:Route ───FK───► Schema:Fleet (driverId in driver_assignments)
Schema:Fleet ───FK───► Schema:Geolocation (deviceId FK in position_events)
Schema:Notifications ──publishes──► Kafka topics consumed by other services
Schema:Audit ────append-only───► Records all cross-service critical actions
```

### 5.2 IAM Schema — 5 Tables

**profiles**
| Column | Type | Constraints | Description |
| --- | --- | --- | --- |
| id | UUID | PK, default NEWID() | Unique profile identifier |
| person_id | INT | FK → persons.id, NOT NULL | Link to person record |
| role | VARCHAR(32) | CHECK IN ('SUPER_ADMIN','ADMIN','DRIVER','PARENT','STUDENT'), NOT NULL | Role enumeration |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Login credential |
| password_hash | VARCHAR(255) | NOT NULL | bcrypt hash |
| is_active | BIT | DEFAULT 1 | Soft-delete flag |
| created_at | DATETIME2 | DEFAULT GETUTCDATE() | Creation timestamp |
| updated_at | DATETIME2 | NULL | Last modification timestamp |

**roles** (lookup/reference table)
| Column | Type | Constraints |
| --- | --- | --- |
| id | VARCHAR(32) | PK (matches enum values) |
| name | VARCHAR(64) | UNIQUE, NOT NULL |
| description | TEXT | NULL |

**credentials**
| Column | Type | Constraints |
| --- | --- | --- |
| id | BIGINT | PK, IDENTITY |
| profile_id | UUID | FK → profiles.id, NOT NULL, CASCADE DELETE |
| attempt_count | INT | DEFAULT 0 |
| locked_until | DATETIME2 | NULL |
| last_attempt_at | DATETIME2 | NULL |

**sessions**
| Column | Type | Constraints |
| --- | --- | --- |
| id | UUID | PK |
| profile_id | UUID | FK → profiles.id, NOT NULL |
| ip_address | VARCHAR(45) | IPv6-capable |
| user_agent | VARCHAR(512) | Client browser/app info |
| issued_at | DATETIME2 | Session start time |
| expires_at | DATETIME2 | JWT expiry window |

**refresh_tokens**
| Column | Type | Constraints |
| --- | --- | --- |
| id | UUID | PK |
| profile_id | UUID | FK → profiles.id, NOT NULL |
| token_hash | VARCHAR(255) | Hashed storage (never plain text) |
| expires_at | DATETIME2 | Token lifetime end |
| revoked_at | DATETIME2 | NULL (set when rotated/expired) |

### 5.3 UserManagement Schema — 4 Tables

**persons**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| document_type | VARCHAR(10) | CC, TI, CE pasaporto |
| document_number | VARCHAR(30) | UNIQUE, NOT NULL |
| first_name | VARCHAR(100) | NOT NULL |
| middle_name | VARCHAR(100) | NULL |
| last_name | VARCHAR(100) | NOT NULL |
| gender | VARCHAR(10) | NULL |
| birth_date | DATE | NULL |
| email | VARCHAR(255) | UNIQUE |
| phone | VARCHAR(20) | NULL |
| address | TEXT | NULL |
| is_active | BIT | DEFAULT 1 |
| created_at | DATETIME2 | DEFAULT GETUTCDATE() |

**academic_records**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| person_id | INT | FK → persons.id (student only) |
| branch_id | INT | FK → branches.id |
| course_code | VARCHAR(20) | NULL |
| grade_level | INT | NULL |
| enrollment_year | INT | NULL |

**guardian_links**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| guardian_person_id | INT | FK → persons.id (role=PARENT) |
| student_person_id | INT | FK → persons.id (role=STUDENT) |
| relationship_type | VARCHAR(50) | Father, Mother, Legal Guardian, Other |
| is_primary | BIT | DEFAULT 0; only one primary per student |
| effective_from | DATE | DEFAULT CURRENT_DATE |
| effective_to | DATE | NULL |

**notifications_preferences**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| person_id | INT | FK → persons.id |
| push_enabled | BIT | DEFAULT 1 |
| sms_enabled | BIT | DEFAULT 0 |
| email_enabled | BIT | DEFAULT 1 |
| updated_at | DATETIME2 | DEFAULT GETUTCDATE() |

### 5.4 School Schema — 4 Tables

**institutions**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| name | VARCHAR(200) | UNIQUE, NOT NULL |
| nit | VARCHAR(20) | Tax ID; UNIQUE |
| legal_representative | VARCHAR(200) | NULL |
| contact_email | VARCHAR(255) | NULL |
| contact_phone | VARCHAR(20) | NULL |
| address | TEXT | NULL |
| city | VARCHAR(100) | DEFAULT 'Neiva' |
| department | VARCHAR(100) | DEFAULT 'Huila' |
| country | VARCHAR(50) | DEFAULT 'Colombia' |
| is_active | BIT | DEFAULT 1 |

**branches**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| institution_id | INT | FK → institutions.id, NOT NULL |
| name | VARCHAR(200) | NOT NULL |
| code | VARCHAR(20) | UNIQUE within institution |
| address | TEXT | NULL |
| is_central | BIT | DEFAULT 0; one central campus per institution |
| latitude | DECIMAL(9,6) | NULL |
| longitude | DECIMAL(9,6) | NULL |

**courses**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| branch_id | INT | FK → branches.id, NOT NULL |
| code | VARCHAR(20) | UNIQUE within branch |
| name | VARCHAR(100) | NOT NULL |
| grade_level | INT | NULL |
| capacity | INT | NULL |

**grades**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| course_id | INT | FK → courses.id, NOT NULL |
| name | VARCHAR(50) | NOT NULL |
| semester | INT | NULL |

### 5.5 Fleet Schema — 4 Tables

**buses**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| institution_id | INT | FK → institutions.id, NOT NULL |
| license_plate | VARCHAR(20) | UNIQUE, NOT NULL |
| brand | VARCHAR(100) | NULL |
| model | VARCHAR(100) | NULL |
| year | SMALLINT | NULL |
| capacity | INT | CHECK > 0, NOT NULL |
| soat_expiry | DATE | NULL |
| tecnomecanica_expiry | DATE | NULL |
| is_active | BIT | DEFAULT 1 |
| created_at | DATETIME2 | DEFAULT GETUTCDATE() |

**gps_devices**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| bus_id | INT | FK → buses.id; UNIQUE; ON DELETE SET NULL |
| serial_number | VARCHAR(50) | UNIQUE, NOT NULL |
| imei | VARCHAR(20) | NULL |
| manufacturer | VARCHAR(100) | NULL |
| last_position_update | DATETIME2 | NULL |
| battery_level | INT | 0–100; NULL until first report |
| status | VARCHAR(20) | ONLINE/OFFLINE/LOW_BATTERY/CRITICAL |

**vehicle_maintenance**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| bus_id | INT | FK → buses.id, NOT NULL |
| maintenance_date | DATE | NOT NULL |
| type | VARCHAR(50) | Oil change, Tire rotation, Brake inspection, Other |
| description | TEXT | NULL |
| cost | DECIMAL(10,2) | NULL |
| provider | VARCHAR(200) | NULL |

**driver_bus_assignment**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| driver_person_id | INT | FK → persons.id, NOT NULL |
| bus_id | INT | FK → buses.id, NOT NULL |
| assigned_at | DATETIME2 | DEFAULT GETUTCDATE() |
| assigned_by | UUID | FK → sessions.id; who made this assignment |
| is_active | BIT | DEFAULT 1; soft delete pattern |

### 5.6 Route Schema — 7 Tables

**routes**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| institution_id | INT | FK → institutions.id, NOT NULL |
| name | VARCHAR(100) | UNIQUE within institution, NOT NULL |
| sector | VARCHAR(100) | Geographic zone reference |
| total_distance_km | DECIMAL(8,2) | NULL |
| estimated_duration_min | INT | NULL |
| direction | VARCHAR(50) | TO_SCHOOL / FROM_SCHOOL |
| is_active | BIT | DEFAULT 1 |
| created_at | DATETIME2 | DEFAULT GETUTCDATE() |

**stops**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| route_id | INT | FK → routes.id, NOT NULL |
| stop_sequence | INT | Ordered position within route; UNIQUE within route, NOT NULL |
| name | VARCHAR(100) | Descriptive name |
| address | TEXT | NULL |
| latitude | DECIMAL(9,6) | NULL |
| longitude | DECIMAL(9,6) | NULL |

**route_schedules**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| route_id | INT | FK → routes.id, NOT NULL |
| date | DATE | NOT NULL |
| departure_time | TIME | NULL |
| arrival_time_estimated | TIME | NULL |
| status | VARCHAR(20) | SCHEDULED/IN_PROGRESS/COMPLETED/CANCELLED |
| bus_id | INT | FK → buses.id; nullable until assignment confirmed |
| driver_id | INT | FK → persons.id (must have role=DRIVER) |
| created_at | DATETIME2 | DEFAULT GETUTCDATE() |

**route_assignments**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| route_schedule_id | INT | FK → route_schedules.id, NOT NULL |
| bus_id | INT | FK → buses.id, NOT NULL |
| driver_id | INT | FK → persons.id, NOT NULL |
| assigned_at | DATETIME2 | DEFAULT GETUTCDATE() |

**student_route_assignments**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| student_person_id | INT | FK → persons.id, NOT NULL |
| route_schedule_id | INT | FK → route_schedules.id, NOT NULL |
| pickup_stop_id | INT | FK → stops.id, NOT NULL |
| dropoff_stop_id | INT | FK → stops.id, NOT NULL |
| assigned_at | DATETIME2 | DEFAULT GETUTCDATE() |
| UNIQUE(student_person_id, route_schedule_id) | | One assignment per student per schedule |

**trip_summary**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| route_schedule_id | INT | FK → route_schedules.id, UNIQUE |
| started_at | DATETIME2 | NULL |
| completed_at | DATETIME2 | NULL |
| total_stops_scanned | INT | DEFAULT 0 |
| missing_students | INT | DEFAULT 0; flagged at completion |
| duration_minutes | INT | NULL; computed from started/completed |
| status | VARCHAR(30) | PLANNED/IN_PROGRESS/COMPLETED/ABNORMAL_TERMINATION |

### 5.7 GPS Schema — 2 Tables

**position_events**
| Column | Type | Constraints |
| --- | --- | --- |
| id | BIGINT | PK, IDENTITY |
| route_execution_id | INT | FK → trip_summary.id |
| gps_device_id | INT | FK → gps_devices.id |
| latitude | DECIMAL(9,6) | NOT NULL |
| longitude | DECIMAL(9,6) | NOT NULL |
| altitude | DECIMAL(7,2) | NULL |
| speed_kmh | DECIMAL(6,2) | NULL |
| heading_degrees | DECIMAL(6,2) | NULL |
| accuracy_meters | DECIMAL(5,2) | NULL; GPS reported accuracy |
| recorded_at | DATETIME2 | Timestamp from GPS device |
| received_at | DATETIME2 | WHEN server received it (for delta calc) |

**connection_states**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| gps_device_id | INT | FK → gps_devices.id, NOT NULL |
| entered_offline_at | DATETIME2 | When signal was last seen |
| restored_at | DATETIME2 | NULL; set when connection resumes |
| duration_seconds | INT | COMPUTED column: restored_at - entered_offline_at |

### 5.8 Notification Schema — 5 Tables

**notification_queue**
| Column | Type | Constraints |
| --- | --- | --- |
| id | BIGINT | PK, IDENTITY |
| recipient_profile_id | UUID | FK → profiles.id; who receives this notification |
| template_key | VARCHAR(100) | Identifies notification type |
| payload_json | NVARCHAR(MAX) | Dynamic content: student names, times, locations |
| priority | INT | DEFAULT 5; 1=urgent, 10=normal |
| status | VARCHAR(20) | QUEUED/DISPATCHED/DELIVERED/FAILED |
| created_at | DATETIME2 | DEFAULT GETUTCDATE() |
| scheduled_for | DATETIME2 | NULL; for delayed notifications |

**push_delivery_attempts**
| Column | Type | Constraints |
| --- | --- | --- |
| id | BIGINT | PK, IDENTITY |
| notification_id | BIGINT | FK → notification_queue.id |
| attempt_number | INT | 1, 2, 3... |
| status | VARCHAR(20) | SUCCESS/FAILED/TIMEOUT |
| error_message | VARCHAR(500) | NULL; reason for failure |
| attempted_at | DATETIME2 | DEFAULT GETUTCDATE() |

**delivery_log**
| Column | Type | Constraints |
| --- | --- | --- |
| id | BIGINT | PK, IDENTITY |
| notification_id | BIGINT | FK → notification_queue.id |
| channel | VARCHAR(20) | PUSH/SMS/EMAIL |
| status | VARCHAR(20) | SENT/DELIVERED/READ/BOUNCED |
| sent_at | DATETIME2 | NULL |
| delivered_at | DATETIME2 | NULL; push-specific receipt |

**alert_templates**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| key | VARCHAR(100) | UNIQUE; e.g., "boarding.pickup", "incident.critical" |
| title_template | NVARCHAR(200) | "{student} picked up at {stop}" |
| body_template | NVARCHAR(500) | "Your child boarded route {route} at {time}" |
| channel_types | VARCHAR(100) | Comma-separated: PUSH,SMS |
| severity | VARCHAR(20) | LOW/MEDIUM/HIGH/CRITICAL |
| is_active | BIT | DEFAULT 1 |

**notification_preferences_override**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| profile_id | INT | FK → persons.id |
| alert_type_key | VARCHAR(100) | Which template this overrides |
| channel | VARCHAR(20) | PUSH/SMS/EMAIL |
| enabled | BIT | DEFAULT 1 |
| effective_from | DATE | NULL |
| effective_to | DATE | NULL |

### 5.9 Exceptional Schema — 2 Tables

**incident_reports**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| reporter_profile_id | UUID | FK → profiles.id; who reported it |
| route_schedule_id | INT | FK → route_schedules.id; associated trip |
| incident_type | VARCHAR(50) | MECHANICAL, ACCIDENT, MEDICAL, SECURITY, WEATHER, MISSED_STUDENT, OTHER |
| severity | VARCHAR(20) | LOW/MEDIUM/HIGH/CRITICAL |
| title | VARCHAR(200) | Brief description |
| description | TEXT | Detailed account |
| location_lat | DECIMAL(9,6) | NULL |
| location_lng | DECIMAL(9,6) | NULL |
| status | VARCHAR(30) | OPEN/ACKNOWLEDGED/INVESTIGATING/RESOLVED/CLOSED |
| resolved_at | DATETIME2 | NULL |
| resolved_by | UUID | FK → profiles.id |
| created_at | DATETIME2 | DEFAULT GETUTCDATE() |
| attachments_json | NVARCHAR(MAX) | File references (photos, documents) |

**remediation_actions**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| incident_report_id | INT | FK → incident_reports.id, NOT NULL |
| action_taken | TEXT | What was done to resolve |
| taken_by | UUID | FK → profiles.id |
| taken_at | DATETIME2 | DEFAULT GETUTCDATE() |
| follow_up_required | BIT | DEFAULT 0 |
| follow_up_due_date | DATE | NULL |

### 5.10 Settings Schema — 2 Tables

**platform_settings**
| Column | Type | Constraints |
| --- | --- | --- |
| id | INT | PK, IDENTITY |
| key | VARCHAR(100) | UNIQUE; setting identifier |
| value | NVARCHAR(MAX) | Setting value (JSON for complex objects) |
| category | VARCHAR(50) | GENERAL, NOTIFICATION, GPS, APPEARANCE |
| updated_by | UUID | FK → profiles.id |
| updated_at | DATETIME2 | DEFAULT GETUTCDATE() |

Example keys: `max_stationary_threshold_sec`, `default_language`, `gps_update_interval_sec`, `rate_limit_per_minute`, `theme_default_palette`.

**audit_actions**
| Column | Type | Constraints |
| --- | --- | --- |
| id | BIGINT | PK, IDENTITY |
| actor_profile_id | UUID | FK → profiles.id; who performed action |
| action_type | VARCHAR(100) | CREATE, UPDATE, DELETE, LOGIN, LOGOUT, ASSIGN, DEACTIVATE |
| entity_type | VARCHAR(50) | USER, ROUTE, BUS, ACCOUNT, CONFIG |
| entity_id | VARCHAR(100) | Affected record identifier |
| previous_values_json | NVARCHAR(MAX) | Snapshot before change |
| new_values_json | NVARCHAR(MAX) | Snapshot after change |
| ip_address | VARCHAR(45) | Actor's IP |
| user_agent | VARCHAR(512) | Browser/app string |
| application_name | VARCHAR(50) | angular-web, react-driver, react-parent |
| timestamp | DATETIME2 | DEFAULT GETUTCDATE() |

*Indexes*: Composite `(timestamp DESC)` for chronological queries; `(entity_type, entity_id)` for entity-centric lookups; `(actor_profile_id, timestamp)` for user activity timeline.

---

*Document end — Software Design Document for Guardian Escolar.*
