# Unit Testing Document — School Guardian (School Transportation Platform)

| Field | Detail |
|---|---|
| **Project** | School Guardian — School transportation platform (microservices + web application) |
| **Version** | 1.0 |
| **Date** | October 10, 2026 |
| **Prepared by** | Juan Pablo Chala Ramírez · Johan Smith Santamaria Fernández · Sharik Dayanna Rojas Ibarra |
| **Program** | Software Analysis and Development Technologist (ADSO), group 3145556 |
| **Training center** | Centro La Industria, La Empresa y Los Servicios (SENA) |

---

## 1. Introduction

### 1.1 Purpose and scope

This document defines and documents the **unit tests** for the School Guardian project, a school transportation platform composed of a microservices architecture for the backend and a web application (frontend) developed in Angular.

The scope of this document is limited exclusively to **unit tests**: the isolated verification of business logic, validations, calculations, domain rules, mappings, application services, queries, and interface components, executed without access to a database, network, or file system. Integration tests, end-to-end (E2E) tests, and performance tests are out of scope and will be documented in separate deliverables.

**Technical scope (based on the project's real code):**

| Component | Stack | Projects/services | Test projects |
|---|---|---|---|
| Backend — .NET microservices | C# / .NET 10 (`net10.0`), **NUnit** | `ms-configuration`, `ms-exceptional`, `ms-fleet`, `ms-route`, `ms-school-management`, `ms-user-management`, `sg-ms-forgot-information` | Yes, one per service in `tests/{service}.Tests` |
| Backend — IAM | Java 21, Spring Boot 4.1.1, Maven, JUnit 5 + Mockito | `ms-iam` | Yes (`src/test/java`) |
| Backend — notifications | C# / .NET 10, **NUnit** | `ms-notification` | **Does not exist** (audit pending) |
| Gateway | Kong (declarative configuration `kong.yml`) | `ms-api-gateway` | Not applicable (no code) |
| Messaging infrastructure | Kafka (Docker Compose, topics) | `kafka` | Not applicable (no code) |
| Frontend | Angular 21, TypeScript 5.9, Vitest 4 + jsdom | Web application `dvlp-web` | Yes (64 `*.spec.ts` files) |

**Functional modules covered** (cross-referenced against the requirements report):

- **RF-01 User management**: creation, update, activation/deactivation of accounts by role, and linking students to guardians (`ms-user-management`, `ms-school-management`).
- **RF-02 Authentication**: sign-in and sign-out, token issuance and renewal, credential recovery (`ms-iam` and `sg-ms-forgot-information` flows).
- **RF-03 School routes, fleet, and assignments**: routes with ordered stops and schedules, buses, drivers, assignments (`ms-route`, `ms-fleet`).
- **RF-04 / RF-05 QR scanning and boarding notifications**: boarding and alighting control and automatic notification to guardians (`ms-notification`; integration with the mobile frontend).
- **RF-07 Trip management**: start and end of the trip lifecycle (`ms-route`).
- **RF-09 Route lookup**: guardian and driver queries.
- **RF-10 Personalization**: color theme and languages (frontend).

### 1.2 Audience

| Audience | Expected use |
|---|---|
| Backend and frontend developers | Know what to test, conventions, and how to run the tests |
| QA team | Run, extend, and maintain the test suite |
| Technical lead | Evaluate compliance with quality and coverage criteria |

### 1.3 Definitions and acronyms

| Term | Definition |
|---|---|
| **Unit test** | A test that verifies a minimal unit of behavior (method or function) in isolation, without real external dependencies. |
| **Unit** | In this project: a use case (`*Service`) from the `Application` layer, a domain rule (`*Rules`), a search strategy (`*SearchStrategy`), a security filter, or a frontend service/component. |
| **Mock** | An object that records expected calls and returns programmed responses, allowing interactions to be verified (e.g., `mock(ProfileJpaRepository.class)` in `ms-iam`). |
| **Stub** | An object that returns fixed data so that the system under test (SUT) can run; it does not verify interactions. |
| **Fake** | A lightweight, functional implementation of a dependency (e.g., `InMemoryBusRepository`) used as a real substitute in the test. |
| **Fixture** | A set of data or initial state prepared for a test. |
| **Coverage** | Percentage of code (lines, branches, or conditions) executed by the test suite. |
| **SUT** | *System Under Test*: the unit being tested. |
| **AAA** | Arrange-Act-Assert: a pattern for structuring a test (arrange, act, assert). |
| **EC / BVA** | Equivalence partitioning / Boundary value analysis: test case design techniques. |
| **TDD** | *Test-Driven Development*. |

### 1.4 References

| Reference | Document/artifact |
|---|---|
| Requirements | Software Requirements Specification Report — document 01 of the documentation repository (requirement groups RF-01 to RF-10). |
| Software design | Document 04 — Software Design (architecture, graphical interfaces, data model). |
| Code artifacts | `.csproj`/`pom.xml`/`package.json` projects, classes of the `Application`/`Domain`/`Infrastructure` layer of each microservice, data schema (`db-school-guardian.dbml` ↔ DDL), frontend sources, and their `.spec.ts` specifications. |

### 1.5 Version control

| Version | Date | Description of changes | Author |
|---|---|---|---|
| 1.0 | 2026-10-10 | Initial version: definition of the strategy and unit test case design based on the project's real code. | ADSO Team |

---

## 2. Unit testing strategy

### 2.1 Objectives

1. Verify the **business logic** of the use cases without access to a database, network, or file system.
2. Ensure that the **validations** (uniqueness, ranges, formats, schedules) reject invalid data and accept valid data.
3. Cover sensitive **calculations and domain rules**: generation and expiration of OTP codes, trip lifecycle, boarding/alighting control, password policies, and data masking.
4. Verify the backend's **application services**, **per-tenant searches/configurations**, and **security filters**.
5. Verify in the frontend the **HTTP services (network mocks)**, **error handling (ProblemDetails)**, and the **behavior of pages and components**.
6. Serve as a **regression net** against requirement changes and refactorings.

### 2.2 What is tested

| Category | Concrete units in the project |
|---|---|
| Business logic (use cases) | `CreateBusService`, `AssignDriverToBusService`, `CreateRouteService`, `StartTripService`, `EndTripService`, `AssignStudentToRouteService`, `CreateStudentService`, `RegisterFamilyService`, `CreateSchoolWithCampusesService`, `ProcessScanService`, `LoginService`, `ForgotPasswordService`, etc. |
| Validations and rules | `FamilyRules`, `DriverLicenseRules`, `SchoolCampusRequestRules`, `SchoolLocationRequestRules`, `OtpOptions`, `PersonRequestDto` validations, plate/identification/license uniqueness. |
| Calculations | OTP generation (`OtpCodeGenerator`), code and token expiration (`VerificationCodeService`, `JwtTokenProvider`), masking (`EmailLogMask`), `SecretHasher`, trip summaries, and driver queries (`GetDriverRouteTodayService`). |
| Searches and filters | `NameSearchStrategy`, `EmailSearchStrategy`, `IdentificationSearchStrategy`, `PhoneSearchStrategy`, `PlateSearchStrategy`, `TenantPersonFilter`, `RoleFilteredList`, per-tenant lists. |
| Mappings | `PersonProfile` (AutoMapper), `StudentScannedEventMapper`, bus/view DTOs (`BusViewFieldsTests`). |
| Security (backend) | `JwtTokenProvider`, `InternalApiKeyFilter`, `JwtAuthenticationFilter`, `GlobalExceptionHandler`, `AuthExceptionHandler`. |
| Context infrastructure | EF model mapping tests (`SettingsContextTests`, `ExceptionalContextTests`) on a context without a real connection. |
| Frontend — services | `AuthService`, `BusesService`, `RoutesService`, `StopsService`, `StudentsService`, `ForgotInformationService`, `ThemeContextService`, `problem-detail` parsing. |
| Frontend — components and pages | Administration pages (buses, routes, stops, drivers, families, guardians, students), dashboards, profile flows (change contact/email/password), and shared components. |

### 2.3 What is not tested

| Category | Reason |
|---|---|
| Generated code | EF migrations, Angular schematics-generated code, template code. |
| Trivial getters/setters | No logic; they add noise without verification value. |
| Configuration | `Program.cs`, `ServiceCollectionExtensions`, `appsettings`, `*Config` classes, `SecurityConfig`, `OpenApiConfig`. |
| Third-party libraries | NUnit, EF Core, Spring Security, JWT libs, Angular Material, Leaflet, RxJS. |
| Kafka broker and topics | Infrastructure; its consumption is simulated in the consuming unit. |
| Kong gateway | Declarative configuration without code (validated with `kong check`, not with unit tests). |
| Visual interface/styles | Covered in E2E and visual review tests, outside this document. |

### 2.4 White-box approach with dependency isolation

The tests follow a **white-box approach**: they are written from the real internal structure of the classes (use cases, strategies, filters), and statement, branch, and condition coverage is measured. To isolate the system under test, **every external dependency is simulated**:

- EF Core repositories → **in-memory fakes** (`InMemoryBusRepository`, `InMemoryPersonRepository`, `InMemoryConfigurationRepositories`, etc.).
- System clock → **controlled time providers** (`ManualTimeProvider` in `sg-ms-forgot-information`, `FixedTimeProvider` in `ms-route`).
- HTTP (IAM API / identity directory) → fake HTTP clients (`FakeIdentityDirectoryClient`) and `HttpTestingController` in the frontend.
- SMTP → simulated sender (`FakeNotificationSenders`).
- JPA repositories and `PasswordEncoder` → **Mockito mocks** in `ms-iam`.
- GPS devices → `FakeGpsDeviceService`.
- Provision of EF contexts: mapping tests with `UseSqlServer("Server=unused;...")` that **do not open a connection**.

### 2.5 Development approach: tests written afterward (current state) with a transition to TDD

The project's current suite was written **after implementation** (characterization tests of existing code). In the frontend, *smoke*-type specifications (e.g., `should create`) are also observed alongside real behavior tests (HTTP services). As of this document:

- The existing tests are **extended and completed** (modules not covered: `ms-notification`, `ms-iam` use cases, `ms-route` trips and stops).
- For **all new functionality**, **TDD** is adopted (red → green → refactor), with the test failing first.
- **Framework for .NET tests: NUnit.** All tests for the .NET microservices are implemented with NUnit (the `NUnit` package + `NUnit3TestAdapter`); the execution and coverage commands are unchanged (see sections 3.1 and 9).

### 2.6 Entry criteria

- The service code compiles.
- The SUT's dependencies can be replaced (constructor injection or port interfaces).
- Fakes/fixtures exist for repositories, clock, HTTP, and messaging.
- No database, network, or file system is required to run (see section 9).

### 2.7 Exit criteria

- **100% of tests passing** and **0 known flaky tests**.
- **Minimum coverage of 80%** of lines and branches in the `Application`/`Domain` layer of each microservice and in the frontend services.
- Every critical validation branch (uniqueness, boundaries, exceptions) exercised at least once.
- The tests do not depend on execution order or on the environment.

---

## 3. Tools and environment

### 3.1 Backend — .NET microservices

| Tool | Detail (verified in the real `.csproj` files) |
|---|---|
| Language / target | C# on **.NET 10** (`<TargetFramework>net10.0</TargetFramework>`) |
| Test framework | **NUnit** (`NUnit` package 4.x) + **`NUnit3TestAdapter`** adapter |
| Test SDK | `Microsoft.NET.Test.Sdk` 17.12.0 (ms-forgot-information), 17.14.1 (ms-configuration, ms-exceptional, ms-school-management), and 18.0.0 (ms-fleet, ms-route, ms-user-management) |
| Mocking | **No mocking libraries** (Moq/NSubstitute/FakeItEasy are not used): the project uses **hand-written fakes** (`InMemory*Repository`, `Fake*Service`, `ManualTimeProvider`/`FixedTimeProvider`) |
| Coverage | `coverlet.collector` 6.0.4 in ms-configuration, ms-exceptional, and ms-school-management; the other projects use the Test SDK's built-in code coverage collector. The collector works the same with NUnit (see section 9) |
| IDE | Visual Studio 2022+, JetBrains Rider, or VS Code with C# Dev Kit |
| CLI | `dotnet` (SDK 10) |

### 3.2 Backend — IAM microservice (ms-iam)

| Tool | Detail (verified in the real `pom.xml`) |
|---|---|
| Language / version | **Java 21** (`<java.version>21</java.version>`) |
| Framework | **Spring Boot 4.1.1** (parent `spring-boot-starter-parent`) |
| Test framework | **JUnit 5 (Jupiter)** via `spring-boot-starter-webmvc-test`, `spring-boot-starter-security-test`, `spring-boot-starter-data-jpa-test`, and `spring-boot-starter-validation-test` |
| Mocking | **Mockito** (real imports in: `InternalProfileServiceTest` with `mock(ProfileJpaRepository.class)` and `mock(PasswordEncoder.class)`) |
| Coverage | **Not configured** (there is no JaCoCo plugin in the `pom.xml`); adding it is recommended (see section 12) |
| Build | Maven (the `mvnw` / `mvnw.cmd` wrapper is present) |
| IDE | IntelliJ IDEA, Eclipse/STS, or VS Code with Java extensions |

### 3.3 Frontend — web application

| Tool | Detail (verified in the real `package.json` and `angular.json`) |
|---|---|
| Framework | **Angular 21.2.x** (`@angular/core` ^21.2.9), Angular Material 21, SSR (`@angular/ssr`), `@ngx-translate` |
| Language | **TypeScript 5.9.2** |
| Test runner | **Vitest 4.0.8** + **jsdom 28** through the `@ngx-env/builder:unit-test` builder (target `test` in `angular.json`; command `ng test`) |
| HTTP mocking | `@angular/common/http/testing` (`HttpTestingController`) |
| Package manager | npm 10.9.1 (`packageManager: "npm@10.9.1"`) |
| IDE | VS Code (with Angular Language Server) or WebStorm |

### 3.4 Environment setup

| Step | Command |
|---|---|
| .NET | `dotnet restore` in each solution/service |
| Java | `./mvnw -q dependency:resolve` in `ms-iam` |
| Frontend | `npm ci` in the application's root directory |

Machine requirements: .NET SDK 10, JDK 21, Node.js 20 LTS+, Docker (for the containerized execution in section 9).

---

## 4. Structure and conventions

### 4.1 Location of the tests in the repository

| Layer | Location |
|---|---|
| .NET backend | `{service}/tests/{service}.Tests/`, with subfolders that **mirror** the structure of `src/{service}.Api/`: `Application/` (use case tests), `Infrastructure/` (persistence context tests), `Fakes/` (shared doubles: in-memory repositories, fake services and time providers), `SmokeTests.cs` |
| Java backend | `ms-iam/src/test/java/...` with the **same package** as the code under test (`com.school_guardian.ms_iam.application.usecase`, `...infrastructure.web.filter`, `...infrastructure.config`) |
| Frontend | `*.spec.ts` files **placed next to** the unit: `src/app/core/services/auth.service.spec.ts`, `src/app/features/admin/routes-buses/pages/buses/buses.spec.ts`, etc. |

### 4.2 Relationship between code classes and test classes

**1-to-N** rule: a code class may have one or several test classes, but a test class always focuses on **a single unit** (use cases are not mixed in the same test class, except for smoke suites). Per-service suites go in `Fakes/` and are shared only within the test project.

### 4.3 Naming convention

| Layer | Convention | Real example |
|---|---|---|
| .NET | `Method_Condition_ExpectedResult` | `ExecuteAsync_WithDuplicatePlate_ShouldThrow` (ms-fleet) |
| Java | Descriptive method in lower camelCase | `lookupNormalizesEmailAndReturnsContract` (ms-iam) |
| Frontend | `describe('unit', ...)` + `it('behavior...')` in narrative form | `describe('AuthService session')` / `it('saves profileId/personId/email from the login response')` |

### 4.4 Structuring pattern

**AAA (Arrange-Act-Assert)**, equivalent to Given-When-Then (**NUnit** syntax):

```csharp
// Arrange
var repo = new InMemoryBusRepository();
var gpsService = new FakeGpsDeviceService();
var useCase = new CreateBusService(repo, gpsService);

// Act
var busId = await useCase.ExecuteAsync(
    Guid.NewGuid(), DateTime.UtcNow.AddYears(1), Guid.NewGuid(),
    40, "ABC123", 1);

// Assert
Assert.That(busId, Is.Not.EqualTo(Guid.Empty));
```

In the frontend: `beforeEach` (Arrange: `TestBed.configureTestingModule` + HTTP fakes) → call/expectation with `httpMock.expectOne` (Act) → `expect(...)` (Assert).

### 4.5 Suite quality rules

- **Independence**: each test creates its own fakes and fixture; it does not share state between tests.
- **Repeatability**: identical results in any order; the clock and GUIDs are controlled or not assumed.
- **Speed**: execution in milliseconds per test (no real I/O).
- **No order**: there is no dependency between tests; parallelization is defined explicitly with `[Parallelizable]` when applicable.
- **One assertion per test**: the name states the condition and the expected result.

---

## 5. Components or units to test

Table conventions: the *ID* column groups by module; the **[no tests]** mark indicates units that **have no existing test** today and require creation; the rest already have tests that must be kept and extended.

### 5.1 Backend — ms-configuration (RF-02: security policies and settings)

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| CON-01 | `UpdateSettingService` | `ExecuteAsync` | Updates a security setting (login attempts, lockout period) with range validation | High |
| CON-02 | `UpdatePasswordPolicyService` | `ExecuteAsync` | Updates the password policy (minimum/maximum length, requirements) validating consistency | High |
| CON-03 | `GetSettingService` | `Execute` | Looks up a setting by key/classification | Medium |
| CON-04 | `ListSettingsService` / `ListPasswordPoliciesService` | `Execute` | Lists settings and policies | Medium |

### 5.2 Backend — ms-fleet (RF-03.6, RF-03.7)

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| FLE-01 | `CreateBusService` | `ExecuteAsync` | Creates a bus; rejects a duplicate plate; generates an identifier | High |
| FLE-02 | `UpdateBusService` / `ChangeBusStatusService` / `DeleteBusService` | `ExecuteAsync` | Editing, activation/deactivation, and deletion of buses | High |
| FLE-03 | `AssignDriverToBusService` / `UnassignDriverFromBusService` | `ExecuteAsync` | Assigning/unassigning a driver to a bus; prevents double assignment | High |
| FLE-04 | `GetBusAssignedToDriverService` | `ExecuteAsync` | Obtaining the bus assigned to a driver | Medium |
| FLE-05 | `GetBusDetailService` / `ListBusesBasicService` | `ExecuteAsync` | Bus detail and basic listing | Medium |
| FLE-06 | `SearchBusesService` + `NameSearchStrategy` / `PlateSearchStrategy` | `ExecuteAsync` | Search by name or plate | Medium |
| FLE-07 | `ListBrandsService` / `ListModelsService` | `ExecuteAsync` | Brand/model catalogs | Low |
| FLE-08 | Bus view projections (`BusViewFieldsTests`) | — | Verification of the fields exposed in the DTO/view | Medium |

> Note: `CreateBusServiceTests.cs` presents a style defect (duplicated `using Xunit;` directive on lines 1 and 2); see section 12.

### 5.3 Backend — ms-route (RF-03.1 to RF-03.5, RF-03.8, RF-07.1, RF-09)

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| ROU-01 | `CreateRouteService` | `ExecuteAsync` | Route creation with data, ordered stops, and schedules; route uniqueness | High |
| ROU-02 | `UpdateRouteService` / `DeleteRouteService` | `ExecuteAsync` | Update and deletion of routes | High |
| ROU-03 | `AssignBusToRouteService` | `ExecuteAsync` | Assigning a bus to a route | High |
| ROU-04 | `AssignStudentToRouteService` | `ExecuteAsync` | Assigning a student with a boarding and alighting stop | High |
| ROU-05 | `StartTripService` **[no tests]** / `EndTripService` **[no tests]** | `ExecuteAsync` | Trip lifecycle: start and end (RF-07.1), active trip control | High |
| ROU-06 | `GetCurrentTripService` / `GetCurrentRouteService` | `ExecuteAsync` | Lookup of the current trip/route (depends on the clock) | High |
| ROU-07 | `GetStudentStopOnRouteService` / `GetStudentRouteService` | `ExecuteAsync` | Student's boarding/alighting point and route | Medium |
| ROU-08 | `GetDriverRouteTodayService` | `ExecuteAsync` | Route of the day for the driver (RF-09.2) | Medium |
| ROU-09 | `SearchRouteService` + `NameSearchStrategy` / `ScheduleSearchStrategy` (Route) | `ExecuteAsync` | Route search by name and schedule | Medium |
| ROU-10 | `SearchStopService` + `NameSearchStrategy` (Stop) **[no tests]** | `ExecuteAsync` | Stop search | Medium |
| ROU-11 | `CreateStopService` **[no tests]** / `UpdateStopService` **[no tests]** / `DeleteStopService` **[no tests]** / `AttachStopToRouteService` **[no tests]** | `ExecuteAsync` | Stop CRUD and attachment to a route | Medium |
| ROU-12 | `ListRouteService` / `ListRouteServiceTenant` / `ListStopService` / `ListCityService` | `ExecuteAsync` | Listings with tenant/city filtering | Medium |

### 5.4 Backend — ms-exceptional

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| EXC-01 | `CreateDriverUsageService` | `ExecuteAsync` | Registration of a driver's exceptional use with its validations | High |
| EXC-02 | `CreateRouteUsageService` | `ExecuteAsync` | Registration of a route's exceptional use with its validations | High |
| EXC-03 | `ListDriverUsagesService` / `ListRouteUsagesService` | `ExecuteAsync` | Exceptional use queries | Medium |

### 5.5 Backend — ms-school-management

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| SCM-01 | `CreateSchoolWithCampusesService` | `ExecuteAsync` | Creates a school with its campuses (`SchoolCampusRequestRules`, `SchoolLocationRequestRules` rules) | High |
| SCM-02 | `UpdateSchoolWithCampusesService` / `UpdateSchoolService` | `ExecuteAsync` | Update of school and campuses | High |
| SCM-03 | `CreateSchoolService` / `DeleteSchoolService` | `ExecuteAsync` | Creation and removal of a school | High |
| SCM-04 | `LinkAdminSchoolService` / `GetAdminSchoolService` | `ExecuteAsync` | Administrator ↔ school linking and its lookup | High |
| SCM-05 | `SearchSchoolsService` + `NameSearchStrategy` | `ExecuteAsync` | School search by name | Medium |
| SCM-06 | `ListSchoolsService` / `GetCampusService` / `ListCampusesBySchoolService` / `ListSchoolCampusesService` | `ExecuteAsync` | Listings and campus lookups | Medium |

### 5.6 Backend — ms-user-management (RF-01)

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| USM-01 | `CreateStudentService` / `CreateDriverService` / `CreateParentService` / `CreateAdminService` | `ExecuteAsync` | Creation of accounts by role with unique identification validation | High |
| USM-02 | `RegisterFamilyService` | `ExecuteAsync` | Family registration and linking of students (RF-01.4) | High |
| USM-03 | `CreateDriverLicenseService` / `UpdateDriverLicenseService` / `DeleteDriverLicenseService` | `ExecuteAsync` | Driver's license management with a unique number (`DriverLicenseRules`) | High |
| USM-04 | `UpdateStudentService` / `UpdateDriverService` / `UpdateParentService` / `UpdateAdminService` / `UpdateFamilyService` | `ExecuteAsync` | Update of account data (RF-01.2) | High |
| USM-05 | `DeleteStudentService` / `DeleteDriverService` / `DeleteParentService` / `DeleteAdminService` / `DeleteFamilyService` | `ExecuteAsync` | Activation/deactivation or deletion (RF-01.3) | Medium |
| USM-06 | `SearchPersonService` + `NameSearchStrategy` / `EmailSearchStrategy` / `IdentificationSearchStrategy` / `PhoneSearchStrategy` | `ExecuteAsync` | Person search by the four criteria | Medium |
| USM-07 | `SearchFamilyService` / `GetFamilyService` / `GetFamilyMembersByStudentService` / `ListFamilyService` | `ExecuteAsync` | Family: lookups and members per student | Medium |
| USM-08 | `GetStudentService` / `GetDriverService` / `GetParentService` / `GetAdminService` / `GetDriverLicenseService` / `List*Service` | `ExecuteAsync` | Individual lookups and listings | Medium |
| USM-09 | `TenantPersonFilter` / `RoleFilteredList` | — | Isolation by tenant and by role (data from other tenants is not leaked) | High |
| USM-10 | `PersonProfile` (AutoMapper) / `DriverLicenseView` | — | Person and license projections into DTO/view | Medium |

### 5.7 Backend — sg-ms-forgot-information (RF-02: credential recovery, contact changes)

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| FGT-01 | `VerificationCodeService` | `GenerateAsync` / `VerifyAsync` | Generation, validation, and expiration of verification codes | High |
| FGT-02 | `ForgotPasswordService` / `VerifyPasswordResetCodeService` / `ResetPasswordService` | `ExecuteAsync` | Password reset flow (does not reveal account existence) | High |
| FGT-03 | `RequestPhoneChangeService` / `RequestPhoneVerificationService` / `VerifyPhoneChangeIdentityService` / `CheckPhoneVerificationService` / `ResendPhoneVerificationService` | `ExecuteAsync` | Phone number change with SMS verification | High |
| FGT-04 | `RequestEmailChangeService` / `VerifyEmailChangeCodeService` / `SubmitNewEmailService` / `ConfirmEmailChangeService` / `ResendNewEmailService` | `ExecuteAsync` | Email change with two-step verification | High |
| FGT-05 | `OtpCodeGenerator` + `OtpOptions` | `Generate` | Generation of a 6-digit OTP (range 000000–999999) | High |
| FGT-06 | `SecretHasher` / `EmailLogMask` / `InputValidators` | various | Secret hashing, email masking in logs, input validators | Medium |
| FGT-07 | `TransactionalEmails` (templates) / `SmtpEmailSender` / `IdentityDirectoryHttpClient` | various | Email templates and transport; HTTP client to the identity directory | Medium |

### 5.8 Backend — ms-notification (RF-04, RF-05, RF-08) — **no test project**

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| NOT-01 | `ProcessScanService` | `ExecuteAsync` | Processes the QR scan and decides boarding/alighting (ON_BOARD/OFF_BOARD) and recipients | High |
| NOT-02 | `RegisterDeviceService` | `ExecuteAsync` | Registration/deregistration of the guardian's device token | High |
| NOT-03 | `GetGuardianNotificationsService` | `ExecuteAsync` | Lookup of a guardian's alerts and notifications | Medium |
| NOT-04 | `StudentScannedConsumer` / `StudentScannedEventMapper` / `KafkaEventPublisher` | various | Consumption of the scan event and publication on the bus; event mapping | Medium |

### 5.9 Backend — ms-iam (RF-02, RF-04.1, RF-08)

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| IAM-01 | `LoginService` **[no tests]** | `login` | Authentication with credentials, password validation, and token issuance (RF-02.1) | High |
| IAM-02 | `RefreshTokenService` **[no tests]** / `LogoutService` **[no tests]** | `refresh` / `logout` | Token renewal and revocation with a deny list (RF-02.2) | High |
| IAM-03 | `CreateProfileService` **[no tests]** | `execute` | Profile creation with a role and a domain event | High |
| IAM-04 | `InternalProfileService` | `lookup` | Internal profile lookup by normalized email (used by other services) | High |
| IAM-05 | `GetProfileService` / `UpdateAdminSchoolService` **[no tests]** | `execute` | Profile lookup and administrator school update | Medium |
| IAM-06 | `GetTermsStatusService` **[no tests]** / `RecordTermsAcceptanceService` **[no tests]** / `ListTermsAcceptancesService` **[no tests]** | `execute` | Terms and conditions acceptance and status | Medium |
| IAM-07 | `JwtTokenProvider` **[no tests]** | `generate` / `validate` | JWT token generation, signing, validation, and expiration | High |
| IAM-08 | `InternalApiKeyFilter` | `doFilterInternal` | API key validation for internal endpoints | High |
| IAM-09 | `JwtAuthenticationFilter` **[no tests]** | `doFilterInternal` | JWT authentication on frontend requests | High |
| IAM-10 | `GlobalExceptionHandler` / `AuthExceptionHandler` | — | Translation of exceptions into structured responses | Medium |
| IAM-11 | `PersonCreatedConsumer` **[no tests]** / `KafkaDomainEventPublisher` **[no tests]** / `KafkaConsumerConfig` / `KafkaProducerConfig` | — | Consumption/publication of domain events | Medium |

### 5.10 Frontend — web application (RF-01 to RF-10 in the interface)

| ID | Module/Class | Method/Function | Description | Priority |
|---|---|---|---|---|
| FE-01 | `AuthService` | `login` / `logout` | JWT session management, persistence of `profileId`/`personId`/email | High |
| FE-02 | `StudentsService` / `BusesService` / `RoutesService` / `StopsService` | HTTP CRUD | Consumption of the administration endpoints with network mocks | High |
| FE-03 | `ForgotInformationService` | recovery flows | Credential/contact change requests (email, phone, password) | High |
| FE-04 | `ThemeContextService` | — | Persistence and application of the theme (RF-10.1) | Medium |
| FE-05 | `problem-detail` (core/http) | parse | Transformation of the backend's ProblemDetails error responses into manageable errors | High |
| FE-06 | Admin pages: `Buses`, `Routes`, `Stops`, `Drivers`, `Families`, `Guardians`, `Students` | — | Screen behavior: loading, filters, forms, messages | High |
| FE-07 | Dashboards: `dashboard-admin`, `dashboard-superadmin` | — | Summaries and navigation by role | Medium |
| FE-08 | Profile: `change-contact` (code/phone/reset steps), `change-email`, `change-password` | — | Guided contact data change flows | High |
| FE-09 | Public feature (login/recovery) and `superadmin` (admins, schools) | — | Public authentication and global administration | Medium |
| FE-10 | Shared components (`shared/components`, 19 specifications) | — | Reusable: tables, dialogs, controls | Medium |
| FE-11 | Guards, interceptors, and validators (`core/guards`, `core/interceptors`, `core/validators`) | — | Route protection, token attachment, form validations | High |

---

## 6. Unit test case design

### 6.1 Techniques applied

The tests in table 6.3 explicitly apply the following white-box techniques:

- **Equivalence partitioning (EC)**: input data is grouped into equivalent classes (valid / invalid) and at least one case per class is designed. Example: identification `null`, `""`, invalid format, and valid format.
- **Boundary value analysis (BVA)**: values at the edges of each range are tested. Examples: minimum/maximum password length, OTP `000000` and `999999`, bus capacity `0`, expiration at the exact instant.
- **Exception and error tests**: it is verified that the SUT throws the correct exception (or returns the structured error) on invalid states (`Assert.That(..., Throws.TypeOf<T>())` in NUnit, `assertThrows` in JUnit, `expect(() => ...).toThrow()` in Vitest).
- **Branch, statement, and condition coverage**: every `if`/`switch` and every compound logical operator (AND/OR) in the SUT must be exercised at least once as true and once as false; it is measured with the coverage tool of each layer (see sections 3 and 9).

### 6.2 Format of the cases

Each case is documented in the table with: **ID**, **name**, **unit tested**, **related requirement** (real RF-xx references from the requirements report), **description and objective**, **preconditions and configuration** (fixtures/mocks), **input data**, **expected result**, **obtained result**, **status**, and **type**.

> Status: at the close of drafting, the tests **have not been executed** (see section 10); the *obtained result* field remains "—" and the status remains "Not executed". Cases marked **[to create]** do not yet have an implemented test in the code.

### 6.3 Unit test cases

| ID | Case (name) | Unit tested | Requirement | Description and objective | Preconditions and configuration | Input data | Expected result | Obtained result | Status | Type |
|---|---|---|---|---|---|---|---|---|---|---|
| PU-001 | Creating a bus with a duplicate plate rejects the operation | `CreateBusService.ExecuteAsync` (ms-fleet) | RF-03.6 | Verify that an already-registered plate prevents creation and throws an exception | `InMemoryBusRepository` preloaded with a bus with plate `ABC123`; `FakeGpsDeviceService` | Same plate `ABC123`, capacity 40, period 1 | `ArgumentException`; a second bus is not created | — | Not executed | Negative / Exception |
| PU-002 | Creating a bus with valid data returns an identifier | `CreateBusService.ExecuteAsync` (ms-fleet) | RF-03.6 | Verify the happy path of bus creation | Empty `InMemoryBusRepository`; `FakeGpsDeviceService` | Plate `ABC123`, capacity 40, license expiry +1 year | Non-empty `Guid`; bus persisted in the fake | — | Not executed | Positive |
| PU-003 | Creating a bus with limit capacity 0 or negative | `CreateBusService.ExecuteAsync` (ms-fleet) | RF-03.6 | **BVA**: validate the lower bound of capacity | Fake repository and GPS | Capacity `0` and `-1` | Argument exception (invalid capacity) | — | Not executed | Boundary value |
| PU-004 | Assigning a student to a nonexistent stop | `AssignStudentToRouteService.ExecuteAsync` (ms-route) | RF-03.8 | Verify that assignment does not occur with stops not valid on the route | Route/stop repository fakes; route with 2 stops | Boarding stop that does not belong to the route | Controlled exception; no assignment created | — | Not executed | Negative / Exception |
| PU-005 | Current trip with no active trip returns empty | `GetCurrentTripService.ExecuteAsync` (ms-route) | RF-07.1 | **BVA/EC**: behavior when there is no trip in progress | `FixedTimeProvider` at a time with no trip; empty trip repository | `/current` query with no trips | Empty/`null` response by contract, without an exception | — | Not executed | Boundary value |
| PU-006 | Starting a second trip with an active trip **[to create]** | `StartTripService.ExecuteAsync` (ms-route) | RF-07.1 | Verify the active-trip exclusivity rule (condition branch) | `FixedTimeProvider`; a trip already started and not finished | `StartTrip` with the same bus/route | Rejection or controlled exception; the first trip intact | — | Not executed | Negative / Exception |
| PU-007 | Ending a trip with no active trip **[to create]** | `EndTripService.ExecuteAsync` (ms-route) | RF-07.1 | **BVA**: closing a nonexistent trip must not corrupt state | Trip repository with no active trips | `EndTrip` on an inactive trip | No-op or controlled error; consistent state | — | Not executed | Boundary value |
| PU-008 | Route schedule rejects Saturday and Sunday | `CreateRouteService` schedule rules (ms-route) | RF-03.1 | **BVA**: the schedule is restricted to Monday–Friday | Route/stop fakes | Days `Saturday` and `Sunday` in the schedule | Rejection of the non-working day | — | Not executed | Boundary value |
| PU-009 | Route search by partial name | `SearchRouteService` + `NameSearchStrategy` (ms-route) | RF-09 | **EC**: partial match, case/accent insensitive | Fake repository with routes "Colegio A", "Colegio B" | `query = "colegio"` | Returns both routes | — | Not executed | Positive |
| PU-010 | Route listing isolated by tenant | `ListRouteServiceTenant` (ms-route) | RF-03.1 | Verify that a tenant does not see another tenant's data | Fakes with routes from two tenants | Query with `tenantId` A | Only routes from tenant A | — | Not executed | Positive (partition) |
| PU-011 | Updating a setting with login attempts out of range | `UpdateSettingService.ExecuteAsync` (ms-configuration) | RF-02 | **BVA**: lower and upper bound of the attempts policy | `InMemoryConfigurationRepositories` | Attempts `0` and `10` (policy maximum) | Exception for an out-of-range value | — | Not executed | Boundary value |
| PU-012 | Policy with minimum length greater than maximum | `UpdatePasswordPolicyService.ExecuteAsync` (ms-configuration) | RF-02 | **BVA/consistency**: logically invalid combination | Fake policy repository | Minimum `12`, maximum `6` | Exception; policy unchanged | — | Not executed | Boundary value |
| PU-013 | Creating a student with duplicate identification | `CreateStudentService.ExecuteAsync` (ms-user-management) | RF-01.1 | Verify identification uniqueness across persons | `InMemoryPersonRepository` with a student with ID `12345` | Same document `12345`, student role | Uniqueness exception | — | Not executed | Negative / Exception |
| PU-014 | Creating a student with null or empty identification | `CreateStudentService.ExecuteAsync` (ms-user-management) | RF-01.1 | **EC/BVA**: invalid input classes (`null`, `""`, spaces) | Empty fake repository | `null`, `""`, `"   "` | Validation fails (input error) for all three classes | — | Not executed | Boundary value / Negative |
| PU-015 | Registering a license with a duplicate number | `CreateDriverLicenseService.ExecuteAsync` (ms-user-management) | RF-03.7 | Verify uniqueness of the license number (`DriverLicenseRules`) | `InMemoryDriverLicenseRepository` with a license | Repeated license number | Exception; license not created | — | Not executed | Negative / Exception |
| PU-016 | Expired verification code is rejected | `VerificationCodeService.VerifyAsync` (sg-ms-forgot-information) | RF-02 | **BVA**: OTP expiration at the time boundary | `ManualTimeProvider`; code generated with a 5-min TTL | Verification 6 min after generation | Rejection due to expiration | — | Not executed | Boundary value |
| PU-017 | Generated OTP within the 6-digit range | `OtpCodeGenerator.Generate` (sg-ms-forgot-information) | RF-02 | **BVA/EC**: bounds `000000` and `999999`; fixed format | `OtpOptions` with length 6 | 1000 random generations | All codes within `[000000, 999999]` with length 6 | — | Not executed | Boundary value / Positive |
| PU-018 | Reset with an unregistered email does not reveal the account | `ForgotPasswordService.ExecuteAsync` (sg-ms-forgot-information) | RF-02 | Security: identical generic response (prevents account enumeration) | Empty `FakeVerificationRequestRepository`; `FakeNotificationSenders` | Nonexistent email | Same response as an existing email; no sending | — | Not executed | Negative |
| PU-019 | New password below the policy minimum | `ResetPasswordService.ExecuteAsync` (sg-ms-forgot-information) | RF-02 | **BVA**: lower bound of password length | Policy minimum `PwdMinLength = 8`; valid code | 7-character password | Rejection by policy | — | Not executed | Boundary value |
| PU-020 | Email masking in logs | `EmailLogMask` (sg-ms-forgot-information) | RF-02 | Verify that the email is partially masked in records | Direct instance of `EmailLogMask` | `juan.perez@example.com` | Mask that hides part of the user and the domain | — | Not executed | Positive |
| PU-021 | Student scan with no assignment to an active route | `ProcessScanService.ExecuteAsync` (ms-notification) **[to create]** | RF-04.2 / RF-08.1 | Verify handling of a scan from a student with no trip in progress or no assignment | Fake dependencies: boarding, alert, recipient repositories; simulated Kafka events | Student QR scan with no assignment | No boarding generated; controlled alert/incident | — | Not executed | Negative / Exception |
| PU-022 | Valid boarding generates an ON_BOARD event and notifies | `ProcessScanService.ExecuteAsync` (ms-notification) **[to create]** | RF-04.2 / RF-05.1 / RF-05.2 | Verify the happy path: boarding record, event, and recipients | Fakes with a student assigned to an active route; fixed clock | Valid boarding QR scan | `Boarding` created; `StudentScanned` event published; guardians notified | — | Not executed | Positive |
| PU-023 | Login with correct credentials issues a token | `LoginService.login` (ms-iam) **[to create]** | RF-02.1 | Verify JWT issuance with role claims after authentication | Mockito mocks: profile repository, `PasswordEncoder`, `TokenProvider` | Valid credentials of an active profile | Response with `accessToken`/`refreshToken` and profile | — | Not executed | Positive |
| PU-024 | Login with an incorrect password is rejected | `LoginService.login` (ms-iam) **[to create]** | RF-02.1 | Verify the invalid-credentials exception branch | Mocks; `PasswordEncoder.matches` returns `false` | Valid email + incorrect password | `ResponseStatusException` 401; no token issued | — | Not executed | Negative / Exception |
| PU-025 | Renewal with a token on the deny list | `RefreshTokenService.refresh` (ms-iam) **[to create]** | RF-02.2 | Verify that revoked tokens are not renewed | `InMemoryTokenDenyList` with the token; repository mocks | Revoked token + `refreshToken` | Refresh rejection | — | Not executed | Negative / Exception |
| PU-026 | Expired token is invalidated | `JwtTokenProvider.validate` (ms-iam) **[to create]** | RF-02.2 | **BVA**: JWT expiration at the time boundary | `JwtTokenProvider` with a controlled clock/`JwtParser` | Token with expiration in the past | Token declared invalid | — | Not executed | Boundary value |
| PU-027 | Invalid internal API key is rejected | `InternalApiKeyFilter.doFilterInternal` (ms-iam) | RF-02 | Verify protection of internal endpoints | Mock of `FilterChain` and `HttpServletRequest/Response` | Incorrect `X-Api-Key` header | 401 response; `FilterChain` not invoked | — | Not executed | Negative |
| PU-028 | Domain exception is translated into a structured response | `GlobalExceptionHandler` (ms-iam) | RF-02 | Verify the `ErrorResponseDto` format for a known exception | Handler instance; domain exception thrown | Request that triggers `ProfileAlreadyExistsException` | Structured response with code and message | — | Not executed | Positive |
| PU-029 | Frontend login saves profileId/personId | `AuthService.login` (frontend) | RF-02.1 | Verify persistence of the session on the client | `HttpTestingController`; test JWT token generated in the spec | `/login` response with `accessToken`, `profileId`, `personId`, email | Session fields persisted; subscription completes | — | Not executed | Positive |
| PU-030 | Backend ProblemDetails error is handled | `problem-detail` / `AuthService` (frontend) | RF-02 | Verify parsing of error responses (negative) | `HttpTestingController` returning 401/400 with a ProblemDetails body | Error response with `status`, `title`, `detail` | Typed error consumable by the UI; no uncontrolled exception | — | Not executed | Negative |
| PU-031 | Non-parseable error payload does not break the flow | `problem-detail.parse` (frontend) | RF-02 | **BVA/EC**: empty or non-JSON body | `HttpTestingController` with 404 responses without a body and 500 HTML | Body `""` or `null` | Default object; does not throw | — | Not executed | Boundary value |
| PU-032 | `BusesService` maps the listing response | `BusesService.list` (frontend) | RF-03.6 | Verify the HTTP contract: search query and mapping to the model | `HttpTestingController`; mocked `Bus[]` response | GET request with `search` | Correct query sent and buses mapped | — | Not executed | Positive |

> **Extension**: cases PU-001 to PU-032 are the initial suite. Every new unit referenced in section 5 must generate its own cases following the template in Annex A (section 16), with consecutive IDs (PU-033, PU-034, …).

---

## 7. Dependency handling (mocks, stubs, fakes)

### 7.1 Simulated dependencies and how they are configured

| Dependency | Real nature | Where it is simulated | What simulates it | How it is configured |
|---|---|---|---|---|
| EF Core database (SQL Server) | `*Repository` repositories | All .NET services | **Fakes** `InMemory*Repository` (e.g., `InMemoryBusRepository`, `InMemoryPersonRepository`, `InMemoryExceptionalRepositories`, `InMemoryConfigurationRepositories`) | Injected into the use case constructor instead of the real repository; they persist in in-memory collections |
| EF context (mapping) | `SettingsContext`, `ExceptionalContext` | `Infrastructure/*ContextTests` tests | Context with `UseSqlServer("Server=unused;...")` | Does not open a connection: only queries model metadata |
| JPA repositories + encoder | `ProfileJpaRepository`, `PasswordEncoder` | `ms-iam` | **Mockito mocks** (`mock(...)` + `when(...).thenReturn(...)`) | Mockito (JUnit 5); responses are programmed and interactions verified |
| System clock | `DateTime.UtcNow` / `Clock` | `ms-route`, `sg-ms-forgot-information` | `FixedTimeProvider`, `ManualTimeProvider` | Controlled time provider injected; allows testing expiration and "today" |
| GPS device | External GPS service | `ms-fleet` | `FakeGpsDeviceService` | Returns a simulated device identifier |
| Identity directory API (HTTP) | `IdentityDirectoryHttpClient` | `sg-ms-forgot-information` | `FakeIdentityDirectoryClient` | Fake implementation of the HTTP client |
| SMTP / email | `SmtpEmailSender`, templates | `sg-ms-forgot-information` | `FakeNotificationSenders` / simulated sender | Replaces the real transport with in-memory sending |
| SMS verification | External SMS provider | `sg-ms-forgot-information` | `FakeSmsVerificationService` | Returns deterministic codes/states |
| Verification request repository | EF repository | `sg-ms-forgot-information` | `FakeVerificationRequestRepository` | In-memory record of requests and codes |
| Frontend HTTP | `HttpClient` to the gateway | Frontend (services) | `HttpTestingController` (`provideHttpClientTesting()`) | `expectOne(...)`, `flush(...)`/`error(...)` per test; `afterEach(() => httpMock.verify())` |
| Kafka messaging | Broker | `ms-notification`, `ms-iam` | In-memory publishers/consumers in the fakes | Fake implementations of the event publisher are injected |

### 7.2 Criteria for choosing a mock, stub, or fake

| Double | When to use it | Example in the project |
|---|---|---|
| **Fake** | When the dependency has **its own logic** worth reusing (stateful repositories, searches) or when a complete functional contract is needed | `InMemory*Repository`, `ManualTimeProvider` |
| **Stub** | When only a **fixed value** needs to be returned so the SUT can proceed; interactions do not matter | `FakeGpsDeviceService`, preprogrammed responses in `when(...).thenReturn(...)` |
| **Mock** | When the **interaction** must be verified (what was called, how many times, with which arguments) | `mock(ProfileJpaRepository.class)` in ms-iam; `HttpTestingController` in the frontend |
| **Dummy** | When the parameter is not used in the scenario (minimum values `Guid.NewGuid()`, tolerated nulls) | Convenience arguments in `ExecuteAsync` calls |

Rule of thumb: **prefer fakes for repositories** (realistic and reusable state), **stubs for stateless external services**, and **mocks only when interaction verification adds regression value**. Avoid unnecessary mocks to reduce the coupling of tests to implementation details.

---

## 8. Test data

### 8.1 Data classification by domain (EC and BVA)

| Domain | Valid data | Invalid data | Boundary values to test |
|---|---|---|---|
| Person identification | Non-empty, unique numeric document | `null`, `""`, spaces, duplicate of another record | Minimum/maximum document length; `0000000000` |
| Bus plate | Plate existing in the catalog (e.g., `ABC123`) | Duplicate plate, blank | Unique plate first time vs. second time |
| Bus capacity | 1–99 passengers | 0, negative, > max | `0`, `1`, `99`, `100` |
| Route schedule | Monday to Friday | Saturday, Sunday | `Monday` (lower bound) and `Friday` (upper bound) |
| License age/expiry | Future date | Past date, `null` | Today (valid) vs. yesterday (expired) |
| OTP code | 6 digits in `[000000, 999999]` | 5 or 7 digits, letters, empty | `000000`, `999999` |
| OTP/token expiration | Within the TTL | After the TTL | Instant TTL − 1 min, exact TTL, TTL + 1 min |
| Password | Length between the policy minimum and maximum, meets requirements | Short, missing requirements | Length `PwdMinLength − 1`, `PwdMinLength`, `PwdMaxLength`, `PwdMaxLength + 1` |
| Searches (name/email/ID/phone) | Present substring, uppercase, accents | Absent, empty, `null` string | 1-character string, full string, no matches |
| Frontend query | Valid parameters (`search`, `page`, `pageSize`) | Null parameters | No results → empty list; 404/500 error without a body |

### 8.2 Fixtures, builders, and factories

- The project's fakes already act as data *factories*: `InMemory*Repository` can be preloaded with sample entities when the instance is created.
- In `ms-route` and `sg-ms-forgot-information`, fixed time providers (`FixedTimeProvider`, `ManualTimeProvider`) are used as a *temporal fixture*.
- In the frontend, the specs build **test JWT tokens** through a local helper function (`jwt(claims)` that base64-encodes the header/claims) and typed `flush` responses for each service.
- For new tests, it is recommended to centralize **builders** (e.g., `TestBus()`, `TestStudent()`, `TestOtpRequest()`) in the `Fakes/` folder of each test project, reusing the real domain entities and DTOs.

### 8.3 In-memory data and support files

- **In-memory data**: `List<T>`/`Dictionary` collections inside the fakes (no I/O), ideal for the required isolation.
- **Support files**: they are not required for unit tests; if necessary (very large expressions or templates), they would be placed as *embedded resources* of the test project and read without touching the real file system (avoid `File.ReadAllText` in the SUT).

---

## 9. Running the tests

### 9.1 Local execution (without a container)

**.NET microservices** (per service):

```bash
cd <service>
dotnet restore
dotnet test tests/<service>.Tests/<service>.Tests.csproj \
  --configuration Release \
  --collect:"XPlat Code Coverage" \   # or "--collect:Code Coverage" in projects without coverlet
  --logger "trx;LogFileName=results.trx"
```

**IAM (Java):**

```bash
cd ms-iam
./mvnw test          # JUnit 5 via spring-boot-starter-*-test
```

**Frontend:**

```bash
npm ci
npx ng test --watch=false   # Vitest via @ngx-env/builder:unit-test (jsdom)
```

> Times and reports: `--collect` generates per-project coverage XML files and `--logger trx` produces result reports; in Vitest the default report is shown in the console and can be exported to JUnit/HTML by adding the corresponding reporters.

> **Note (NUnit):** `dotnet test` runs the NUnit tests through `NUnit3TestAdapter`; the test project references the `NUnit` package in addition to `Microsoft.NET.Test.Sdk`.

### 9.2 Containerized execution (dedicated Docker only for tests)

The goal is to **package a copy of the project** into a disposable container that runs only the unit tests, without exposing ports or requiring the real infrastructure (SQL Server, Kafka, SMTP).

**9.2.1 Base image by stack**

| Stack | Base image (SDK, not runtime) | Justification |
|---|---|---|
| .NET 10 | `mcr.microsoft.com/dotnet/sdk:10.0` | Compile and run the .NET (NUnit) tests without an additional runtime |
| Java 21 + Maven | `maven:3.9-eclipse-temurin-21` | Compile and run JUnit/Mockito |
| Angular/NPM | `node:22-alpine` | `npm ci` + Vitest in jsdom |

**9.2.2 Example: Dockerfile for a .NET microservice**

```dockerfile
# Dockerfile.test — only for running unit tests
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS test
WORKDIR /app

# 1. Restore dependencies (cacheable layer)
COPY ms-fleet/src/ms-fleet.Api/ms-fleet.Api.csproj ms-fleet/src/ms-fleet.Api/
COPY ms-fleet/tests/ms-fleet.Tests/ms-fleet.Tests.csproj ms-fleet/tests/ms-fleet.Tests/
RUN dotnet restore ms-fleet/tests/ms-fleet.Tests/ms-fleet.Tests.csproj

# 2. Copy the service and test source code
COPY ms-fleet/ ms-fleet/

# 3. Run the tests with coverage and a report
CMD ["dotnet", "test", "ms-fleet/tests/ms-fleet.Tests/ms-fleet.Tests.csproj", \
     "--configuration", "Release", \
     "--collect:XPlat Code Coverage", \
     "--logger", "trx;LogFileName=results.trx"]
```

**9.2.3 Example: docker-compose for the test suite**

```yaml
# docker-compose.test.yml — orchestration of test containers (without real services)
services:
  test-fleet:
    build: { context: ., dockerfile: Dockerfile.test }
    volumes:
      - ./reports/fleet:/app/TestResults   # export TRX + coverage
    networks: [test]
  test-route:
    build: { context: ms-route, dockerfile: Dockerfile.test }
    volumes:
      - ./reports/route:/app/TestResults
    networks: [test]
  test-iam:
    image: maven:3.9-eclipse-temurin-21
    working_dir: /app
    volumes:
      - ./ms-iam:/app
      - ./reports/iam:/app/target/surefire-reports
    command: ./mvnw test
    networks: [test]
  test-web:
    image: node:22-alpine
    working_dir: /app
    volumes:
      - ./dvlp-web:/app
    command: sh -c "npm ci && npx ng test --watch=false"
    networks: [test]

networks:
  test: {}
```

Orchestration commands:

```bash
# The whole suite (each container starts, runs, and dies; no ports exposed)
docker compose -f docker-compose.test.yml up --build --abort-on-container-exit

# A single service
docker compose -f docker-compose.test.yml run --rm test-iam

# Cleanup afterward
docker compose -f docker-compose.test.yml down
```

**9.2.4 Notes for the test container**

- No SQL Server, Kafka, SMTP, or the gateway are brought up: the tests use exclusively fakes and mocks.
- The mounted volumes only export reports (`trx`, coverage XML, `surefire-reports`, Vitest HTML) to demonstrate the results.
- In CI, one job per service is recommended to isolate failures and parallelize execution.

---

## 10. Results

### 10.1 Execution status

At the close of this version, **the tests have not been executed** in a formal verification environment; therefore, all result fields are marked as **"Not executed"**. The counts of **existing** tests come from a static inspection of the code (inventory), not a run.

| Module | Existing tests (static inventory) | Executed | Passed | Failed | Skipped | Time |
|---|---|---|---|---|---|---|
| ms-configuration | 16 (NUnit) | Not executed | — | — | — | — |
| ms-exceptional | 17 (NUnit) | Not executed | — | — | — | — |
| ms-fleet | 25 (NUnit) | Not executed | — | — | — | — |
| ms-route | 45 (NUnit) | Not executed | — | — | — | — |
| ms-school-management | 15 (NUnit) | Not executed | — | — | — | — |
| ms-user-management | 41 (NUnit) | Not executed | — | — | — | — |
| sg-ms-forgot-information | 47 (NUnit) | Not executed | — | — | — | — |
| ms-iam | 18 (JUnit @Test) | Not executed | — | — | — | — |
| **Backend (total)** | **224 test methods** | Not executed | — | — | — | — |
| Frontend (dvlp-web) | 64 `*.spec.ts` files | Not executed | — | — | — | — |
| ms-notification | 0 (no test project) | — | — | — | — | — |

### 10.2 How to generate the reports automatically

Running section 9 automatically generates:

| Report | Command | Artifact |
|---|---|---|
| .NET results | `dotnet test --logger "trx;LogFileName=results.trx"` | `.trx` (Visual Studio Test Results) |
| .NET coverage | `dotnet test --collect:XPlat Code Coverage` (or `--collect:Code Coverage`) | `coverage.cobertura.xml` / `*.xml` |
| Java results | `./mvnw test` | `target/surefire-reports/*.txt` and `*.xml` |
| Java coverage (recommended) | `./mvnw test` after adding JaCoCo (see section 12) | `target/site/jacoco/*` (HTML/XML) |
| Frontend | `npx ng test --watch=false` | Console + JUnit/HTML reports if Vitest reporters are added |

The evidence (reports and screenshots) will be attached by the responsible party in section 16 as agreed in the project.

---

## 12. Defects found

Findings from the static review of the code (test status and configuration), prior to running the suite:

| ID | Defect | Affected module | Probable cause | Applied / recommended fix | Status | Ticket |
|---|---|---|---|---|---|---|
| DEF-01 | **No unit test project exists** | `ms-notification` | The service was created without a test suite (the other services have `tests/{service}.Tests`) | Create `ms-notification.Tests` and cover `ProcessScanService`, `RegisterDeviceService`, `GetGuardianNotificationsService`, and `StudentScannedConsumer` (cases NOT-01..NOT-04) | To fix | — (create in Jira/GitHub Issues) |
| DEF-02 | **Insufficient test coverage in ms-iam**; no tests for login, tokens, and JWT filters | `ms-iam` | Only 6 classes tested (18 `@Test`); `LoginService`, `RefreshTokenService`, `LogoutService`, `CreateProfileService`, `JwtTokenProvider`, `JwtAuthenticationFilter`, `GetTermsStatusService`, `RecordTermsAcceptanceService`, `ListTermsAcceptancesService`, `UpdateAdminSchoolService`, `PersonCreatedConsumer` without tests | Implement cases IAM-01 to IAM-11 of this document (PU-023..PU-028) | To fix | — |
| DEF-03 | Duplicated `using Xunit;` directive | `ms-fleet` (`CreateBusServiceTests.cs`, lines 1–2) | Copy/paste when creating the file | Remove the duplication (trivial fix) | To fix | — |
| DEF-04 | **JaCoCo not configured** (no Java coverage metrics) | `ms-iam` (`pom.xml`) | The `pom.xml` declares only `spring-boot-maven-plugin` and `maven-compiler-plugin` | Add `org.jacoco:jacoco-maven-plugin` with `argLine` preparation and verification reporting | To fix | — |
| DEF-05 | **Non-uniform .NET coverage**: `coverlet.collector` missing in 4 projects | ms-fleet, ms-route, ms-user-management, ms-forgot-information | Only 3 of 7 projects declared Coverlet | Add `coverlet.collector` to the 4 projects or standardize the Test SDK's built-in collector across all of them | To fix | — |
| DEF-06 | **No continuous integration** (no `.github/workflows` in backend or frontend) | Infrastructure | Tests are only run locally | Create CI pipelines with one job per service and a coverage threshold; containerized execution per section 9 | To fix | — |
| DEF-07 | **Weak smoke tests on frontend pages** | Frontend (e.g., `buses.spec.ts`) | Several page specifications verify only `should create` | Extend with behavior tests (data loading, filters, error states) aligned with FE-06..FE-09 | To fix | — |

---

## 13. Test maintenance

### 13.1 When to update the tests

- **Requirement change**: every new business rule (or change to an existing one) requires updating the corresponding test **in the same change** (TDD for new functionality).
- **Refactoring**: if the behavior does not change, the suite must remain green; a refactoring that breaks tests indicates coupling to implementation details and must be fixed in the design.
- **Contract/DTO changes**: `PersonProfile`, `BusViewFieldsTests`, mappings, and projections are reviewed when DTOs are modified.
- **Defect fixes**: every fix is accompanied by its regression test (see section 12).

### 13.2 Flaky or obsolete tests

- Every flaky test is isolated and tagged with `[Category("Flaky")]` (NUnit) / `@Tag("flaky")` (JUnit) / a temporary `it.skip` (Vitest), and a defect is created on the board.
- A test left with `skip` for more than two iterations is **deleted or rewritten**; indefinite skips are not accumulated.
- Obsolete tests (SUT removed or contract permanently changed) are deleted in the PR that removes the functionality.

### 13.3 Review in code review (test quality)

In the review of each PR, it is verified that new tests: (1) test a single thing with a descriptive name; (2) do not touch DB/network/files; (3) cover boundary, negative, and exception cases; (4) follow the AAA pattern and the conventions of section 4; (5) are not fragile against execution order; and (6) maintain the minimum target coverage of 80% in business logic.

---

## 15. Conclusions and recommendations

### 15.1 Compliance with criteria

- The test technical framework is **present and defined** for 7 of the 8 .NET services (**NUnit** + in-memory fakes + controlled time providers) and in the frontend (Vitest + HTTP mocks), with 224 test methods in the backend and 64 specifications in the frontend.
- The global criterion of 80% coverage and 100% passing tests is **not met** today: critical gaps persist (`ms-notification` without tests; `ms-iam` with 18 critical units uncovered; `ms-route` trips and stops without tests) and coverage measurement is not configured in 5 projects.
- The formal execution is pending (section 10) and is the next milestone to validate the exit criteria.

### 15.2 Risks and limitations

- **Security regression risk**: authentication, token renewal, and JWT filters in `ms-iam` have no tests today (RF-02). Impact: high.
- **Operational risk**: `ms-notification` is responsible for boarding/alighting control and guardian notifications (RF-04/RF-05/RF-08) and lacks tests. Impact: high.
- **No CI**: the absence of pipelines means changes can break the suite without being detected.
- **Limitation**: the counts in section 10 are a static inventory; the real numbers (passed/failed/time) will be confirmed after the first run.

### 15.3 Suggested improvements

1. Create the `ms-notification` test project and complete cases NOT-01..NOT-04.
2. Complete the `ms-iam` suite (IAM-01..IAM-11) with Mockito, including the token lifecycle tests.
3. Add JaCoCo to `ms-iam` and uniform `coverlet.collector` in all .NET projects (closes DEF-04 and DEF-05).
4. Create CI pipelines with one job per service, an 80% coverage threshold in `Application`/`Domain`, and automatic reports (closes DEF-06).
5. Extend the frontend *smoke* specifications with behavior tests (closes DEF-07).
6. Adopt TDD for all new functionality and use the builders/fixtures from section 8.2.

---

## 16. Annexes

### Annex A — Unit test case template

| Field | Content |
|---|---|
| ID | (consecutive PU-###) |
| Name | `Method_Condition_Result` |
| Unit tested | SUT class and method |
| Related requirement | RF-xx.y |
| Description and objective | What is verified and why |
| Preconditions and configuration | Fixtures/mocks used and their initial state |
| Input data | Concrete values (include boundaries) |
| Expected result | Expected behavior/value/exception |
| Obtained result | — (filled in after execution) |
| Status | Not executed / Passed / Failed |
| Type | Positive / Negative / Boundary value / Exception |

### Annex B — Reports generated by tools

> Responsible: the author of the evidence (as agreed). The artifacts produced by the commands in section 9.2 will be attached:
> TRX and coverage XML from each .NET service, `surefire-reports` and the JaCoCo report from `ms-iam`, and the Vitest report from the frontend, together with the updated summary of the table in section 10.

### Annex C — Example of a well-written test (real project code)

**Backend (.NET)** — `CreateBusServiceTests.cs` (ms-fleet), in NUnit syntax:

```csharp
[Test]
public async Task ExecuteAsync_WithDuplicatePlate_ShouldThrow()
{
    var repo = new InMemoryBusRepository();
    var gpsService = new FakeGpsDeviceService();
    var useCase = new CreateBusService(repo, gpsService);

    await useCase.ExecuteAsync(
        Guid.NewGuid(), DateTime.UtcNow.AddYears(1), Guid.NewGuid(),
        40, "ABC123", 1);

    Assert.That(async () => await useCase.ExecuteAsync(
            Guid.NewGuid(), DateTime.UtcNow.AddYears(1), Guid.NewGuid(),
            40, "ABC123", 1),
        Throws.TypeOf<ArgumentException>());
}
```

**Frontend (Vitest)** — `auth.service.spec.ts` (dvlp-web):

```typescript
it('saves profileId/personId/email from the login response', () => {
  let done = false;
  service.login('admin@school.com', 'secret').subscribe(() => (done = true));

  httpMock
    .expectOne((req) => req.url.endsWith('/login'))
    .flush({
      accessToken: jwt({ roleId: 1 }),
      profileId: '10',
      personId: '20',
      email: 'admin@school.com',
    });

  expect(done).toBe(true);
});
```

### Annex D — Glossary

See the definitions in sections 1.3 and 7.2 (unit, mock, stub, fake, fixture, coverage, SUT, AAA, EC, BVA, TDD).

### Annex E — Change history

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-10-10 | Initial version (strategy, units to test, cases PU-001..PU-032, dependencies, data, Docker execution, and findings DEF-01..DEF-07). |

---

*Unit testing document — School Guardian · SENA, ADSO Technologist, group 3145556 · October 10, 2026.*
