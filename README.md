# Guardian Escolar — Documentación

Documentación oficial del proyecto **Guardian Escolar**, organizada por
documento y por idioma.

## Estructura

Cada documento tiene dos versiones:

- `es/`: versión en español.
- `en/`: versión en inglés.

```text
documentation/
├── 01-requisitos/
│   ├── es/
│   └── en/
├── 02-vistas-dinamicas/
│   ├── es/
│   └── en/
├── 03-propuesta-tecnica/
│   ├── es/
│   └── en/
├── 04-diseno/
│   ├── es/
│   └── en/
├── 05-implementacion/
│   ├── es/
│   └── en/
└── 06-mejoramiento/
    ├── es/
    └── en/
```

## Documentos

| Área | Español | English |
| --- | --- | --- |
| Requisitos | [Informe de especificación de requisitos](./01-requisitos/es/informe-especificacion-requisitos-software.md) | [Software Requirements Specification](./01-requisitos/en/software-requirements-specification.md) |
| Vistas dinámicas | [Análisis de vistas dinámicas](./02-vistas-dinamicas/es/analisis-vistas-dinamicas.md) | [Dynamic Views Analysis](./02-vistas-dinamicas/en/dynamic-views-analysis.md) |
| Propuesta técnica | [Propuesta técnica](./03-propuesta-tecnica/es/propuesta-tecnica.md) | [Technical Proposal](./03-propuesta-tecnica/en/technical-proposal.md) |
| Diseño | [Diseño del software](./04-diseno/es/diseno-software.md) | [Software Design](./04-diseno/en/software-design.md) |
| Implementación | [Procedimiento de implementación](./05-implementacion/es/procedimiento-implementacion-software.md) | [Implementation Procedure](./05-implementacion/en/software-implementation-procedure.md) |
| Mejoramiento | [Acciones correctivas, preventivas y de mejoramiento](./06-mejoramiento/es/acciones-correctivas-prevenciones-mejoramiento.md) | [Corrective, Preventive and Improvement Actions](./06-mejoramiento/en/corrective-preventive-improvement-actions.md) |

## Convenciones editoriales

- Los identificadores de requisitos se conservan en ambos idiomas: `RF`, `RNF`,
  `HU`, `EP`, `UC`, `SEQ`, `ACT` y `EST`.
- Los enlaces relativos deben apuntar al documento del mismo idioma.
- Cada documento debe conservar portada, control de versiones, índice,
  contenido y referencias.
- Las cifras, reglas de negocio, criterios de aceptación y nombres de
  componentes deben coincidir entre las versiones `es/` y `en/`.
- La versión en inglés no debe traducir identificadores técnicos ni nombres
  propios del producto.

## Orden de actualización

La documentación se mantiene en este orden:

1. Requisitos.
2. Vistas dinámicas.
3. Propuesta técnica.
4. Diseño.
5. Implementación.
6. Mejoramiento.

Cuando se modifica una versión, se debe revisar su equivalente en el otro
idioma y actualizar los enlaces del índice.
