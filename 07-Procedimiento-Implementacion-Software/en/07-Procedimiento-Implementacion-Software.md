
**Software Implementation Procedure**

Guardian Escolar

Software for the creation of the "Guardian Escolar" application

Johan Smith Santamaria Fernández  
Sharik Dayanna Rojas Ibarra  
Juan Pablo Chala Ramírez

SENA — Servicio Nacional de Aprendizaje  
Programming cohort 3145556  
Neiva training center (Huila)

Instructor: José de Jesús Motta Vargas  
Neiva, October 6, 2026

# Version control

**Table 1**. Document version control

| Version | Date       | Author                  | Change description            |
| ------- | ---------- | ----------------------- | ----------------------------- |
| 1.0     | 06/10/2026 | Guardian Escolar Team   | Initial version of the document |

**Table of contents**

Press F9 (or right-click → Update field) to generate the table of contents.

# 1 Purpose and scope

## 1.1 Purpose

This procedure establishes the reproducible sequence by which the **Guardian Escolar** platform is deployed in a target environment, its operation is verified, and it is delivered to the educational institution for operation. The platform comprises an administrative web application, mobile applications for the driver and the guardian, a single gateway, nine domain microservices, an event broker, a low-latency communication channel, and a single relational database organized into schemas.

The procedure pursues four verifiable outcomes:

- Infrastructure configured identically in every environment, with no undocumented differences.
- A complete database —**11 schemas** and **38 tables**— by means of versioned and verifiable migrations.
- Each component exposed only on the authorized ports and responding to its health checks.
- Objective acceptance evidence, with a **PASS/FAIL** verdict per item, before the deployment is declared complete.

## 1.2 Scope

Includes: preparation of prerequisites; base infrastructure; migrations; deployment of the nine domain services; publication of the client applications; initial configuration of policies, catalogs and seed users; post-deployment verification; versioning; rollback; continuous integration and publication channel; monitoring and support; responsibility matrix and schedule; change management and acceptance.

Excludes: feature development and code-level testing, performed in the earlier phases of the life cycle; the acquisition of third-party licenses; and the provisioning of mobile terminals and geolocation devices, which is the responsibility of the receiving institution.

## 1.3 Audience and responsibilities

**Table 2**. User profiles of the procedure

| Profile                          | Use of the procedure                                        |
| -------------------------------- | ----------------------------------------------------------- |
| Technical lead                   | Approves stages and releases; decides on rollback.          |
| Platform engineer                | Executes infrastructure, services, edge and clients.        |
| Database owner                   | Executes and verifies migrations, backups and restores.     |
| Quality owner                    | Executes post-deployment verification and issues the verdict. |
| Operations owner                 | Receives the system; operates monitoring, alerts and incidents. |
| Client (educational institution) | Provides observations and signs acceptance.                 |

## 1.4 Usage conventions

- Values between angle brackets, for example &lt;SA_PASSWORD&gt; or &lt;ADMIN_EMAIL&gt;, are **secret placeholders**: real values are never written in this document or in the deployment records.
- Every technical command is reproduced in a bash block; the expected results are described in prose or in a table.
- A step closes with a check; if it is not met, the step fails and no progress is made.
- The schedule durations are **planning estimates**; contractual commitments are established in the acceptance record.
- "Environment" designates the set of components deployed for the same purpose; "component", any deployable unit.

# 2 Prerequisites

## 2.1 Hardware resources

The sizing is agreed with the institution according to the expected volume of students, routes and devices. The deployment host must meet:

**Table 3**. Hardware requirements of the deployment host

| Resource             | Functional requirement                                                                                                           | Verification criterion                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Processor and memory | Run the data engine, broker, gateway and services simultaneously without sustained saturation or memory swapping               | Stable usage below the alert threshold during the test window     |
| Storage              | Persistent volume for the database and broker, with space for full and log backups                                               | Free space above the alert threshold                              |
| Operating system     | Platform with container execution and a working orchestration client                                                             | Version and service status verified                              |
| Mobile terminals     | Camera for QR scanning and push notifications, one per driver on duty                                                            | Functional verification in step 4.6                               |

## 2.2 Base software

**Table 4**. Base software and minimum versions

| Component                      | Minimum version | Purpose                                  | Check                                   |
| ------------------------------ | --------------- | ---------------------------------------- | --------------------------------------- |
| Container engine               | 24.0            | Execution of components                  | docker --version                        |
| Container orchestrator         | 2.20            | Definition and inversion of sets         | docker compose version                  |
| Version control                | 2.40            | Retrieval of the code baseline           | git --version                           |
| SQL Server                     | 2022            | Single relational persistence            | Healthy container                       |
| Apache Kafka                   | 4.1.0 (KRaft)   | Domain events                            | Operational topic consumption           |
| Kong                           | 3.9             | Routing, rate limiting and CORS          | Response on the public port             |
| Liquibase                      | 4.29            | Versioned schema changes                 | Zero pending changesets                 |
| Java with Maven                | 21              | Identity and access; geolocation         | Build without errors                    |
| .NET                           | 10              | Remaining domain services                | Build without errors                    |
| Node.js with package manager   | LTS             | Build of client applications             | Build without errors                    |
| Angular with Angular Material  | 21 and Material 3 | Web application                        | Satisfactory production build           |
| React with Expo                | Active version  | Mobile applications                      | Successfully generated binary           |

## 2.3 Network and ports

**Table 5**. Authorized ports per component

| Port        | Component                               | Allowed scope                                   |
| ----------- | --------------------------------------- | ----------------------------------------------- |
| 8000        | Gateway (proxy)                         | Public: single entry point                      |
| 8443        | Gateway (proxy TLS)                     | Public: TLS termination                         |
| 8001 and 8002 | Edge API and administrative interface  | Internal network only                           |
| 8080        | Domain services                         | Container network only, resolved by name        |
| 5001        | Geolocation gRPC channel                | Internal network only                           |
| 1433        | SQL Server                              | Internal network only; not exposed to the Internet |
| 9092        | Apache Kafka                            | Internal network only; not exposed to the Internet |
| 6379        | Redis                                   | Internal network only: cache and token blacklist |
| 8090        | Data query tool                         | Development only; forbidden in production       |
| 4200        | Web development server                  | Development only: CORS reference origin         |

Before starting, it is confirmed that no port (in particular 8000, 1433 and 9092) is occupied by another process on the host, with lsof -i :&lt;port&gt;.

## 2.4 Permissions, secrets and accesses

**Table 6**. Required permissions, secrets and accesses

| Element                                                           | Requirement                                                 | Handling                                          |
| ----------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------- |
| Host access                                                       | Container system administration privileges                   | Personal account with traceability                |
| Engine password (&lt;SA_PASSWORD&gt;) and user (&lt;SA_USER&gt;)  | Generated before the first start and different per environment | Secret store; never in the code                  |
| Signing private key (&lt;JWT_PRIVATE_KEY&gt;)                     | Exclusive to the identity and access service                | RS256: 15-minute access and 30-day refresh        |
| Verification public key                                           | Distributed to the other services and to the edge           | Declared configuration                            |
| Mobile messaging credentials (&lt;PUSH_CREDENTIALS&gt;)           | Provisioned by the provider                                 | Environment secret store                          |
| Allowed web origin (&lt;WEB_ORIGIN&gt;)                           | Public domain of the web application                        | Declared edge configuration                       |
| Image registry                                                    | Publish and pull permission                                 | Credentials rotated per environment               |

## 2.5 Pre-deployment validation

docker info

docker compose version

df -h

docker network ls

docker images

Deployment does not begin if any check fails or if the code baseline does not correspond to the version approved for release.

# 3 Environment architecture

## 3.1 Supported environments

**Table 7**. Supported environments and their purpose

| Environment             | Purpose                                     | Update                                          | Access                                     | Data                          |
| ----------------------- | ------------------------------------------- | ----------------------------------------------- | ------------------------------------------ | ----------------------------- |
| Development             | Individual build                            | Manual                                          | Technical team                             | Synthetic demonstration       |
| Integration             | Continuous validation of each approved change | Automatic upon merging into the development branch | Technical team                        | Synthetic                     |
| Testing (pre-production) | Verification, demonstration and acceptance | Automatic upon merging into the main branch     | Technical team and product owner           | Anonymized from production    |
| Production              | Operation with real users                   | Manual with formal approval                     | Technical lead and platform team           | Real                          |

## 3.2 Deployment topology

Guardian (mobile) Driver (mobile) Administrator (web)

\\ | /

+-------+-------+--------+--------+

| TLS/HTTP :8000 |

v |

+------------------+ |

| Gateway |--- CORS, rate limiting, JWT

| (Kong OSS) |

+--------+---------+

| container network, each service :8080

+---------+-------+------+----------+----------+-----------+

v v v v v v

+-----+ +--------+ +---------+ +---------+ +----------+ +----------+

| IAM | |Users| | School | | Fleet | | Routes | |Notificat.|

+--+--+ +---+----+ +----+-----+ +----+----+ +----+-----+ +----+-----+

| | | | | |

+-------+------------+-----+------+-----------+-------------+

| gRPC channel :5001

v

+----------------+ +---------------+

| Exceptional | | Audit and |

+----------------+ | Configuration |

+---------------+

|

v

+------------------+ +------------------+ +------------------+

| SQL Server :1433 | | Kafka :9092 | | Redis :6379 |

| 11 schemas | | KRaft mode | | ephemeral cache |

| 38 tables | | 6 topics | +------------------+

+------------------+ +------------------+

**Figure 1**. Deployment topology of the platform

## 3.3 Component and port inventory

**Table 8**. Component and port inventory

| Component                     | Nature          | Port               | Persistence                            |
| ----------------------------- | --------------- | ------------------ | -------------------------------------- |
| Gateway (Kong OSS)            | Infrastructure  | 8000 / 8443        | Versioned declared configuration       |
| Identity and access (IAM)     | Domain          | 8080               | Iam schema                             |
| User management               | Domain          | 8080               | UserManagement schema                  |
| School management             | Domain          | 8080 / 5001 (gRPC) | School and Geographic schemas          |
| Fleet                         | Domain          | 8080               | Fleet schema                           |
| Routes                        | Domain          | 8080               | Route schema                           |
| Notifications                 | Domain          | 8080               | Notification schema                    |
| Exceptional uses              | Domain          | 8080               | Exceptional schema                     |
| Audit                         | Domain          | 8080               | Audit schema                           |
| Configuration                 | Domain          | 8080               | Settings schema                        |
| SQL Server 2022               | Infrastructure  | 1433               | Persistent volume                      |
| Apache Kafka 4.1.0 (KRaft)    | Infrastructure  | 9092               | Persistent volume                      |
| Redis                         | Infrastructure  | 6379               | Ephemeral, no restore requirement      |

## 3.4 Configuration policies per environment

**Table 9**. Configuration policies per environment

| Parameter            | Development              | Integration                        | Testing                        | Production                        |
| -------------------- | ------------------------ | ---------------------------------- | ------------------------------ | --------------------------------- |
| Log level            | debug                    | info                               | info                           | warn                              |
| Allowed web origin   | <http://localhost:4200>  | https://&lt;integration-domain&gt; | https://&lt;testing-domain&gt; | https://&lt;production-domain&gt; |
| Secret source        | Local environment file   | Secret store                       | Secret store                   | Secret store                      |
| Data                 | Synthetic                | Synthetic                          | Anonymized                     | Real                              |
| Publication          | Manual                   | Automatic                          | Automatic                      | Manual with double approval       |

The variable naming convention is APP_\[SERVICE\]\_\[VARIABLE\]; each service also keeps its connection string and broker address keys.

# 4 Step-by-step implementation procedure

## 4.1 Step 1: base infrastructure

### 4.1.1 Container network and secrets

1. The shared network that interconnects all the service sets is created.
2. The environment file of each set is prepared from its example template.
3. All angle-bracket placeholders are replaced with values from the environment secret store.
4. It is confirmed that no environment file with real values is incorporated into the code baseline.

docker network create &lt;network-name&gt;

cp .env.example .env

openssl rand -base64 32

docker network ls

Closing: the network appears in the inventory and the environment file contains only values permitted by the secret policy.

### 4.1.2 Database engine

1. SQL Server 2022 is started with port 1433 assigned only to the internal network and with a persistent volume.
2. It waits until the built-in health check —SELECT 1 every 5 seconds, up to 20 retries and 20 seconds initial wait— reports a healthy status.
3. The initialization container is run; it creates the school-guardian database if it does not exist and exits with code 0.

docker compose up -d

docker compose ps

docker inspect --format '{{json .State.Health.Status}}' &lt;sql-container-name&gt;

### 4.1.3 Event broker

Apache Kafka 4.1.0 is started in KRaft mode, with a single node that performs the roles of broker and controller and a controller quorum on port 9093. The initialization container waits until the broker responds to the topic enumeration and creates the pending topics with one partition and replication factor 1.

**Table 10**. Topics declared in the event broker

| Topic           | Partitions | Replication factor | Use                                   |
| --------------- | ---------- | ------------------ | ------------------------------------- |
| student.created | 1          | 1                  | Student registration toward identity  |
| driver.created  | 1          | 1                  | Driver registration toward identity   |
| admin.created   | 1          | 1                  | Administrator registration toward identity |
| parent.created  | 1          | 1                  | Guardian registration toward identity |
| profile.created | 1          | 1                  | Profile created in identity           |
| student.scanned | 1          | 1                  | QR scan and boarding record           |

docker exec &lt;kafka-container-name&gt; /opt/kafka/bin/kafka-topics.sh \\

\--bootstrap-server kafka:9092 --list

Closing: the six topics appear in the listing and the initialization container exited with code 0.

### 4.1.4 Ephemeral store

Redis is started on port 6379 for caching reference data, rate limiting and token blacklisting. Being ephemeral, no recovery objective is defined: if it is lost, the instance is recreated and the clients reconnect.

### 4.1.5 Gateway

Kong is started with declared configuration and without its own database, mounting the manifest in read-only mode; the proxy ports (8000 and 8443) and the administrative ports (8001 and 8002) are verified, the latter restricted to the internal network.

curl -fsS <http://localhost:8000/health>

Closing: a response with status ok and a timestamp. At this point the gateway does not yet route business traffic: the routes are declared in step 4.4.

## 4.2 Step 2: database and migrations

### 4.2.1 Structure of the changes

Each schema has its own change tree managed by Liquibase, with the stages in this order: creation of the schema, custom types, tables, constraints and foreign keys between schemas, views, functions, procedures, triggers and indexes; then the initial data load, engine permissions, transactional blocks and release tags; and, lastly, a reserved tree of rollback statements.

The full set amounts to **11 schemas**, **38 tables** and **127 changesets**, each of which corresponds to a data definition script.

### 4.2.2 Order of application

The order is derived from the foreign keys that cross schemas; the orchestration chains the migration containers by means of successful-completion dependencies:

**Table 11**. Order of application of the migrations

| Order | Schema         | Changesets | Depends on                     |
| ----- | -------------- | ---------- | ------------------------------ |
| 1     | Geographic     | 3          | Engine initialization          |
| 2     | Gps            | 6          | Engine initialization          |
| 3     | Settings       | 3          | Engine initialization          |
| 4     | School         | 12         | Geographic                     |
| 5     | Iam            | 24         | School                         |
| 6     | Fleet          | 12         | Gps, School, Iam               |
| 7     | Route          | 25         | Geographic, School, Fleet, Iam |
| 8     | UserManagement | 18         | Iam                            |
| 9     | Notification   | 13         | Iam, Route                     |
| 10    | Audit          | 4          | Iam                            |
| 11    | Exceptional    | 7          | Iam, Fleet, Route              |

There are reciprocal relationships between the identity, school and user management schemas. When a changeset requires an object that does not yet exist, the engine reports it as not applied and leaves it pending; a second pass over the same set applies it once the schemas involved have been created. That is why the procedure expressly provides for re-execution before declaring the stage complete.

### 4.2.3 Execution

docker compose up -d sqlserver sqlserver-init

docker compose up -d

docker compose up -d

docker compose ps --filter "status=exited"

Each migration is run with the search path for the changelogs, the master changelog of the schema and the update command. Execution is idempotent: only the pending changesets are applied.

### 4.2.4 Verification

docker exec -it &lt;sql-container-name&gt; /opt/mssql-tools18/bin/sqlcmd \\

\-S localhost,1433 -U "&lt;SA_USER&gt;" -P "&lt;SA_PASSWORD&gt;" -C -d school-guardian \\

\-Q "SELECT ID, AUTHOR, DATEEXECUTED FROM DATABASECHANGELOG ORDER BY DATEEXECUTED;"

docker exec -it &lt;sql-container-name&gt; /opt/mssql-tools18/bin/sqlcmd \\

\-S localhost,1433 -U "&lt;SA_USER&gt;" -P "&lt;SA_PASSWORD&gt;" -C -d school-guardian \\

\-Q "SELECT SCHEMA_NAME(SCHEMA_ID) AS schema, COUNT(\*) AS tables FROM sys.tables GROUP BY SCHEMA_NAME(SCHEMA_ID) ORDER BY 1;"

docker exec -it &lt;sql-container-name&gt; /opt/mssql-tools18/bin/sqlcmd \\

\-S localhost,1433 -U "&lt;SA_USER&gt;" -P "&lt;SA_PASSWORD&gt;" -C -d school-guardian \\

\-Q "SELECT OBJECT_NAME(parent_object_id) AS table, name AS key FROM sys.foreign_keys ORDER BY 1;"

The stage is satisfactory when there are no pending changesets, 11 schemas and 38 tables are counted, no active change lock exists and the foreign keys between schemas are present.

### 4.2.5 Rollback

1. The SQL that would be applied is obtained without executing it, in order to review the impact.
2. The last applied change is rolled back when its rollback is declared in the rollback tree.
3. If it has no declared rollback, a forward fix approved by the database owner is applied.
4. Every rollback is recorded with reason and owner.

docker run --rm liquibase/liquibase:4.29 \\

\--searchPath=/workspace \\

\--changeLogFile=&lt;master-changelog-of-the-schema&gt; \\

\--url="jdbc:sqlserver://&lt;sql-host&gt;:1433;databaseName=school-guardian;encrypt=false;trustServerCertificate=true" \\

\--username="&lt;SA_USER&gt;" --password="&lt;SA_PASSWORD&gt;" \\

update-sql

The rollback-count 1 command instead of update-sql rolls back the last change. Before any rollback in an environment with data, a verified backup is restored; schema rollback does not replace data recovery.

## 4.3 Step 3: domain services

### 4.3.1 Startup order

Synchronous dependencies are resolved at request time and do not block startup; the deterministic order is set by the database, already available after the previous stage. The recommended order guarantees that identity is available before the services that validate people and profiles:

**Table 12**. Startup order of the domain services

| Order | Service                       | Schema             | Runtime               | Main logical dependency              |
| ----- | ----------------------------- | ------------------ | --------------------- | ------------------------------------ |
| 1     | Identity and access (IAM)     | Iam                | Java 21 + Spring Boot | Token issuance and verification      |
| 2     | User management               | UserManagement     | .NET 10               | Identity validation against IAM      |
| 3     | School management             | School, Geographic | .NET 10               | People and campus                    |
| 4     | Configuration                 | Settings           | .NET 10               | None: global parameters              |
| 5     | Fleet                         | Fleet              | .NET 10               | Institution, devices and profiles    |
| 6     | Routes                        | Route              | .NET 10               | Fleet, institution and profiles      |
| 7     | Notifications                 | Notification       | .NET 10               | Routes, fleet and profiles           |
| 8     | Exceptional uses              | Exceptional        | .NET 10               | Routes, fleet and institution        |
| 9     | Audit                         | Audit              | .NET 10               | Events from all domains              |

### 4.3.2 Build and startup

dotnet restore

dotnet build

dotnet test

./mvnw clean package

docker compose up -d --build

Each set publishes on port 8080 inside the container network, so no collisions occur on the host. The secrets and the addresses of the other services are injected through environment variables; they are never hard-coded in the source.

### 4.3.3 Health check and ports

**Table 13**. Health checks of the domain services

| Service                 | Check                                                    | Acceptance criterion             |
| ----------------------- | -------------------------------------------------------- | -------------------------------- |
| Identity and access     | Spring framework health endpoint (/actuator/health)      | HTTP 200 and status UP           |
| .NET services           | Health endpoint exposed by the web host                  | HTTP 200                         |
| All                     | Lookup by service name on the container network          | Response in less than two seconds |
| gRPC channel (5001)     | Channel establishment from the client                    | Channel available without errors |
| Broker                  | Production and consumption of a test event               | Event delivered to the consumer  |

curl -fsS http://&lt;service-name&gt;:8080/health

curl -fsS http://&lt;iam-service-name&gt;:8080/actuator/health

docker compose logs --tail=100 &lt;service-name&gt;

In the event of an unexpected response, the following are verified, in this order: container status, service logs, environment variables, availability of the database and of the broker. No progress is made until the nine services respond.

## 4.4 Step 4: gateway and routing

### 4.4.1 Service and route registration

The edge configuration is declared and versioned as code; every modification enters through a reviewed change request. Each domain service registers its destination on the container network, its routes under the versioned /api/v1 prefix, the accepted methods and its policies.

**Table 14**. Services declared in the gateway and their routes

| Declared service        | Destination                           | Public routes          | Methods                         |
| ----------------------- | ------------------------------------- | ---------------------- | ------------------------------- |
| Identity and access     | http://&lt;iam&gt;:8080               | /api/v1/auth           | GET, POST, PUT, DELETE, OPTIONS |
| User management         | http://&lt;user-management&gt;:8080   | People and families    | GET, POST, PUT, DELETE, OPTIONS |
| School management       | http://&lt;school-management&gt;:8080 | Institutions and campuses | GET, POST, PUT, DELETE, OPTIONS |
| Fleet                   | http://&lt;fleet&gt;:8080             | Buses and assignments  | GET, POST, PUT, DELETE, OPTIONS |
| Routes                  | http://&lt;route&gt;:8080             | Routes and stops       | GET, POST, PUT, DELETE, OPTIONS |
| Notifications           | http://&lt;notification&gt;:8080      | Alerts and boardings   | GET, POST                       |
| Exceptional uses        | http://&lt;exceptional&gt;:8080       | Exceptional uses       | GET, POST, PUT, DELETE, OPTIONS |
| Audit                   | http://&lt;audit&gt;:8080             | Audit and errors       | GET                             |
| Configuration           | http://&lt;configuration&gt;:8080     | Parameters and policies | GET, PUT                        |

Before releasing, it is verified that the declared prefixes conform to the versioned contract and that no domain service is left without a published route.

### 4.4.2 Cross-cutting policies

**Table 15**. Cross-cutting policies of the edge

| Policy           | Configuration                                                                                                                                                                                                  | Scope                                         |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| CORS             | Environment web origin and authorized development origins; methods GET, POST, PUT, DELETE, OPTIONS; headers Accept, Authorization, Content-Type; credentials active; 3600-second preflight | All routes                                    |
| Rate limiting    | 300 requests per minute, local policy, classification by Authorization header                                                                                                                                    | Authentication route, extensible to the others |
| Authentication   | RS256 JWT token verification at the edge, with deny by default                                                                                                                                                 | All resources except login and refresh        |
| TLS termination  | Environment certificate on proxy 8443                                                                                                                                                                          | Public traffic                                |
| Logging          | Standard output with correlation identifier                                                                                                                                                                    | All routes                                    |

### 4.4.3 Routing verification

curl -fsS <http://localhost:8000/health>

curl -i -X GET <http://localhost:8000/api/v1/&lt;protected-resource>&gt;

curl -i -X GET <http://localhost:8000/api/v1/&lt;protected-resource>&gt; \\

\-H "Authorization: Bearer &lt;ACCESS_TOKEN&gt;"

Closing: the edge responds; the protected resource without credentials is denied; with valid credentials the destination service responds; and the web application origin appears in the CORS policy.

## 4.5 Step 5: web application

1. The production build is run with the environment origin as the configuration value.
2. The result is reviewed: no compilation or static analysis errors.
3. The static artifact is published on the environment's content service, served over HTTPS.
4. The published origin is registered in the edge CORS policy.
5. The static content cache of the previous version is invalidated.

npm ci

npm run lint

npm run test

npm run build -- --configuration production

Closing: the application loads from the published origin, the browser reports no CORS violations, the login shows the form in accordance with the current length policy and the side menu reflects the authenticated role.

## 4.6 Step 6: mobile application

1. The binary version is set according to the scheme in section 6.
2. It is compiled for the two platforms with the configuration of the target environment.
3. The push notification credentials and the gateway base address are registered.
4. The camera (QR scanning), notification and location permissions are verified.
5. It is distributed: internal distribution package for testing and store package for production.
6. It is installed on a test terminal per role and the minimum walkthrough is executed.

npm ci

npm run test

npx eas build --platform android --profile production

npx eas build --platform ios --profile production

Closing: the driver logs in, consults the route of the day, opens the scanner and registers a test scan; the guardian consults the tracking and receives a notification; the notification token is registered on the platform.

## 4.7 Step 7: initial configuration

### 4.7.1 Environment variables

**Table 16**. Environment variables of the initial configuration

| Variable                              | Description                             | Example value                                                                                                              |
| ------------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| MSSQL_DATABASE                        | Database name                           | school-guardian                                                                                                            |
| MSSQL_USER / MSSQL_SA_PASSWORD        | Engine user and password                | &lt;SA_USER&gt; / &lt;SA_PASSWORD&gt;                                                                                      |
| ConnectionStrings_\_DefaultConnection | Connection string of each service       | Server=sqlserver;Database=school-guardian;User Id=&lt;SA_USER&gt;;Password=&lt;SA_PASSWORD&gt;;TrustServerCertificate=True |
| Kafka_\_BootstrapServers              | Broker address (.NET)                   | kafka:9092                                                                                                                 |
| KAFKA_BOOTSTRAP_SERVERS               | Broker address (Java)                   | &lt;broker-name&gt;:9092                                                                                                   |
| SCHOOL_GUARDIAN_DB_DIR                | Base location of the changelogs         | default value                                                                                                              |
| KONG_DECLARATIVE_CONFIG               | Edge manifest                           | /opt/kong/kong.yml                                                                                                         |
| Services_\_UserManagement_\_BaseUrl   | Base address of the user service        | http://&lt;user-management&gt;:8080                                                                                        |
| Services_\_Route_\_BaseUrl            | Base address of the routes service      | http://&lt;route&gt;:8080                                                                                                  |
| CORS_ORIGIN                           | Allowed web origin                      | https://&lt;production-domain&gt;                                                                                          |
| LOG_LEVEL / REDIS_URL                 | Logging and ephemeral store             | info / redis://&lt;redis&gt;:6379                                                                                          |
| JWT_PRIVATE_KEY                       | RS256 private key, only in IAM          | &lt;JWT_PRIVATE_KEY&gt;                                                                                                    |
| PUSH_CREDENTIALS                      | Push notification credentials           | &lt;PUSH_CREDENTIALS&gt;                                                                                                   |
| DEMO_PASSWORD                         | Password of the demonstration profiles  | &lt;DEMO_PASSWORD&gt;                                                                                                      |

### 4.7.2 Security policies

The policy tables are created empty during the migration, so loading them is mandatory:

**Table 17**. Parameters of the security policies

| Parameter                      | Content                                             | Verification                                           |
| ------------------------------ | --------------------------------------------------- | ------------------------------------------------------ |
| Password length                | minimum and maximum length agreed with the client   | Out-of-range registration rejected                     |
| Uppercase, numbers and symbols | Requirement active or inactive per rule             | Registration without the requirement rejected when active |
| Expiration in days             | Validity declared in the password policy            | Mandatory change upon expiry                           |
| Security settings              | Parameter–value pairs for session, attempts and lockout | The values are reflected in the application        |
| Origins and rate limit         | Configuration declared at the edge                  | Verified in step 4.4                                   |

### 4.7.3 Catalogs and reference data

The migration loads demonstration data that must be reviewed before real operation:

**Table 18**. Catalogs and reference data loaded

| Catalog                  | Loaded content                                           | Required action                               |
| ------------------------ | -------------------------------------------------------- | --------------------------------------------- |
| Roles and permissions    | The five roles of the model and permissions per role     | Purge the additional demonstration roles       |
| People and profiles      | Demonstration population per role with password hash     | Reset the passwords before operation          |
| Cities                   | Reference cities with country and state                  | Confirm or replace with the real catalog      |
| Institutions and campuses | Example institutions, campuses and courses              | Replace with the real institution             |
| Fleet and devices        | Makes, models, buses and geolocation devices             | Replace with the real fleet                   |
| Routes and stops         | Example routes, stops, schedules and executions          | Replace with the real routes                  |
| Alert types              | Twenty alert types with urgency level                    | Adjust the taxonomy if requested              |
| Demonstration sessions   | Session records with example IP address                  | Delete in production                          |

### 4.7.4 Seed users per role

**Table 19**. Seed users per role

| Role        | State after the migration                                | Initial configuration action                                                                                                     |
| ----------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| SUPER_ADMIN | The role exists in the catalog; there is no associated profile | Create the platform administrator profile with &lt;ADMIN_EMAIL&gt; and &lt;ADMIN_PASSWORD&gt; and verify its audit permission |
| ADMIN       | School administrator profiles with an associated person  | Verify the assigned campus and reset the initial password                                                                        |
| DRIVER      | Demonstration profiles with an associated license        | Reset passwords and confirm the driver–bus assignment                                                                            |
| PARENT      | Demonstration profiles with a family link                | Reset passwords and verify the guardian–student relationship                                                                     |
| STUDENT     | Demonstration profiles with a linked QR identification   | Verify the linkage with campus and route                                                                                         |

curl -i -X POST <http://localhost:8000/api/v1/auth/login> \\

\-H "Content-Type: application/json" \\

\-d '{"email":"&lt;initial_admin&gt;@example.com","password":"&lt;ADMIN_PASSWORD&gt;"}'

Closing: each of the five roles logs in, the permissions correspond to the role and the initial password forces a change on first access.

### 4.7.5 Closing of the initial configuration

The applied configuration is recorded with date, owner and environment; the provisional development credentials are rotated if the environment is testing or production; it is confirmed that no secret remains present in the published artifacts or in the logs; and, finally, the full verification of section 5 is executed.

# 5 Post-deployment verification

## 5.1 Verdict criterion

- **PASS**: the check is executed and meets the stated criterion.
- **FAIL**: it is not executed, does not meet the criterion, or its result is not reproducible.
- The deployment is declared complete only with **PASS** on all items; a **FAIL** triggers section 7 or the reopening of the corresponding stage.
- Each item records evidence: command output, response or log reference.

## 5.2 Verification checklist

**Infrastructure and data**

**Table 20**. Infrastructure and data verification checklist

| #   | Check                  | Criterion                                           | Verdict   |
| --- | ---------------------- | --------------------------------------------------- | --------- |
| I-1 | Data engine            | Healthy status and port 1433 only on the internal network | PASS/FAIL |
| I-2 | Initialization         | school-guardian database created                    | PASS/FAIL |
| I-3 | Broker                 | Six topics listed and initialization with code 0    | PASS/FAIL |
| I-4 | Edge                   | Health response on port 8000                        | PASS/FAIL |
| I-5 | Ports                  | No domain service exposed to the outside            | PASS/FAIL |
| B-1 | Schemas and tables     | 11 schemas and 38 tables present                    | PASS/FAIL |
| B-2 | Migrations             | Zero pending changesets and no active lock          | PASS/FAIL |
| B-3 | Referential integrity  | Foreign keys between schemas created                | PASS/FAIL |
| B-4 | Reference data         | Catalogs loaded and reviewed                        | PASS/FAIL |
| B-5 | Initial backup         | Full and log backup verified                        | PASS/FAIL |

**Domain services**

**Table 21**. Domain services verification checklist

| #   | Check          | Criterion                                                | Verdict   |
| --- | -------------- | -------------------------------------------------------- | --------- |
| S-1 | Availability   | The nine services respond to their health endpoint        | PASS/FAIL |
| S-2 | Ports          | Each service publishes only on 8080                       | PASS/FAIL |
| S-3 | gRPC channel   | Channel available on port 5001                            | PASS/FAIL |
| S-4 | Events         | Production and consumption of a test event                | PASS/FAIL |
| S-5 | Isolation      | Each service operates on its own schema                   | PASS/FAIL |
| S-6 | Logs           | Structured records with correlation identifier            | PASS/FAIL |

**Gateway and clients**

**Table 22**. Gateway and clients verification checklist

| #   | Check                 | Criterion                                                       | Verdict   |
| --- | --------------------- | --------------------------------------------------------------- | --------- |
| G-1 | Routes                | One declared service per domain under /api/v1                   | PASS/FAIL |
| G-2 | Authorization         | Protected resource denied without a valid token                 | PASS/FAIL |
| G-3 | CORS and rate limit   | Published origin and limit demonstrated with test requests      | PASS/FAIL |
| G-4 | Web application       | Loads over HTTPS, without CORS violations and with a menu per role | PASS/FAIL |
| G-5 | Mobile application    | Satisfactory QR scanning, tracking and test notification        | PASS/FAIL |

**Security, configuration and operation**

**Table 23**. Security, configuration and operation verification checklist

| #   | Check                   | Criterion                                                         | Verdict   |
| --- | ----------------------- | ----------------------------------------------------------------- | --------- |
| C-1 | Secrets                 | No real value in artifacts or in logs                             | PASS/FAIL |
| C-2 | Password policies       | Length, uppercase, numbers, symbols and expiration operational    | PASS/FAIL |
| C-3 | Seed users              | The five roles authenticate with the correct permissions          | PASS/FAIL |
| C-4 | Audit                   | Critical action recorded with who, what, when, IP and application | PASS/FAIL |
| O-1 | Dashboards              | Availability, P95 latency, error rate, load and instances         | PASS/FAIL |
| O-2 | Alerts                  | Rules active and test notification received                       | PASS/FAIL |
| O-3 | Service objectives      | 99.9 % availability and P95 under 300 ms instrumented             | PASS/FAIL |
| O-4 | Support                 | Attention and escalation procedures defined                       | PASS/FAIL |

## 5.3 Evidence log and annex

The quality owner completes one row per item with identifier, date, environment, owner, check result, verdict and evidence reference. The log is kept with the acceptance record and feeds the management review.

# 6 Versioning and release publication

## 6.1 Versioning scheme

**Table 24**. Artifact versioning scheme

| Artifact                        | Scheme                                 | Rule                                                                                                               |
| ------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Platform (set version)          | MAJOR.MINOR.PATCH                      | MAJOR for incompatible contract or schema changes; MINOR for compatible functionality; PATCH for fixes             |
| Container image                 | Immutable tag tied to the version      | A published tag is never reused                                                                                    |
| API contract                    | /api/v1 prefix                         | An incompatible change requires a new contract version                                                             |
| Broker events                   | Additive versioning                    | Consumers tolerate unknown fields                                                                                  |
| Migrations                      | Changesets with a unique identifier    | An already applied changeset is never edited                                                                       |
| Mobile application              | Version number visible to the user     | Incremented on every distribution                                                                                  |

## 6.2 Branch and approval flow

main ← production; only receives release integrations; always stable

└── dev ← continuous integration; receives the feature branches

├── feature/\[description\] ← one user story

├── fix/\[description\] ← one fix

├── chore/\[description\] ← infrastructure, dependencies, documentation

└── hotfix/\[description\] ← urgent fix directly to production

**Figure 2**. Branch and approval flow

Rules: no direct integration into protected branches; one task per branch; review with at least one approval; recommended maximum size of 400 lines of code excluding tests; integration messages in conventional format (feat, fix, docs, refactor, test, chore, perf).

## 6.3 Content of a release

1. Assigned version and corresponding image tag.
2. List of user stories and fixes included.
3. Schema changes applied and their rollback status.
4. Edge configuration changes.
5. Result of the post-deployment verification.
6. Rollback instructions in force for that version.
7. Entry in the deployment history: date, service, version, environment, owner and incidents.

## 6.4 Release windows

**Table 25**. Release windows per environment

| Environment | Window                                                                                     | Approval                                                    |
| ----------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| Integration | Continuous after each approved integration                                                 | Automatic if all gates pass                                 |
| Testing     | Continuous after merging into the main branch                                              | Automatic, with notice to the product owner                 |
| Production  | Window agreed with the institution, avoiding the school transportation service hours        | Technical lead and, for major changes, double approval      |

# 7 Rollback procedure

## 7.1 Triggers

**Table 26**. Rollback triggers and associated decision

| Trigger                            | Activation condition                                          | Decision                                         |
| ---------------------------------- | ------------------------------------------------------------- | ------------------------------------------------ |
| Failed post-deployment verification | At least one item in FAIL after two correction attempts       | Full rollback to the previous version            |
| Degraded canary deployment         | Metrics out of threshold during the 15-minute observation     | Automatic rollback                               |
| High error rate                    | Above 5 % for 5 minutes                                       | Immediate rollback                               |
| Degraded latency                   | P95 above 500 ms for 5 minutes                                | Rollback or mitigation depending on scope        |
| Service down                       | Failed health check for more than 1 minute                    | Restart and, if it persists, rollback            |
| Incompatible migration             | Schema error that prevents operation                          | Stop and data rollback after prior approval      |
| P0 or P1 incident                  | Critical severity classification                               | Decision of the incident commander               |

## 7.2 Rollback steps

1. The incident is declared and its severity classified.
2. The ongoing rollback is communicated to the stakeholders.
3. The release is stopped and the related changes are frozen.
4. The previous version of the images of the affected services is restored.
5. If the edge configuration changed, the previous declared manifest is restored.
6. If the web application was published, the previous artifact is restored and the cache is cleared.
7. If the mobile application was distributed, the channel is withdrawn and the version to be used is communicated.
8. The verification of section 5 is executed on the affected items.
9. The incident is closed with root cause and improvement actions.

kubectl rollout undo deployment/&lt;service&gt; -n &lt;namespace&gt;

kubectl rollout status deployment/&lt;service&gt; -n &lt;namespace&gt;

In container-based deployments without an orchestrator, the rollback consists of re-deploying the previous image tag and restarting the service set.

## 7.3 Database rollback

1. Writes from the affected services are suspended.
2. It is confirmed that the change has a declared rollback.
3. The rollback of one change is executed in accordance with section 4.2.5.
4. If no declared rollback exists, the most recent previous backup is restored on a standby instance, the connection configuration is pointed over in a controlled window, and it is verified schema by schema: identity login, school reading, latest geolocation positions and audit annex.
5. The decision on the broker event window is documented: replay from the checkpoint or accept the gap.
6. The operation is recorded in the incident report.

**Table 27**. Backup methods and recovery points

| Asset                               | Backup method                                | Recovery point        | Recovery time          | Owner                             |
| ----------------------------------- | -------------------------------------------- | --------------------- | ---------------------- | --------------------------------- |
| SQL Server: full and log            | Native or managed backup                     | ≤ 15 min              | ≤ 2 h                  | Platform and data owner           |
| Migration changelogs                | Code version history                         | 0                     | Minutes                | All services                      |
| Broker retention                    | Per-topic retention and mirror if required   | Per topic             | Per topic              | Platform                          |
| Edge configuration                  | Versioned declarative export                 | 0                     | Minutes                | Edge                              |
| Ephemeral store                     | No restore requirement                       | —                     | Recreation             | Edge                              |

## 7.4 Post-rollback verification

**Table 28**. Post-rollback verification

| #   | Check         | Criterion                                                  | Verdict   |
| --- | ------------- | ---------------------------------------------------------- | --------- |
| R-1 | Services      | The affected services respond to their health endpoint      | PASS/FAIL |
| R-2 | Database      | Consistent migrations and no pending changesets            | PASS/FAIL |
| R-3 | Edge          | Routes and policies of the previous version active         | PASS/FAIL |
| R-4 | Clients       | Web and mobile application operate against the restored version | PASS/FAIL |
| R-5 | Metrics       | Error rate and latency within threshold                    | PASS/FAIL |
| R-6 | Log           | Incident documented with root cause and actions            | PASS/FAIL |

# 8 Continuous integration and publication channel

## 8.1 Stages of the channel

**Table 29**. Stages of the continuous integration and publication channel

| Order | Stage                          | Trigger                                        | Target duration | Content                                                                                                                                                        |
| ----- | ------------------------------ | ---------------------------------------------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Change request                 | Any branch update                              | < 2 min         | Style analysis, contract validation and detection of incompatible changes; if it fails, the merge is blocked                                                  |
| 2     | Code quality                   | Changes in the code of a service               | < 5 min         | Unit tests, build and static security analysis                                                                                                                 |
| 3     | Data change validation         | Changes in changelogs                          | < 3 min         | Changelog validation and SQL preview; if it fails, the merge is blocked                                                                                        |
| 4     | Deployment to integration      | Approved merge into the development branch     | < 15 min        | Integration tests, image build, deployment and smoke check on port 8000                                                                                        |
| 5     | Deployment to testing          | Merge into the main branch                     | < 45 min        | Full suite, contracts, image tagging, deployment, smoke check, performance and notice to the product owner                                                     |
| 6     | Production                     | Manual approval after passing all gates        | Variable        | Migration before the code, canary deployment to 10 % and 15-minute observation: satisfactory → 100 % of traffic; unsatisfactory → automatic rollback            |

## 8.2 Quality gates

**Table 30**. Quality gates and effect of their failure

| Gate                       | Pass condition                                                                  | Effect if it fails         |
| -------------------------- | ------------------------------------------------------------------------------- | -------------------------- |
| Style and format           | No findings                                                                     | Merge blocked              |
| Contracts                  | Satisfactory validation and no undeclared incompatible changes                  | Merge blocked              |
| Unit tests                 | 100 % satisfactory; line coverage ≥ 80 % and ≥ 90 % in the domain layer         | Merge blocked              |
| Integration tests          | All satisfactory                                                                | Deployment stopped         |
| Data changes               | Satisfactory changelog validation                                               | Merge blocked              |
| Smoke                      | Edge health response and minimum walkthrough                                    | Deployment stopped         |
| Performance                | P95 under 300 ms and error rate below 1 %                                       | Release blocked            |
| Post-deployment verification | All items in PASS                                                             | Rollback triggered         |

The coverage and performance thresholds come from the project quality strategy; when a threshold is not formalized, it is proposed as an acceptance criterion and requires approval before being used as a gate.

## 8.3 Canary release in production

**Table 31**. Steps of the canary release in production

| Step | Action                                                          | Continuation criterion                   |
| ---- | --------------------------------------------------------------- | ---------------------------------------- |
| 1    | Formal approval by the technical lead                           | Release approved                         |
| 2    | Verification of the version's tests, including performance      | All gates in PASS                        |
| 3    | Approval of the functional scope by the product owner           | Included stories accepted                |
| 4    | Database migration before the code                              | Satisfactory migration verification      |
| 5    | Deployment to 10 % of traffic                                   | Metrics within threshold for 15 minutes  |
| 6    | Expansion to 100 % of traffic                                   | No active alerts of severity P0 or P1    |
| 7    | Intensive observation during the first 30 minutes               | On-call owner available                  |
| 8    | Recording of the release                                         | Deployment history updated               |

# 9 Monitoring and post-implementation support

## 9.1 Service level objectives

**Table 32**. Service level objectives

| Indicator                  | Objective                         | Measurement                          | Error budget                                  |
| -------------------------- | --------------------------------- | ------------------------------------ | --------------------------------------------- |
| Availability               | 99.9 %                            | Successful requests in 30 days       | 43.8 minutes of unavailability per month      |
| API latency                | P95 under 300 ms                  | Request duration histogram           | Alert above 500 ms for 5 minutes              |
| Error rate                 | Below 1 % under load              | Failed over total                    | Alert above 5 % for 5 minutes                 |
| Notification delivery      | Notification delivery rate        | Delivered over emitted               | Defined with the institution                  |
| Boarding record            | Record without incidents          | Boardings recorded without error     | Defined with the institution                  |

## 9.2 Metrics, logs and traces

**Table 33**. Observability pillars and associated tool

| Pillar    | Content                                                                                                                | Tool                                             |
| --------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Logs      | Structured JSON with timestamp, level, service, version, environment, correlationId, userId, message and context       | Centralized collection with index-based query   |
| Metrics   | http_requests_total, http_request_duration_seconds, http_requests_in_flight, business and data connection metrics      | Prometheus and Grafana dashboards               |
| Traces    | Trace identifier propagation between the edge and services, with automatic instrumentation of data access              | OpenTelemetry with a trace console              |

The metrics exposure endpoint (GET /metrics) is kept only on the internal network and is not published at the edge. No passwords, full tokens or unmasked personal data are logged. The required dashboards are: executive (error rate, P95 latency per service, load, availability against the objective and instances) and per service (rate, error and duration metrics, runtime, data connection and broker metrics).

## 9.3 Alert rules

**Table 34**. Alert rules and associated severity

| Alert                 | Condition                                     | Severity | Destination                   |
| --------------------- | --------------------------------------------- | -------- | ----------------------------- |
| High error rate       | Above 5 % for 5 minutes                       | P1       | On-call and incident channel  |
| High latency          | P95 above 500 ms for 5 minutes                | P2       | Alert channel                 |
| Service down          | Failed health check for more than 1 minute    | P1       | On-call                       |
| Unconsumed messages   | Age out of threshold                          | P2       | Alert channel                 |
| Low disk space        | Usage above 80 %                              | P2       | Alert channel                 |
| Frequent restarts     | More than three restarts in 10 minutes        | P1       | On-call                       |

## 9.4 Incident management

**Table 35**. Incident management levels

| Level | Definition                                                   | Response            | Resolution |
| ----- | ------------------------------------------------------------ | ------------------- | ---------- |
| P0    | System fully down with total impact                          | Immediate (< 5 min) | 1 hour     |
| P1    | Critical functionality degraded with majority impact         | 15 minutes          | 4 hours    |
| P2    | Secondary functionality degraded with an available alternative | 1 hour            | 24 hours   |
| P3    | Minor inconvenience with no operational impact               | 4 hours             | 1 week     |

Attention sequence: detection and acknowledgment of the alert → severity classification → initial communication with the affected service, impact and next update → diagnosis with dashboards, logs, traces, container status and manual health check → mitigation starting with the safest option (rollback, disabling the functionality, escalation, increasing timeouts) → resolution once the metrics recover their levels, the health checks respond and the product owner confirms operation → retrospective within the next two business days, with root cause and actions.

## 9.5 Backups and recovery

**Table 36**. Frequency of backup and restore verification

| Verification                                         | Frequency          |
| ---------------------------------------------------- | ------------------ |
| Success of the backup job                            | Daily, with alert  |
| Restore exercise to a point in time                  | Quarterly          |
| Integrity of the changelogs in the restored copy     | Every exercise     |

Principles: the full instance is backed up and the recovery owners per schema are documented; the edge configuration and secrets are part of the backup set; a backup that has never been restored is not a backup. The restore involves declaring the severity, identifying the most recent full backup and the subsequent logs, restoring on a standby instance, changing the connection configuration in a controlled window, verifying schema by schema, deciding the event replay window and closing with a retrospective.

## 9.6 Maintenance and scaling

- **Maintenance windows**: scheduled with the institution outside the transportation service hours; they are communicated in advance and limited to approved changes.
- **Updates**: the order of section 6.3 is followed —migration, services, edge configuration, smoke— with verification at the close of each stage.
- **Horizontal scaling**: the services are stateless and scale behind the edge; routes and notifications scale by event volume, and the read ones by number of requests.
- **Error budget**: once the availability objective is exhausted, fixes are prioritized over new functionality.
- **Post-implementation support**: accompaniment period with a defined on-call rotation, weekly review of alerts and status report at the close of each week.

# 10 RACI matrix and deployment schedule

## 10.1 Roles of the deployment team

**Table 37**. Roles of the deployment team

| Role                         | Main responsibility                                       |
| ---------------------------- | --------------------------------------------------------- |
| Technical lead               | Technical direction, approvals and rollback decisions     |
| Platform engineer            | Infrastructure, services, edge and clients                |
| Database owner               | Migrations, verification, backups and recovery            |
| Quality owner                | Post-deployment verification and issuing of verdicts      |
| Operations owner             | Monitoring, alerts, incidents and maintenance             |
| Client (institution)         | Data definition, observations and acceptance              |
| Academic advisor             | Accompaniment and review of the procedure                 |

## 10.2 RACI matrix

R: responsible for execution · A: ultimately accountable · C: consulted · I: informed

**Table 38**. RACI matrix of the deployment activities

| Activity                                  | Lead | Platform | Data | Quality | Operations | Client | Advisor |
| ----------------------------------------- | ---- | -------- | ---- | ------- | ---------- | ------ | ------- |
| Prerequisite validation                   | A    | R        | C    | C       | I          | I      | I       |
| Base infrastructure                       | A    | R        | C    | I       | I          | I      | I       |
| Migrations and schema verification        | A    | C        | R    | C       | I          | I      | I       |
| Deployment of domain services             | A    | R        | C    | I       | I          | I      | I       |
| Gateway and routing                       | A    | R        | I    | C       | C          | I      | I       |
| Publication of the web application        | A    | R        | I    | C       | I          | I      | I       |
| Generation and distribution of the mobile app | A | R       | I    | C       | I          | C      | I       |
| Initial configuration and seed users      | A    | C        | C    | I       | R          | C      | I       |
| Post-deployment verification              | A    | C        | C    | R       | C          | I      | I       |
| Rollback upon incidents                   | A    | R        | R    | C       | C          | I      | I       |
| Monitoring and incident handling          | A    | C        | I    | I       | R          | I      | I       |
| Acceptance and closure                    | A    | C        | I    | C       | C          | R      | C       |

## 10.3 Schedule

Planning estimates of duration; contractual commitments are established with the client.

**Table 39**. Estimated deployment schedule

| Phase                     | Central activity                                      | Estimated duration | Dependency          | Outcome                   |
| ------------------------- | ----------------------------------------------------- | ------------------ | ------------------- | ------------------------- |
| 1 Preparation             | Prerequisites, secrets and accesses                   | 2 working days     | Approved baseline   | Environment enabled       |
| 2 Base infrastructure     | Engine, broker, ephemeral store and edge              | 1 working day      | Phase 1             | Healthy infrastructure    |
| 3 Database                | Migrations of the 11 schemas and verification         | 2 working days     | Phase 2             | 38 tables available       |
| 4 Domain services         | Build, startup and health of the nine services        | 2 working days     | Phase 3             | Services operational      |
| 5 Gateway                 | Routes, policies and routing verification             | 1 working day      | Phase 4             | Traffic routed            |
| 6 Clients                 | Web publication and mobile generation and distribution | 2 working days    | Phase 5             | Applications available    |
| 7 Initial configuration   | Variables, policies, catalogs and seed users          | 1 working day      | Phase 6             | System parameterized       |
| 8 Verification            | Full checklist with verdict                           | 2 working days     | Phase 7             | PASS verdict              |
| 9 Acceptance              | Demonstration, observations and signature             | 1 working day      | Phase 8             | Signed acceptance record  |
| 10 Support                | Post-implementation accompaniment                     | 30 days            | Phase 9             | Support closure           |

Estimated total: 14 deployment working days plus 30 days of subsequent support.

## 10.4 Milestones and acceptance gates

**Table 40**. Milestones and acceptance gates

| Milestone                  | Crossing gate                                      | Evidence                         |
| -------------------------- | -------------------------------------------------- | -------------------------------- |
| M1 — Infrastructure ready  | Health of engine, broker and edge                  | Check outputs                    |
| M2 — Data available        | No pending changesets; 11 schemas and 38 tables    | Change history query             |
| M3 — Services operational  | Health of the nine services and gRPC channel       | Check outputs                    |
| M4 — Traffic routed        | Denial without a token and routing with a token    | Check outputs                    |
| M5 — Clients published     | Satisfactory web and mobile walkthrough            | Walkthrough log                  |
| M6 — System parameterized  | Policies and seed users operational                | Configuration log                |
| M7 — Acceptance            | Verification in PASS and signature of the record   | Acceptance record                |

# 11 Change management and acceptance

## 11.1 Classification of changes

**Table 41**. Classification of changes

| Type       | Scope                                              | Example                                          | Approval                                            |
| ---------- | -------------------------------------------------- | ------------------------------------------------ | --------------------------------------------------- |
| Standard   | Pre-approved and reversible                        | Log level adjustment, secret rotation            | Platform owner                                      |
| Major      | Affects schema, contract or edge configuration     | New column, new public route                     | Technical lead and database owner                   |
| Emergency  | Fixes an ongoing incident                          | Rollback of a defective version                  | Incident commander, with subsequent ratification    |

## 11.2 Approval flow

1. **Request**: identification of the change, requester, reason, affected environment and urgency.
2. **Impact assessment**: components involved, risk, need for migration and rollback plan.
3. **Approval**: according to the classification in section 11.1.
4. **Preparation**: prior execution in the testing environment with its verification.
5. **Execution**: in the authorized window, following section 4 and the order migration → services → edge configuration → smoke.
6. **Verification**: checklist of section 5 scoped to what was modified.
7. **Closure**: updated log, communication to the parties and, if applicable, update of the rollback instruction.

## 11.3 Change log

**Table 42**. Fields of the change log

| Field                     | Content                                              |
| ------------------------- | ---------------------------------------------------- |
| Identifier and date       | Unique sequence number and request date              |
| Description and requester | What changes, why and who requests it                |
| Classification            | Standard, major or emergency                         |
| Affected components       | Service, schema, edge or client                      |
| Associated migration      | Yes/no and its rollback status                       |
| Approver and window       | Owner who authorizes and authorized date             |
| Verification              | Result of the applicable checklist                   |
| Available rollback        | Yes/no and its procedure                             |
| Status                    | Requested, approved, executed, verified, closed      |

## 11.4 Acceptance criteria

**Table 43**. Acceptance criteria and their verification

| #    | Criterion                                                                   | Verification                                      |
| ---- | --------------------------------------------------------------------------- | ------------------------------------------------- |
| A-1  | All checklist items in PASS status                                          | Signed verification log                           |
| A-2  | 11 schemas and 38 tables with no pending changesets                         | Change history query                              |
| A-3  | The nine services operate and respond to their health                       | Check outputs                                     |
| A-4  | Single public entry point: gateway on port 8000                             | Port inventory                                    |
| A-5  | Web application published over HTTPS with a menu per role                   | Acceptance walkthrough                            |
| A-6  | Mobile applications installed with functional QR scanning and notifications | Walkthrough per role                              |
| A-7  | The five roles authenticate with the corresponding permissions              | Session log per role                              |
| A-8  | Password policies and security settings operational                         | Registration and expiration tests                 |
| A-9  | 99.9 % availability and P95 under 300 ms instrumented                        | Dashboards and alert rules                        |
| A-10 | Backups with recovery ≤ 15 min and ≤ 2 h                                    | Backup log and restore exercise                   |
| A-11 | Rollback instructions available and tested                                  | Rollback test log                                 |
| A-12 | Operations personnel trained in incident handling                           | Training log                                      |

## 11.5 Acceptance record and closure

The record contains: identification of the platform and of the deployed version, environment, date, verified scope, checklist result with its overall verdict, observations with their attention plan, owners of the deployment and of the operation, and the subsequent support commitment.

**Table 44**. Fields of the acceptance record

| Field                     | Content                                          |
| ------------------------- | ------------------------------------------------ |
| Deployed version          | Set version and corresponding images             |
| Deployment date           | Closing date of the verification                  |
| Overall verdict           | PASS / FAIL                                       |
| Client observations       | List with owner and committed date                |
| Signatures                | Technical lead and client                         |
| Committed support         | Period, attention channel and response times      |

If the overall verdict is FAIL, the deployment does not close: section 7 is triggered or the corresponding stage is reopened, and acceptance is scheduled again when all items are in PASS. Once signed, the system moves to operation in accordance with section 9, and subsequent changes are managed in accordance with section 11.
