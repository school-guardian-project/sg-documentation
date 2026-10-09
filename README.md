# Guardian Escolar — Official Documentation

Bilingual documentation for the **Guardian Escolar** project, a web and mobile platform for school transportation monitoring and safety. Each document is available in both languages within its respective folder.

## Structure

```text
documentation/
├── 01-requisitos/          # Software Requirements Specification (SRS)
│   ├── es/                 # Español: informe-especificacion-requisitos-software.md
│   └── en/                 # English: software-requirements-specification.md
├── 02-vistas-dinamicas/    # Dynamic Views Analysis (UML diagrams)
│   ├── es/                 # Español: analisis-vistas-dinamicas.md
│   └── en/                 # English: dynamic-views-analysis.md
├── 03-propuesta-tecnica/   # Technical Proposal
│   ├── es/                 # Español: propuesta-tecnica.md
│   └── en/                 # English: technical-proposal.md
├── 04-diseno/              # Software Design Document
│   ├── es/                 # Español: diseno-software.md
│   └── en/                 # English: software-design.md
├── 05-implementacion/      # Implementation Procedure
│   ├── es/                 # Español: procedimiento-implementacion-software.md
│   └── en/                 # English: software-implementation-procedure.md
└── 06-mejoramiento/        # Corrective, Preventive & Improvement Actions (ACPM)
    ├── es/                 # Español: acciones-correctivas-prevenciones-mejoramiento.md
    └── en/                 # English: corrective-preventive-improvement-actions.md
```

## Documents

| # | Area | 🇪🇸 Spanish | 🇬🇧 English |
|---|------|------------|------------|
| 01 | Requirements (SRS) | [Especificación de Requisitos](./01-requisitos/es/informe-especificacion-requisitos-software.md) | [Requirements Specification](./01-requisitos/en/software-requirements-specification.md) |
| 02 | Dynamic Views | [Análisis de Vistas Dinámicas](./02-vistas-dinamicas/es/analisis-vistas-dinamicas.md) | [Dynamic Views Analysis](./02-vistas-dinamicas/en/dynamic-views-analysis.md) |
| 03 | Technical Proposal | [Propuesta Técnica](./03-propuesta-tecnica/es/propuesta-tecnica.md) | [Technical Proposal](./03-propuesta-tecnica/en/technical-proposal.md) |
| 04 | Software Design | [Diseño del Software](./04-diseno/es/diseno-software.md) | [Software Design Document](./04-diseno/en/software-design.md) |
| 05 | Implementation | [Procedimiento de Implementación](./05-implementacion/es/procedimiento-implementacion-software.md) | [Implementation Procedure](./05-implementacion/en/software-implementation-procedure.md) |
| 06 | Improvement (ACPM) | [Acciones Correctivas y Preventivas](./06-mejoramiento/es/acciones-correctivas-prevenciones-mejoramiento.md) | [Corrective & Preventive Actions](./06-mejoramiento/en/corrective-preventive-improvement-actions.md) |

## Suggested Reading Order

The documents are designed to be read sequentially, following the Software Design Documentation (SDD) methodology:

1. **Start with** `03-propuesta-tecnica` — understand what we're building and why
2. **Then** `01-requisitos` — know exactly what the system must do
3. **Then** `02-vistas-dinamicas` — understand how components interact over time
4. **Then** `04-diseno` — see the architecture, data model, and interfaces
5. **Then** `05-implementacion` — follow the deployment procedure
6. **Finally** `06-mejoramiento` — learn how quality issues are managed

Each document's internal table of contents provides cross-links to related sections.

## Conventions

### Bilingual Consistency
- Identifiers (`RF`, `RNF`, `HU`, `SEQ`, `ACT`, `EST`) are preserved identically in both languages.
- Cross-references between documents use matching section numbers.
- Technical terms not translated: API, REST, gRPC, JWT, RBAC, Kafka, Docker, etc.
- Product name "Guardian Escolar" remains consistent; translations use "Guardian Escolar" not literal translation.

### Internal Navigation
- Each document's table of contents uses working anchor links (#section-id).
- Section numbering follows hierarchical format: `1`, `1.1`, `1.1.1`, etc.
- Tables use markdown syntax for compatibility across viewers.

### Source Reference
- Content is based on the Guardian Escolar project backend at `../backend/` (9 microservices).
- Frontend/mobile apps referenced from `../../school-guardian-project/`.
- Architecture decisions aligned with SDD framework documentation at `../docs/`.

## Project Context

**Guardian Escolar** is a microservices-based platform comprising:
- 9 domain microservices (IAM, User Management, School, Fleet, Routes, Notifications, Exceptional, Audit, Settings)
- Kong API Gateway as single entry point
- SQL Server relational database with per-service schemas
- Apache Kafka event bus for domain events
- gRPC channel for real-time GPS telemetry
- Angular 21 web application + React/Expo mobile applications

## License

SENA — Servicio Nacional de Aprendizaje · Ficha 3145556 · Neiva, Huila, Colombia
