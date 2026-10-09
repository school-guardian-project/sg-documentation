# Software Implementation Procedure — Guardian Escolar

---

## Table of Contents

- [1. Purpose and Scope](#1-purpose-and-scope)
  - [1.1 Purpose](#11-purpose)
  - [1.2 Scope](#12-scope)
  - [1.3 Audience and Responsibilities](#13-audience-and-responsibilities)
  - [1.4 Usage Conventions](#14-usage-conventions)
- [2. Prerequisites](#2-prerequisites)
  - [2.1 Hardware Resources](#21-hardware-resources)
  - [2.2 Base Software](#22-base-software)
  - [2.3 Network and Ports](#23-network-and-ports)
  - [2.4 Permissions, Secrets, and Access](#24-permissions-secrets-and-access)
  - [2.5 Pre-Implementation Validation](#25-pre-implementation-validation)
- [3. Environment Architecture](#3-environment-architecture)
  - [3.1 Supported Environments](#31-supported-environments)
  - [3.2 Deployment Topology](#32-deployment-topology)
  - [3.3 Component Inventory and Ports](#33-component-inventory-and-ports)
  - [3.4 Configuration Policies per Environment](#34-configuration-policies-per-environment)
- [4. Step-by-Step Implementation Procedure](#4-step-by-step-implementation-procedure)
  - [4.1 Step 1: Base Infrastructure](#41-step-1-base-infrastructure)
    - [4.1.1 Container Network and Secrets](#411-container-network-and-secrets)
    - [4.1.2 Database Engine](#412-database-engine)
    - [4.1.3 Event Broker](#413-event-broker)
    - [4.1.4 Ephemeral Store](#414-ephemeral-store)
    - [4.1.5 API Gateway](#415-api-gateway)
  - [4.2 Step 2: Database and Migrations](#42-step-2-database-and-migrations)
    - [4.2.1 Change Structure](#421-change-structure)
    - [4.2.2 Application Order](#422-application-order)
    - [4.2.3 Execution](#423-execution)
    - [4.2.4 Verification](#424-verification)
    - [4.2.5 Rollback](#425-rollback)
  - [4.3 Step 3: Domain Services](#43-step-3-domain-services)
    - [4.3.1 Startup Order](#431-startup-order)
    - [4.3.2 Build and Launch](#432-build-and-launch)
    - [4.3.3 Health Check and Ports](#433-health-check-and-ports)
  - [4.4 Step 4: Gateway and Routing](#44-step-4-gateway-and-routing)
    - [4.4.1 Service and Route Registration](#441-service-and-route-registration)
    - [4.4.2 Cross-Cutting Policies](#442-cross-cutting-policies)
    - [4.4.3 Routing Verification](#443-routing-verification)
  - [4.5 Step 5: Web Application](#45-step-5-web-application)
  - [4.6 Step 6: Mobile Application](#46-step-6-mobile-application)
- [5. Verification and Deployment Gates](#5-verification-and-deployment-gates)

---

## 1. Purpose and Scope

### 1.1 Purpose

This procedure establishes the reproducible sequence by which the **Guardian Escolar** platform is deployed to a target environment, verified for correct operation, and delivered to the educational institution for live use. The platform comprises a web administration application, mobile applications for driver and guardian, a single API gateway, nine domain microservices, an event broker, a low-latency communication channel, and a relational database organized in schemas.

The verifiable outcomes this procedure pursues are four:

1. **Identical infrastructure configuration across environments**, without undocumented differences.
2. **Database schemas applied correctly**, with all migrations versioned and rollback-capable.
3. **All services starting and passing health checks**, communicating via REST, events, and gRPC.
4. **API gateway routing verified**, ensuring client requests reach the correct backend service.

### 1.2 Scope

This procedure covers deployment to the following environments:

| Environment | Purpose | Access |
| --- | --- | --- |
| Development (local) | Developer feature work; full stack on developer machine | Developer only |
| Test (staging) | QA, UAT, performance validation | QA team + product owner |
| Production | Live operations serving institution users | Authenticated users only |

Excluded from this procedure: installation of GPS hardware on buses, ISP/network provisioning at the school campus, third-party account creation (Expo, SMS providers), legal procurement processes for software licenses.

### 1.3 Audience and Responsibilities

| Role | Responsibility | Authority Level |
| --- | --- | --- |
| DevOps Engineer | Execute this procedure; document deviations; maintain runbooks | Full access to infrastructure |
| Backend Developer | Verify service health after deployment; confirm endpoints respond | Read-only on prod; full on dev/test |
| QA Engineer | Run verification tests post-deployment | Read-only; test execution rights |
| Project Manager | Approve deployment window; coordinate institution notification | Decision authority on go/no-go |
| Institution IT Admin | Provide SQL Server access, network credentials, DNS records | Local infrastructure access |

### 1.4 Usage Conventions

| Symbol | Meaning |
| --- | --- |
| `[REQUIRED]` | Mandatory step; cannot be skipped |
| `[OPTIONAL]` | Recommended but context-dependent |
| `▶` | Action step — execute command or perform action |
| `☐` | Verification step — confirm result before proceeding |
| `⚠️` | Warning — potential risk if misconfigured |
| `↩` | Rollback step — revert if verification fails |

Commands shown use Docker Compose as the local orchestration mechanism. For production deployments using Kubernetes, equivalent commands are noted in section headers.

---

## 2. Prerequisites

### 2.1 Hardware Resources

| Resource | Development | Staging | Production |
| --- | --- | --- | --- |
| CPU Cores | ≥ 4 | ≥ 8 | ≥ 16 |
| RAM | ≥ 16 GB | ≥ 32 GB | ≥ 64 GB |
| Disk Space | ≥ 50 GB SSD | ≥ 200 GB SSD | ≥ 500 GB SSD |
| Network Bandwidth | ≥ 100 Mbps | ≥ 1 Gbps | ≥ 1 Gbps |

All targets run Linux x86_64 (Debian 12+ recommended). ARM64 builds provided but not officially tested for production.

### 2.2 Base Software

| Software | Version | Required For |
| --- | --- | --- |
| Docker Engine | ≥ 24.0 | Container runtime |
| Docker Compose | ≥ 2.24 | Multi-service orchestration |
| Node.js | ≥ 20 LTS | Web/mobile build tools |
| .NET SDK | ≥ 9.0 | C# service builds |
| JDK | ≥ 21 (LTS) | Java service builds |
| git | ≥ 2.40 | Source code management |
| curl / httpie | Any recent version | Health check verification |

Optional but recommended:
- kubectl ≥ 1.28 (for Kubernetes production deployments)
- Helm ≥ 3.13 (for chart-based deployments)
- GitHub CLI or GitLab CLI (for PR-triggered deployments)

### 2.3 Network and Ports

| Port | Protocol | Component |
| --- | --- | --- |
| 8000 | HTTPS/TLS | Kong API Gateway (external) |
| 8080–8089 | HTTP/REST | Microservices internal (8081-IAM, 8082-UserMgmt, etc.) |
| 9092 | TCP/Kafka | Kafka brokers |
| 5001 | gRPC/TLS | Telemetry stream endpoint |
| 1433 | TDS/TLS | SQL Server |
| 6379 | RESP | Redis cache |
| 3000 | HTTP | NGINX reverse proxy (production, optional) |

Firewall rules must allow inbound traffic on port 8000 (HTTPS) from the public internet in production. All other ports restricted to internal network.

### 2.4 Permissions, Secrets, and Access

Before deployment begins, ensure these secrets and credentials are available:

| Secret | Source | Format | Stored As |
| --- | --- | --- | --- |
| SQL_SA_PASSWORD | IT Admin provides | Strong password (≥ 16 chars, mixed case, numbers, symbols) | `SQL_SA_PASSWORD` env var |
| JWT_PRIVATE_KEY | Generated during setup | PEM-encoded RSA 2048-bit private key | `JWT_PRIVATE_KEY_PEM` |
| JWT_PUBLIC_KEY | Extracted from private key above | PEM-encoded RSA public key | `JWT_PUBLIC_KEY_PEM` |
| KAFKA_BOOTSTRAP_SERVERS | Infra team provides | `kafka:9092` | `KAFKA_BOOTSTRAP_SERVERS` |
| REDIS_CONNECTION_STRING | Infra team provides | `redis://localhost:6379` | `REDIS_URL` |
| EXPO_PUSH_TOKEN | Expo dashboard | Long-lived access token | `EXPO_PUSH_TOKEN` |
| ADMIN_INITIAL_EMAIL | Institution provides | Valid email address | Used during first admin creation |
| ADMIN_INITIAL_PASSWORD | System generates | Randomly generated, communicated securely | Delivered once at account creation |

⚠️ Never commit secret files to source control. Use `.env` files listed in `.gitignore` or external secret managers (Vault, AWS Secrets Manager, Azure Key Vault).

### 2.5 Pre-Implementation Validation

Run these checks before beginning deployment:

```bash
# Verify Docker
docker --version && docker compose version

# Verify connectivity to target host
ping <target-host-ip>

# Verify disk space
df -h /var/lib/docker

# Verify required ports are free
ss -tlnp | grep -E ':(8000|1433|9092|5001|6379)\b'

# Verify timezone synchronization
timedatectl status
```

All checks must pass before proceeding.

---

## 3. Environment Architecture

### 3.1 Supported Environments

| Environment | Deployment Method | Data Isolation | Restart Policy |
| --- | --- | --- | --- |
| Development | `docker compose up -d` on developer machine | Separate DB schema prefix (`dev_`) | Always restart |
| Test | Docker Compose on staging VM or Kubernetes namespace | Separate instance or separate DB schemas | Always restart |
| Production | Kubernetes cluster or managed containers | Separate physical instances where possible | Restart unless deployment in progress |

Environment variable file per environment: `.env.development`, `.env.staging`, `.env.production`. Each file contains environment-specific values; base values defined in `.env.example`.

### 3.2 Deployment Topology

```
┌─────────────── Client Devices ──────────────────────────────┐
│                    (Phones, Tablets, Desktops)              │
└──────────────────────────┬─────────────────────────────────┘
                           │ HTTPS :443
                           ▼
          ┌─────────────────────────────────┐
          │   Optional: NGINX Reverse Proxy  │
          │   TLS termination → Kong GW     │
          └────────────────┬────────────────┘
                           │
                    ┌──────▼──────┐
                    │  Kong GW    │
                    │  :8000      │
                    └──────┬──────┘
                           │ REST :8080
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
     ┌─────────┐    ┌──────────┐    ┌─────────────┐
     │ IAM Svc │    │Routes Svc│    │Notification │
     │ :8081   │    │ :8083    │    │Svc :8085    │
     └────┬────┘    └────┬─────┘    └─────┬───────┘
          │               │                │
     ┌────▼────────────────▼────────────────▼───────┐
     │         SQL Server :1433                      │
     │   (per-schema databases per service)          │
     └───────────────────────────────────────────────┘
     ┌───────────────────────────────────────────────┐
     │         Kafka :9092                            │
     └───────────────────────────────────────────────┘
     ┌───────────────────────────────────────────────┐
     │         Redis :6379                            │
     └───────────────────────────────────────────────┘
```

### 3.3 Component Inventory and Ports

| Component | Image / Build | Internal Port | External Port | Dependencies |
| --- | --- | --- | --- | --- |
| Kong Gateway | kong:3.8 | 8000, 8443 | 8000, 8443 | None (starts first) |
| SQL Server | mcr.microsoft.com/mssql/server:2022-latest | 1433 | 1433 (internal only) | None |
| Kafka | confluentinc/cp-kafka:7.6.1 | 9092 | 9092 (internal) | Depends on ZooKeeper-less mode (KRaft) |
| Redis | redis:7-alpine | 6379 | 6379 (internal) | None |
| IAM Service | iam-service:{commit} | 8081 | — | SQL Server, Kafka, Redis |
| User Mgmt Service | user-mgmt-service:{commit} | 8082 | — | SQL Server, Kafka |
| School Service | school-service:{commit} | 8084 | — | SQL Server, Kafka |
| Fleet Service | fleet-service:{commit} | 8083 | — | SQL Server, Kafka |
| Routes Service | routes-service:{commit} | 8083 | — | SQL Server, Kafka |
| Notifications Service | notification-service:{commit} | 8085 | — | SQL Server, Kafka |
| Exceptional Service | exceptional-service:{commit} | 8086 | — | SQL Server, Kafka |
| Audit Service | audit-service:{commit} | 8087 | — | SQL Server |
| Settings Service | settings-service:{commit} | 8088 | — | SQL Server, Redis |
| GPS Streamer | gps-streamer:{commit} | 5001 | — | SQL Server |
| Web Frontend | angular-web:{commit} | 3000 (dev) | — | Kong GW |
| Mobile Apps | React + Expo | N/A (device-side) | — | Kong GW via HTTPS |

### 3.4 Configuration Policies per Environment

| Setting | Development | Staging | Production |
| --- | --- | --- | --- |
| Log Level | DEBUG | INFO | WARN |
| Rate Limit | 1000 req/min | 500 req/min | 200 req/min |
| JWT Expiry | 60 min | 15 min | 15 min |
| Refresh Token Expiry | 90 days | 30 days | 30 days |
| CORS Origins | `*` (localhost only) | Specific domains | Specific domains only |
| Debug Symbols | Enabled | Disabled | Disabled |
| Sentry/Datadog | Not configured | Configured | Fully instrumented |
| Backups | Manual only | Daily automated | Hourly incremental + daily full |

---

## 4. Step-by-Step Implementation Procedure

### 4.1 Step 1: Base Infrastructure

#### 4.1.1 Container Network and Secrets

```bash
▶ Create Docker network for inter-service communication:
docker network create guardianscolar-network

☐ Verify network created:
docker network ls | grep guardianscolar-network
```

Create `.env` file based on `.env.example`:

```bash
cp .env.example .env
```

Edit `.env` with actual secret values (see Section 2.4). Do NOT use placeholder values.

```
# Core connections
SQL_SERVER_HOST=localhost
SQL_SA_PASSWORD=YourStrongPasswordHere123!
JWT_PRIVATE_KEY_PEM=<contents-of-private-key.pem>
JWT_PUBLIC_KEY_PEM=<contents-of-public-key.pem>

# Messaging
KAFKA_BOOTSTRAP_SERVERS=kafka:9092
REDIS_URL=redis://localhost:6379

# Notifications
EXPO_PUSH_TOKEN=exp-push-token-from-expo-dashboard

# App URLs
FRONTEND_WEB_URL=http://localhost:4200
FRONTEND_MOBILE_URL=exp://<your-device-ip>:8081
KONG_GATEWAY_URL=http://localhost:8000
```

#### 4.1.2 Database Engine

```bash
▶ Start SQL Server container:
docker run -d \
  --name sqlserver \
  --network guardianscolar-network \
  -e ACCEPT_EULA=Y \
  -e SA_PASSWORD=${SQL_SA_PASSWORD} \
  -p 1433:1433 \
  -v $(pwd)/data/sqlserver:/var/opt/mssql \
  mcr.microsoft.com/mssql/server:2022-latest

☐ Wait 60 seconds then verify:
docker exec sqlserver bash -c "echo SELECT 1 | /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P $SQL_SA_PASSWORD"
Expected output: 1
```

Verify no data loss: restart container and re-query same data.

#### 4.1.3 Event Broker

```bash
▶ Start Kafka in KRaft mode (no ZooKeeper):
docker run -d \
  --name kafka \
  --network guardianscolar-network \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@kafka:9093 \
  -e KAFKA_LISTENERS=INTERNAL://:9092,CONTROLLER://:9093,EXTERNAL://:0 \
  -e KAFKA_ADVERTISED_LISTENERS=INTERNAL://kafka:9092 \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=INTERNAL:PLAINTEXT,CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  -v $(pwd)/data/kafka:/var/lib/kafka/data \
  confluentinc/cp-kafka:7.6.1

☐ Verify topic creation endpoint responds:
curl -s http://kafka:9092 | head -5
```

#### 4.1.4 Ephemeral Store

```bash
▶ Start Redis container:
docker run -d \
  --name redis \
  --network guardianscolar-network \
  -p 6379:6379 \
  -v $(pwd)/data/redis:/data \
  redis:7-alpine redis-server --appendonly yes

☐ Verify Redis responds:
docker exec redis redis-cli ping
Expected: PONG
```

#### 4.1.5 API Gateway

```bash
▶ Start Kong Gateway:
docker run -d \
  --name kong \
  --network guardianscolar-network \
  -e KONG_DATABASE=off \
  -e KONG_PROXY_ACCESS_LOG=/dev/stdout \
  -e KONG_ADMIN_ACCESS_LOG=/dev/stdout \
  -e KONG_PROXY_ERROR_LOG=/dev/stderr \
  -e KONG_ADMIN_LISTEN=0.0.0.0:8001 \
  -p 8000:8000 \
  -p 8443:8443 \
  -p 8001:8001 \
  kong:3.8

☐ Verify Kong admin API responds:
curl -s http://localhost:8001/status | jq .database.connection
Expected: {"available":true,"connected":true}
```

Base infrastructure is now running. Proceed to database migrations.

### 4.2 Step 2: Database and Migrations

#### 4.2.1 Change Structure

All schema changes live under `migrations/` directory organized by service:

```
migrations/
├── changelog-master.yaml          # Liquibase master file
├── iam/
│   ├── V1__create_profiles.sql
│   ├── V2__create_roles.sql
│   ├── V3__create_credentials.sql
│   └── ...
├── user-management/
│   ├── V1__create_persons.sql
│   ├── V2__create_academic_records.sql
│   └── ...
├── fleet/
│   ├── V1__create_buses.sql
│   └── ...
├── route/
│   ├── V1__create_routes.sql
│   └── ...
├── geolocation/
│   └── V1__create_position_events.sql
├── notifications/
│   ├── V1__create_notification_queue.sql
│   └── ...
├── exceptional/
│   ├── V1__create_incident_reports.sql
│   └── ...
├── audit/
│   └── V1__create_action_logs.sql
└── settings/
    └── V1__create_platform_settings.sql
```

Each migration file is idempotent: can be applied multiple times safely without side effects.

#### 4.2.2 Application Order

Apply migrations in dependency order:

1. `iam/` — Foundation tables (profiles, roles, credentials)
2. `user-management/` — Person records, academic data
3. `settings/` — Platform configuration defaults
4. `audit/` — Append-only action log (can be applied any time)
5. `school/` — Institutions, branches, courses
6. `fleet/` — Buses, devices
7. `route/` — Routes, stops, schedules
8. `geolocation/` — Position events table
9. `notifications/` — Notification queue, delivery tracking
10. `exceptional/` — Incident reports, remediation actions

#### 4.2.3 Execution

```bash
▶ Run all pending migrations:
cd migrations
liquibase --changeLogFile changelog-master.yaml \
  --url "jdbc:sqlserver://${SQL_SERVER_HOST}:1433;databaseName=guardianscolar;encrypt=true;trustServerCertificate=true;" \
  --username sa \
  --password "${SQL_SA_PASSWORD}" \
  update

☐ Verify no errors:
liquibase --changeLogFile changelog-master.yaml \
  --url "jdbc:sqlserver://${SQL_SERVER_HOST}:1433;databaseName=guardianscolar;encrypt=true;trustServerCertificate=true;" \
  --username sa \
  --password "${SQL_SA_PASSWORD}" \
  status
Expected: "All updates have been completed successfully."
```

Verify each schema exists:

```bash
docker exec sqlserver bash -c "echo \"SELECT name FROM sys.schemas WHERE name IN ('Iam','UserManagement','School','Fleet','Route','Geolocation','Notifications','Exceptional','Audit','Settings')\" | /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P ${SQL_SA_PASSWORD}"
```

Expected: list of all 10 schema names.

#### 4.2.4 Verification

After migrations complete:

| Check | Command | Expected Result |
| --- | --- | --- |
| Tables created | Query `INFORMATION_SCHEMA.TABLES` | ≥ 35 tables across all schemas |
| Indexes present | Query `sys.indexes` | Primary keys, foreign keys, unique constraints indexed |
| Seed data loaded | Count profiles in role ADMIN | ≥ 1 record (initial admin account) |
| No constraint violations | `CHECK_CONSTRAINTS` query | Zero disabled constraints |

#### 4.2.5 Rollback

If verification fails:

```bash
▶ Rollback last migration batch:
liquibase --changeLogFile changelog-master.yaml \
  --url "..." \
  --username sa \
  --password "${SQL_SA_PASSWORD}" \
  rollbackCount 1

☐ Re-execute: fix the migration file, then re-run update.
```

For catastrophic failures requiring full database reset:

```bash
▶ Drop and recreate:
docker exec sqlserver bash -c "echo 'DROP DATABASE guardianscolar' | /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P ${SQL_SA_PASSWORD}"

▶ Recreate fresh and apply migrations again (Sections 4.2.2–4.2.3).
```

⚠️ Rolling back or dropping destroys all data. Only perform on non-production environments without institutional data.

### 4.3 Step 3: Domain Services

#### 4.3.1 Startup Order

Services start in dependency order:

1. `ms-settings` — Reads default config from DB
2. `ms-audit` — Depends only on DB
3. `ms-iam` — Core auth; others depend on it
4. `ms-user-management` — Depends on IAM for profile lookups
5. `ms-school` — Independent except DB
6. `ms-fleet` — Depends on DB; independent otherwise
7. `ms-route` — Depends on fleet assignments
8. `ms-notification` — Depends on all publish-subscribe participants
9. `ms-exceptional` — Depends on route and notification events
10. `ms-gps-streamer` — Last; consumes position events

#### 4.3.2 Build and Launch

```bash
▶ Build all services from source:
cd ../../backend

# Build Java service (IAM)
cd ms-iam && ./gradlew bootJar && cd ../..

# Build .NET services
cd ../ms-fleet && dotnet publish -c Release && cd ../..
# Repeat for: ms-route, ms-notification, ms-exceptional, ms-audit, ms-settings, ms-school, ms-user-management

▶ Start services individually (or via docker-compose.prod.yml for orchestrated launch):
docker compose -f docker-compose.prod.yml up -d

# Or start one at a time for debugging:
docker compose up -d ms-settings ms-audit ms-iam
sleep 15
docker compose up -d ms-user-management ms-school ms-fleet
sleep 15
docker compose up -d ms-route ms-notification ms-exceptional ms-gps-streamer
```

#### 4.3.3 Health Check and Ports

```bash
▶ Wait for services to stabilize (60 seconds after last start):
sleep 60

▶ Check health endpoints:
for svc in iam:8081 user-mgmt:8082 school:8084 fleet:8083 route:8083 notification:8085 exceptional:8086 audit:8087 settings:8088 gps-streamer:5001; do
  echo -n "$svc -> "
  curl -s http://localhost:${svc#*:}/health | jq -r '.status'
done

☐ All should return "UP". If any returns "DOWN" or times out, check logs:
docker logs -f <container-name>
```

Fix failing services before proceeding. Common issues:
- Missing database connection string
- Kafka consumer group error
- Port conflict with another process

### 4.4 Step 4: Gateway and Routing

#### 4.4.1 Service and Route Registration

Configure Kong to route incoming `/api/v1/*` requests to appropriate backend services:

```bash
▶ Register upstream services:
# IAM
curl -s -X POST http://localhost:8001/upstreams \
  -d name=iam-service \
  -d hash_on=uri \
  -d slots=100 | jq .

curl -s -X POST http://localhost:8001/upstreams/iam-service/targets \
  -d target=iam-service:8081 | jq .

# Routes
curl -s -X POST http://localhost:8001/upstreams \
  -d name=routes-service \
  -d hash_on=uri \
  -d slots=100 | jq .

curl -s -X POST http://localhost:8001/upstreams/routes-service/targets \
  -d target=route-service:8083 | jq .

# Notifications
curl -s -X POST http://localhost:8001/upstreams \
  -d name=notification-service \
  -d hash_on=uri \
  -d slots=100 | jq .

curl -s -X POST http://localhost:8001/upstreams/notification-service/targets \
  -d target=notification-service:8085 | jq .

# ... repeat for all services ...

▶ Register routes on gateway:
# Authentication endpoints
curl -s -X POST http://localhost:8001/routes \
  -d name=auth-login \
  -d paths=/api/v1/auth/login \
  -d methods=POST \
  -d upstream_url=http://iam-service:8081 | jq .

curl -s -X POST http://localhost:8001/routes \
  -d name=auth-refresh \
  -d paths=/api/v1/auth/refresh \
  -d methods=POST \
  -d upstream_url=http://iam-service:8081 | jq .

# User management
curl -s -X POST http://localhost:8001/routes \
  -d name=users \
  -d paths=/api/v1/users \
  -d methods=GET,POST,PUT,DELETE \
  -d upstream_url=http://user-mgmt-service:8082 | jq .

# Routes CRUD
curl -s -X POST http://localhost:8001/routes \
  -d name=routes-crud \
  -d paths=/api/v1/routes \
  -d methods=GET,POST,PUT,DELETE \
  -d upstream_url=http://route-service:8083 | jq .

# Boarding/QR scanning
curl -s -X POST http://localhost:8001/routes \
  -d name=boardings \
  -d paths=/api/v1/boardings \
  -d methods=POST \
  -d upstream_url=http://notification-service:8085 | jq .

# ... register remaining routes for each service ...
```

#### 4.4.2 Cross-Cutting Policies

```bash
▶ Apply rate limiting to all routes:
curl -s -X POST http://localhost:8001/plugins \
  -d name=rate-limiting \
  -d config.minute=200 \
  -d config.policy=local | jq .

▶ Apply JWT authentication plugin:
curl -s -X POST http://localhost:8001/plugins \
  -d name=jwt \
  -d config.key_claim_name=iss \
  -d config.claims_to_verify=exp | jq .

▶ Apply CORS plugin:
curl -s -X POST http://localhost:8001/plugins \
  -d name=cors \
  -d config.origins=["https://admin.guardianscolar.edu.co"] \
  -d config.methods=[GET,POST,PUT,DELETE,PATCH] \
  -d config.headers=[Content-Type,Authorization,X-Request-ID] \
  -d config.credentials=true | jq .

▶ Apply request-size-limiting:
curl -s -X POST http://localhost:8001/plugins \
  -d name=request-size-limiting \
  -d config.allowed_payload_size=10 \
  -d config.original_body_size_limit=0 | jq .
```

#### 4.4.3 Routing Verification

```bash
☐ Verify gateway routes correctly:
curl -s http://localhost:8001/routes | jq '.data[].paths'
Expected: list of registered path patterns (/api/v1/...)

☐ Test direct backend call (should bypass gateway):
curl -s http://localhost:8081/health
Expected: {"status":"UP"}

☐ Test through gateway (should forward to backend):
curl -s http://localhost:8000/health
Expected: {"status":"UP"} or routed response

☐ Verify JWT enforcement on protected route:
curl -s http://localhost:8000/api/v1/users
Expected: 401 Unauthorized (no valid JWT)

curl -s -H "Authorization: Bearer INVALID_TOKEN" http://localhost:8000/api/v1/users
Expected: 401 Unauthorized
```

### 4.5 Step 5: Web Application

```bash
▶ Install dependencies:
cd apps/web-admin
npm install

▶ Build Angular application:
ng build --configuration production

☐ Verify build artifacts exist:
ls dist/web-admin/browser/
Expected: index.html, main.*, polyfills.*

▶ Serve locally for testing:
npx serve dist/web-admin/browser -s -l 4200

☐ Open in browser and verify:
http://localhost:4200
Expected: Login page renders; Kong proxy forwards auth calls to gateway.

▶ For production deployment: copy built files to static hosting or embed in Kong as a custom plugin serving static assets.
```

Configuration for production web app:

```typescript
// apps/web-admin/src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://api.guardianscolar.edu.co/api/v1',
  mapTileUrl: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
  appName: 'Guardian Escolar Admin',
  supportEmail: 'soporte@guardianscolar.edu.co'
};
```

### 4.6 Step 6: Mobile Application

```bash
▶ Install dependencies:
cd apps/mobile

# Ensure Expo CLI installed
npx expo install expo-router expo-notifications expo-camera expo-location

▶ Run development build:
npx expo start --dev-client

☐ On physical device:
- Scan QR code from terminal output using Expo Go app
- OR build native binary: npx expo run:android / npx expo run:ios

▶ Build production binary:
# Android
npx expo build:android --release-channel stable

# iOS (requires Mac)
npx expo build:ios --release-channel stable

☐ Verify app connects to backend:
Launch app → Login → Navigate to route screen
Expected: Route data loads; map displays; notifications appear when triggered.
```

---

## 5. Verification and Deployment Gates

After completing all steps, run the full verification suite:

| Gate | Test | Pass Criteria |
| --- | --- | --- |
| Infrastructure | `docker ps | wc -l` | ≥ 15 containers running |
| Database | Liquibase status | All migrations applied |
| Health Checks | curl to `/health` on every service | All return UP |
| Gateway Routing | Test 5 random endpoints through Kong | 200 OK for authenticated calls |
| Event Flow | Publish test event to Kafka → consume on subscriber topic | Event received within 2 seconds |
| Web App | Load admin login page in browser | Renders without JS/console errors |
| Mobile App | Login on device; trigger boarding simulation | Notification arrives within 10 seconds |
| End-to-End | Full boarding flow: create user → assign to route → simulate QR scan → verify notification | Complete chain succeeds |

Deployment is ready for sign-off when all gates pass. Document results in deployment report.

---

*Document end — Software Implementation Procedure for Guardian Escolar.*
