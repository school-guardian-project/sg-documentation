
**Acciones Correctivas, Preventivas y de Mejoramiento**

Guardian Escolar

Software para la creación de la aplicación "Guardian Escolar"

Johan Smith Santamaria Fernández  
Sharik Dayanna Rojas Ibarra  
Juan Pablo Chala Ramírez

SENA — Servicio Nacional de Aprendizaje  
Ficha de programación 3145556  
Centro de formación de Neiva (Huila)

Instructor: José de Jesús Motta Vargas  
Neiva, 6 de octubre de 2026

# Control de versiones

**Tabla 1**. Control de versiones del documento

| Versión | Fecha      | Autor                   | Descripción del cambio        |
| ------- | ---------- | ----------------------- | ----------------------------- |
| 1.0     | 06/10/2026 | Equipo Guardian Escolar | Versión inicial del documento |

**Tabla de contenido**

Presione F9 (o clic derecho → Actualizar campo) para generar el índice.

# 1 Introducción y objetivo

## 1.1 Propósito

El presente documento establece el **procedimiento de gestión de acciones correctivas, preventivas y de mejoramiento (ACPM)** del proyecto Guardian Escolar, así como el **registro maestro** que las contiene, las **métricas** con que se miden y el **tablero de indicadores** que se somete a la revisión de dirección.

Guardian Escolar es una plataforma web y móvil para el monitoreo y la seguridad del transporte escolar: geolocalización GPS, escaneo QR de subida y bajada de estudiantes, notificaciones y alertas en tiempo real. La naturaleza del problema que resuelve —la trazabilidad de cada trayecto y la comunicación oportuna con los acudientes— exige que cualquier desviación detectada entre la especificación, la implementación y la operación del sistema se atienda de forma **trazable, analizada en su causa raíz, verificada en su eficacia y cerrada con evidencia**.

## 1.2 Objetivos

Los objetivos del documento son:

- **Estandarizar** la forma de detectar, registrar, analizar, atender y cerrar las desviaciones y las oportunidades de mejora identificadas en el proyecto.
- **Distinguir** con criterios explícitos entre acción correctiva, acción preventiva y acción de mejoramiento, evitando que una misma iniciativa se clasifique de manera arbitraria.
- **Formalizar el análisis de causa raíz** mediante la técnica de los cinco porqués y el diagrama de Ishikawa, de modo que la acción aborde la causa y no el síntoma.
- **Consolidar el registro maestro** de ACPM con los campos mínimos de identificación, responsabilidad, plazo, estado y evidencia de eficacia.
- **Definir métricas de gestión** con fórmula y meta propuesta, y su presentación periódica ante la revisión de dirección.
- **Respaldar la trazabilidad** entre requisitos, pruebas y acciones, en coherencia con los 30 requisitos funcionales, los 8 requisitos no funcionales, las 20 historias de usuario de las 7 épicas y los 9 microservicios de dominio que componen la solución.

## 1.3 Alcance

El procedimiento aplica a:

- Las **no conformidades** detectadas en revisión de requisitos, revisión de código, pruebas, verificación funcional, monitoreo, gestión de incidentes y auditoría interna del sistema de gestión.
- Los **riesgos identificados** en el registro de riesgos y en las decisiones de arquitectura que hayan generado compromisos de mitigación.
- Las **oportunidades de mejora** provenientes de la deuda técnica, de la revisión de procesos y de la retroalimentación del equipo.

Quedan fuera del alcance: los cambios de alcance de producto, que se gestionan en el backlog funcional, y las decisiones arquitectónicas de fondo, que se documentan como decisiones técnicas y, cuando corresponde, alimentan este registro como acciones de mejoramiento.

## 1.4 Público y forma de uso

El documento está dirigido al equipo del proyecto, al responsable de calidad y pruebas, a los responsables técnicos de servicios y de plataforma, y a la revisión de dirección. Su uso es operativo: el procedimiento de la sección 4 se aplica en cada apertura de una ACPM, el registro de la sección 5 se mantiene vigente y las métricas de las secciones 6 y 7 se actualizan al cierre de cada ciclo de revisión.

## 1.5 Términos y acrónimos

**Tabla 2**. Términos y acrónimos utilizados en el documento

| Término        | Definición                                                                                     |
| -------------- | ---------------------------------------------------------------------------------------------- |
| ACPM           | Acción correctiva, preventiva o de mejoramiento.                                               |
| No conformidad | Incumplimiento de un requisito, de un criterio de aceptación o de un procedimiento definido.   |
| Causa raíz     | Condición de origen cuya eliminación previene la recurrencia de la no conformidad.             |
| Eficacia       | Resultado de la acción verificado contra el criterio definido previamente a su implementación. |
| Evidencia      | Registro objetivo que demuestra la ejecución de la acción y el resultado de su verificación.   |
| RF             | Requisito funcional, identificado con un código de la serie RF-01 a RF-10.                     |
| RNF            | Requisito no funcional, identificado con un código de la serie NFR-001 a NFR-008.              |
| HU             | Historia de usuario.                                                                           |
| P0–P3          | Niveles de severidad de un incidente operativo.                                                |
| RPO / RTO      | Objetivo de punto de recuperación y objetivo de tiempo de recuperación.                        |
| SLO            | Objetivo de nivel de servicio (disponibilidad 99,9 % en producción).                           |

# 2 Marco conceptual de las acciones ACPM

## 2.1 Acción correctiva

**Definición.** Acción adoptada para **eliminar la causa de una no conformidad detectada** y evitar su recurrencia. Se dispara cuando el incumplimiento ya ocurrió y está comprobado con evidencia.

**Ejemplos del proyecto:**

- El inicio de sesión emite el token de sesión, pero el control de acceso basado en rol no está aplicado en las rutas protegidas; la acción correctiva consiste en implementar la verificación de permisos por rol (requisitos RF-02) y cubrirla con pruebas de integración.
- El alta de un bus falla en tiempo de ejecución y el verbo de actualización parcial es bloqueado en el punto de entrada; la acción corrige el ruteo y la validación del comando de alta (requisito RF-03).
- La notificación de bajada entrega el nombre de la parada y no las coordenadas; la acción enriquece el mensaje con la posición GPS (requisito RF-05).

## 2.2 Acción preventiva

**Definición.** Acción adoptada para **eliminar la causa de una no conformidad potencial** o de otro resultado no deseado, cuando la causa aún no se ha materializado. Se dispara de un riesgo identificado, de una decisión con consecuencias conocidas o de un hallazgo en revisión de proceso.

**Ejemplos del proyecto:**

- Se adoptó una única instancia de base de datos con esquema por servicio; la acción preventiva crea un inicio de sesión por servicio con permisos exclusivos sobre su esquema y documenta la matriz de propiedad, antes de que ocurra un acceso cruzado.
- El punto de entrada único es componente crítico ante el exterior; la acción preventiva incorpora a las pruebas de integración la verificación de la validación de token y del límite de tasa, antes de que una configuración errónea llegue a producción.
- Las migraciones de esquema no producen instrucciones de reversión de forma automática; la acción preventiva hace obligatoria esa instrucción en cada cambio, antes de que una migración defectuosa impida volver a una versión estable.

## 2.3 Acción de mejoramiento

**Definición.** Acción adoptada para **aumentar el grado en que se cumplen los requisitos o los objetivos** del sistema de gestión, sobre un proceso, una herramienta o un producto que ya funciona pero puede rendir mejor. No corrige una no conformidad ni neutraliza un riesgo inminente: eleva la barra.

**Ejemplos del proyecto:**

- Los indicadores de negocio documentados en el panel de observabilidad usan ejemplos ajenos al dominio; la acción de mejoramiento los sustituye por bordadas registradas, notificaciones entregadas, viajes iniciados y alertas atendidas.
- Los flujos prioritarios de pruebas extremo a extremo permanecen como campos de ejemplo; la acción de mejoramiento define los flujos críticos del producto y los automatiza.
- El cliente móvil es único para todos los roles; la acción de mejoramiento lo reorganiza en módulos por función para contener el costo de mantenimiento a medida que crecen los roles.

## 2.4 Distinción entre tipos de acción

**Tabla 3**. Distinción entre acción correctiva, preventiva y de mejoramiento

| Criterio              | Correctiva                                                | Preventiva                                              | Mejoramiento                                                    |
| --------------------- | --------------------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------- |
| Pregunta que responde | ¿Por qué ocurrió y cómo se evita repetir?                 | ¿Por qué podría ocurrir y cómo se evita?                | ¿Cómo se logra un resultado mejor que el actual?                |
| Estado de la causa    | Materializada y comprobada                                | Potencial, no materializada                             | Condicional: proceso o producto funcional con margen de aumento |
| Disparador            | No conformidad con evidencia                              | Riesgo, decisión con consecuencias, hallazgo de proceso | Deuda técnica, retroalimentación, revisión de indicadores       |
| Condicional de cierre | Eficacia demostrada sobre la causa raíz                   | Ausencia del evento temido durante el periodo definido  | Incremento medible frente a la línea base                       |
| Ejemplo típico        | Reglas de ciclo de viaje ausentes en el servicio de rutas | Permisos de base de datos por servicio                  | Definición de flujos críticos de prueba extremo a extremo       |

**Regla de desempate.** Cuando un mismo hallazgo admita más de una clasificación, se registra como **correctiva** si existe incumplimiento comprobado y, en paralelo, se abre la preventiva que evita la recurrencia en otro componente. La mejora jamás sustituye a la corrección: una no conformidad abierta se cierra con acción correctiva.

## 2.5 Fuentes de detección y alcance de aplicación

**Tabla 4**. Fuentes de detección, tipo habitual y responsable de primera detección

| Fuente de detección                                                 | Tipo habitual             | Responsable de primera detección        |
| ------------------------------------------------------------------- | ------------------------- | --------------------------------------- |
| Requisitos y matriz de trazabilidad                                 | Correctiva                | Responsable de calidad y pruebas        |
| Revisiones de código                                                | Correctiva / preventiva   | Revisor designado                       |
| Pruebas unitarias, de integración y de contrato                     | Correctiva                | Responsable de calidad y pruebas        |
| Verificación funcional de los paneles y de las aplicaciones móviles | Correctiva                | Responsable de interfaz web y móvil     |
| Monitoreo, alertas y métricas de observabilidad                     | Correctiva / mejoramiento | Responsable de plataforma y operaciones |
| Gestión de incidentes y retrospectiva posterior                     | Correctiva / preventiva   | Responsable de incidentes               |
| Registro de riesgos y decisiones de arquitectura                    | Preventiva                | Arquitecto de software                  |
| Registro de deuda técnica y retrospectivas de sprint                | Mejoramiento              | Líder de proyecto                       |
| Dependencias y preguntas abiertas del proyecto                      | Preventiva                | Líder de proyecto                       |
| Auditoría interna del sistema de gestión                            | Preventiva / mejoramiento | Responsable de calidad y pruebas        |

# 3 Clasificación y criterios de severidad y prioridad

## 3.1 Criterio de clasificación por tipo

La clasificación se realiza en la etapa de registro y se confirma en la etapa de análisis de causa raíz. El criterio es secuencial:

1. ¿Existe una no conformidad comprobada frente a un requisito, un criterio de aceptación o un procedimiento? → **correctiva**.
2. ¿Se identificó una causa potencial o un riesgo no materializado? → **preventiva**.
3. ¿Se trata de elevar el desempeño de un proceso, herramienta o producto conforme? → **mejoramiento**.

## 3.2 Escala de severidad

La **severidad** mide la gravedad del efecto, no la urgencia de la respuesta.

**Tabla 5**. Escala de severidad y efecto esperado de cada nivel

| Severidad   | Criterio de asignación                                                                                                                                | Efecto esperado                                                         | Ejemplo del proyecto                                     |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------- |
| **Crítica** | Compromete seguridad, integridad de datos, protección de datos personales de menores o el cumplimiento de un requisito no funcional de prioridad alta | El sistema no puede operar de forma confiable en el componente afectado | Autorización por rol no aplicada en las rutas protegidas |
| **Alta**    | Afecta un flujo crítico del servicio (bordada, notificación, seguimiento en tiempo real, inicio o cierre de viaje) con alternativa parcial            | Incumplimiento directo de uno de los tres objetivos del producto        | Ausencia de la regla de un solo viaje activo por bus     |
| **Media**   | Afecta funcionalidad secundaria con alternativa de uso disponible                                                                                     | Degradación funcional sin bloqueo del flujo principal                   | Notificación de bajada sin coordenadas de ubicación      |
| **Baja**    | Incidencia menor sin impacto operativo ni funcional medible                                                                                           | Molestia o defecto cosmético                                            | Etiqueta incorrecta en un control de interfaz            |

## 3.3 Escala de prioridad y plazos de atención

La **prioridad** ordena la ejecución y se deriva de la severidad, del alcance de los usuarios afectados y de la existencia de una fecha comprometida externa.

**Tabla 6**. Escala de prioridad y plazos máximos de atención

| Prioridad | Denominación | Plazo máximo de implementación                        | Plazo máximo de verificación           |
| --------- | ------------ | ----------------------------------------------------- | -------------------------------------- |
| **P1**    | Inmediata    | 10 días hábiles desde la aprobación de la acción      | 5 días hábiles tras la implementación  |
| **P2**    | Alta         | 20 días hábiles desde la aprobación de la acción      | 10 días hábiles tras la implementación |
| **P3**    | Normal       | Un ciclo de sprint desde la aprobación de la acción   | 10 días hábiles tras la implementación |
| **P4**    | Diferida     | Dos ciclos de sprint desde la aprobación de la acción | 15 días hábiles tras la implementación |

**Regla de deuda técnica.** Toda acción de mejoramiento derivada de deuda técnica debe resolverse en un tiempo inferior a un ciclo de sprint desde su registro, de acuerdo con el criterio de mantenibilidad adoptado para el producto.

**Correspondencia severidad–prioridad orientativa:** severidad Crítica → P1; severidad Alta → P1 o P2; severidad Media → P2 o P3; severidad Baja → P3 o P4. El responsable de calidad puede elevar la prioridad en cualquier momento; reducirla exige aprobación del líder de proyecto y deja constancia en el registro.

## 3.4 Correspondencia con la clasificación de incidentes

Cuando la fuente de detección es un incidente operativo, se conserva la severidad del incidente y se traduce a prioridad ACPM para la acción que emana de la retrospectiva:

**Tabla 7**. Correspondencia entre la severidad de un incidente y la prioridad ACPM

| Severidad del incidente | Definición operativa                                                    | Prioridad ACPM orientativa |
| ----------------------- | ----------------------------------------------------------------------- | -------------------------- |
| **P0**                  | Sistema inoperante; impacto total a todos los usuarios                  | P1                         |
| **P1**                  | Funcionalidad crítica degradada; impacto a más del 50 % de los usuarios | P1                         |
| **P2**                  | Funcionalidad secundaria degradada; existe alternativa                  | P2                         |
| **P3**                  | Incidencia menor sin impacto operativo                                  | P3 o P4                    |

## 3.5 Evidencia admisible por tipo de acción

La evidencia es el soporte del cierre y se declara al definir la acción, no después. Su admisibilidad se evalúa con la siguiente tabla:

**Tabla 8**. Evidencia admisible e inadmisible por tipo de acción

| Tipo de acción            | Evidencia admisible                                                                                                                                         | Evidencia no admisible                                                 |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Correctiva sobre producto | Ejecución de la prueba que detectó la no conformidad con resultado satisfactorio; prueba de regresión añadida; revisión de código con verificación en verde | Confirmación verbal del implementador; captura sin fecha ni contexto   |
| Correctiva sobre proceso  | Publicación del criterio, registro de su aplicación en al menos un ciclo y constancia de socialización                                                      | Circulación del texto sin evidencia de uso                             |
| Preventiva                | Demostración de que el mecanismo preventivo está activo y de que el evento temido sería detectado o bloqueado                                               | Existencia del documento de la medida sin comprobación de su operación |
| Mejoramiento              | Medición del indicador antes y después, con el mismo método y periodo de observación                                                                        | Opinión de que el resultado «mejoró» sin línea base                    |

## 3.6 Reglas de escalamiento y bloqueo

- Una ACPM de prioridad **P1** vencida escala al líder de proyecto al segundo día hábil de retraso y se informa en la revisión de dirección.
- Una ACPM cuya **verificación de eficacia resulte negativa** regresa a la etapa de análisis de causa raíz conservando su identificador, y su prioridad se eleva un nivel.
- Una ACPM **bloqueada por una dependencia externa** (proveedor de notificaciones, alimentación de dispositivos de posición, datos maestros de instituciones) se mantiene en estado «En espera» con la dependencia registrada, y su plazo se suspende solo con aprobación del líder de proyecto.
- No se cierra ninguna ACPM con causa raíz no confirmada; la ausencia de análisis impide el cierre aunque la acción esté implementada.

# 4 Procedimiento documentado de gestión ACPM

## 4.1 Flujo general del procedimiento

PROCEDIMIENTO ACPM — GUARDIAN ESCOLAR

(1) DETECCIÓN (2) REGISTRO (3) ANÁLISIS DE CAUSA RAÍZ

─────────────── ───────────── ─────────────────────────

Requisitos/pruebas Formulario único Técnica de los 5 porqués

Código/revisión Identificador (C-/P-/M-) Diagrama de Ishikawa (6M)

Monitoreo/alertas ───▶ Severidad y prioridad ──▶ Causa raíz confirmada

Incidentes Responsable Responsable + fecha

Riesgos/decisiones Fecha compromiso │

Deuda técnica Criterio de eficacia │

│ │

│ ▼

│ (4) DEFINICIÓN Y APROBACIÓN DE LA ACCIÓN

│ ─────────────────────────────────────────

│ Tipo: correctiva / preventiva / mejoramiento

│ Alcance, recursos, responsable, plazo

│ Criterio de eficacia medible y método

│ │

│ ▼

│ (5) IMPLEMENTACIÓN

│ ──────────────────

│ Cambio en producto, proceso o herramienta

│ Evidencia de ejecución adjunta

│ │

│ ▼

│ (6) VERIFICACIÓN DE EFICACIA

│ ────────────────────────────

│ Verificación por rol distinto del implementador

│ │ │

│ \[Eficaz\] \[No eficaz\]

│ │ │

│ ▼ │

└─────────────────── (7) CIERRE ◀──┘ └──▶ Regreso a (3)

│ conservando el ID

▼

Evidencia archivada

Lección aprendida registrada

Métricas y tablero actualizados

**Figura 1**. Flujo general del procedimiento ACPM en siete etapas

## 4.2 Etapas, responsables y plazos

**Tabla 9**. Etapas del procedimiento ACPM, responsables y plazos máximos

| Etapa                        | Actividad realizada                                                                | Responsable                                       | Plazo máximo                            |
| ---------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------- | --------------------------------------- |
| 1 Detección                | Identificación del hallazgo con evidencia objetiva                                 | Quien detecta, en el rol que origina la detección | Dentro de la misma jornada              |
| 2 Registro                 | Apertura en el registro maestro con el formulario único y clasificación preliminar | Responsable de calidad y pruebas                  | 2 días hábiles                          |
| 3 Análisis de causa raíz   | Cinco porqués e Ishikawa; confirmación de la causa con evidencia                   | Responsable técnico con el responsable de calidad | 5 días hábiles (3 días en prioridad P1) |
| 4 Definición de la acción  | Tipo, alcance, responsable, fecha compromiso y criterio de eficacia                | Líder de proyecto                                 | 3 días hábiles                          |
| 5 Implementación           | Ejecución del cambio y registro de la evidencia                                    | Responsable asignado en la etapa 4                | Según prioridad (sección 3.3)           |
| 6 Verificación de eficacia | Evaluación contra el criterio definido, por rol distinto del implementador         | Responsable de calidad y pruebas                  | Según prioridad (sección 3.3)           |
| 7 Cierre                   | Aprobación, archivado de evidencia, lección aprendida y actualización de métricas  | Líder de proyecto                                 | 5 días hábiles tras la verificación     |

**Tiempo objetivo de ciclo completo** (de la etapa 1 a la etapa 7): no superior a 45 días naturales para prioridad P1 y P2, y no superior a un ciclo de sprint para acciones de mejoramiento derivadas de deuda técnica.

## 4.3 Etapa 2: registro del hallazgo

El registro se realiza con el formulario único del Anexo A y exige, como mínimo:

- Identificador correlativo según el tipo: C-nn (correctiva), P-nn (preventiva), M-nn (mejoramiento) y EJ-nn para los escenarios ilustrativos de plantilla.
- Fecha de detección, fecha de apertura, fuente de detección y descripción objetiva del hallazgo, sin interpretación de causa en esta etapa.
- Requisito, historia de usuario o componente afectado, cuando el hallazgo sea trazable (por ejemplo, RF-03, RF-06, NFR-001, NFR-006).
- Severidad y prioridad preliminares, con la posibilidad de ajustarlas en la etapa 3.
- Responsable técnico provisional y criterio de eficacia propuesto.

**Regla de calidad del registro:** una descripción es válida si permite reproducir el hallazgo sin necesidad de explicación oral. Se rechaza el registro redactado como opinión («el rendimiento parece bajo») en favor de una descripción verificable («la verificación de la matriz de trazabilidad no halló casos de prueba vinculados a las historias de usuario»).

## 4.4 Etapa 3: análisis de causa raíz

El análisis es obligatorio para toda ACPM antes de definir la acción. Su propósito es desplazar la intervención del síntoma a la condición de origen.

### 4.4.1 Técnica de los cinco porqués

Se formulan sucesivas interrogaciones sobre la respuesta anterior hasta alcanzar una causa sobre la que sea posible actuar con una decisión. El análisis se detiene cuando la respuesta corresponde a un proceso, una configuración, una decisión o una condición de conocimiento que el equipo puede modificar.

**Ejemplo ilustrativo (entrada** EJ-01 **del registro):**

**Tabla 10**. Ejemplo de análisis de los cinco porqués de la entrada EJ-01

| Nivel | Pregunta                                                                                        | Respuesta                                                                                                                     |
| ----- | ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 1.º   | ¿Por qué la notificación de subida no se entrega dentro del tiempo objetivo en la ventana pico? | Porque el mensaje se procesa después de acumularse la demanda de la franja de la mañana.                                      |
| 2.º   | ¿Por qué se acumula la demanda?                                                                 | Porque el consumidor del evento de bordada procesa en cola sin prioridad por ventana horaria.                                 |
| 3.º   | ¿Por qué no prioriza?                                                                           | Porque no existe lógica de priorización en el proceso de entrega.                                                             |
| 4.º   | ¿Por qué no existe?                                                                             | Porque el diseño de entrega no contempló diferenciación por ventana de demanda.                                               |
| 5.º   | ¿Por qué no la contempló?                                                                       | Porque el criterio de escalamiento y de priorización por ventana horaria no se definió al aprobar el objetivo de rendimiento. |

**Causa raíz (ejemplo):** ausencia de un criterio documentado de escalamiento y priorización por ventana horaria. **Acción (ejemplo):** correctiva — ampliar la capacidad de entrega en la ventana pico y medir el indicador extremo a extremo; preventiva — fijar la regla de escalamiento para las futuras ventanas de demanda.

### 4.4.2 Diagrama de Ishikawa (seis categorías)

El diagrama de espinas de pescado organiza las causas potenciales en seis categorías. Se completa en taller breve con el equipo que domina el componente afectado y se contrasta con evidencia.

Mano de obra Método Máquina

│ │ │

▼ ▼ ▼

───────┬─────────────────────┬─────────────────────┬──────────────

│ │ │

└────────▶────────────┴─────────────▶───────┘

│

▼

┌───────────────────────────────┐

│ NO CONFORMIDAD / EFECTO NO │

│ DESEADO DEL COMPONENTE │

└───────────────┬───────────────┘

│

┌────────▶────────────┬──────┴───────▶────────┐

│ │ │

───────┬─────────────────────┬─────────────────────┬──────────────

│ │ │

▼ ▼ ▼

Medida Material Medio ambiente

**Figura 2**. Diagrama de Ishikawa de seis categorías de una no conformidad

**Tabla 11**. Categorías del diagrama de Ishikawa, pregunta guía y ejemplo aplicado

| Categoría      | Pregunta guía                                               | Ejemplo aplicado al proyecto                                                                                    |
| -------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Mano de obra   | ¿El equipo conoce el procedimiento y el dominio?            | Revisiones de código ejecutadas con el mínimo de un revisor, pero sin dominio compartido de las reglas de viaje |
| Método         | ¿El proceso existe y se sigue?                              | Reglas de negocio de ciclo de viaje no trasladadas del diseño a la implementación                               |
| Máquina        | ¿Herramientas, entornos y componentes operan correctamente? | Punto de entrada que no admite todos los verbos del recurso de flota                                            |
| Material       | ¿Datos, configuración y contratos son los correctos?        | Contratos de servicio abiertos sin validación automática en el pipeline                                         |
| Medida         | ¿Se mide con criterio definido y umbral explícito?          | Requisitos no funcionales sin valor, herramienta ni estado de verificación                                      |
| Medio ambiente | ¿Las condiciones de ejecución son representativas?          | Diferencias entre desarrollo y pruebas, y dispositivos móviles reales no disponibles en la validación           |

**Salida obligatoria de la etapa:** causa raíz confirmada, factores contribuyentes (sin asignación de culpa a personas), y la decisión de si el hallazgo genera una, dos o más acciones (por ejemplo, correctiva más preventiva).

### 4.4.3 Criterio de aceptación de la causa raíz

**Tabla 12**. Criterios de aceptación de la causa raíz

| Criterio        | Causa inaceptable                      | Causa aceptable                                                                                                                            |
| --------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Alcance         | «El desarrollador se equivocó»         | «Las reglas de ciclo de viaje no se trasladaron del diseño a la implementación porque no existía un paso de verificación que las exigiera» |
| Accionabilidad  | «La demanda de la mañana fue alta»     | «No existe un criterio de escalamiento por ventana horaria»                                                                                |
| Nivel           | Describe el síntoma con otras palabras | Describe la condición de origen que, al eliminarse, evita la recurrencia                                                                   |
| Verificabilidad | Se apoya en la percepción del equipo   | Se apoya en un registro, una configuración, una prueba o una decisión identificable                                                        |
| Generalización  | Aplica solo al caso concreto observado | Explica por qué el mismo efecto podría repetirse en otro componente equivalente                                                            |

## 4.5 Etapa 4: definición y aprobación de la acción

La acción definida debe cumplir las siguientes reglas:

- **Una acción, un responsable.** El responsable es una persona en un rol, no un equipo.
- **Criterio de eficacia definido antes de implementar**, medible y con método de verificación declarado (reejecución de pruebas, medición de un indicador durante un periodo, revisión de código con verificación en verde, auditoría de registros, simulacro).
- **Fecha compromiso** coherente con la prioridad asignada en la sección 3.3.
- **Alcance explícito**: qué componente, requisito o proceso cubre y qué queda fuera.
- Aprobación del líder de proyecto; para acciones con impacto en seguridad o en protección de datos personales, conformidad adicional del responsable de seguridad.

## 4.6 Etapa 5: implementación

El responsable ejecuta el cambio y adjunta la evidencia de ejecución. En el caso de cambios de producto, la implementación se rige por las reglas de ingeniería adoptadas: ramas de funcionalidad, un cambio de esquema por solicitud de incorporación, verificación continua en verde, un revisor mínimo por solicitud y actualización de la documentación afectada en la misma solicitud. En el caso de cambios de proceso, la implementación incluye la publicación del criterio y la socialización con el equipo.

Durante la implementación no se modifica el criterio de eficacia; si el alcance cambia, la acción se reabre en la etapa 4 con nueva aprobación.

## 4.7 Etapa 6: verificación de eficacia

**Tabla 13**. Reglas de verificación de eficacia de una ACPM

| Aspecto                   | Regla                                                                                                                                                                                                                  |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Quién verifica            | Rol distinto del implementador, designado por el responsable de calidad                                                                                                                                                |
| Contra qué se verifica    | El criterio de eficacia aprobado en la etapa 4, sin modificaciones                                                                                                                                                     |
| Métodos admitidos         | Reejecución de la prueba que detectó la no conformidad; nueva prueba de regresión; revisión de código con verificación en verde; medición del indicador durante el periodo acordado; auditoría de registros; simulacro |
| Resultado                 | «Eficaz» o «No eficaz», con la evidencia correspondiente                                                                                                                                                               |
| Si es no eficaz           | Regreso a la etapa 3, conservando el identificador, elevación de prioridad un nivel y reapertura obligatoria                                                                                                           |
| Si la evidencia no existe | La acción no puede cerrarse                                                                                                                                                                                            |

Para acciones preventivas, la verificación exige además la demostración de que el mecanismo está activo (por ejemplo: el inicio de sesión por servicio existe y su permiso se limita al esquema propio; la instrucción de reversión está presente en el cambio de esquema y se verifica en el pipeline).

## 4.8 Etapa 7: cierre y lecciones aprendidas

Condiciones acumulativas para el cierre:

1. Causa raíz confirmada en la etapa 3.
2. Acción implementada con evidencia de ejecución.
3. Verificación de eficacia positiva, realizada por rol distinto del implementador.
4. Evidencia de eficacia archivada y asociada al identificador.
5. Métricas de la sección 6 y tablero de la sección 7 actualizados.
6. Lección aprendida redactada en una frase con el patrón «hallazgo → causa → regla que se incorpora».

El líder de proyecto aprueba el cierre. Las acciones no eficaces nunca se cierran: se reabren.

## 4.9 Roles y responsabilidades

**Tabla 14**. Roles y responsabilidades en el procedimiento ACPM

| Rol                                     | Responsabilidad en el procedimiento                                                                                 |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Quien detecta                           | Reporta el hallazgo con evidencia dentro de la jornada en que lo identifica                                         |
| Responsable de calidad y pruebas        | Administra el registro, valida la clasificación, verifica la eficacia y calcula las métricas                        |
| Responsable técnico de servicios        | Analiza la causa raíz e implementa las acciones sobre los servicios de dominio                                      |
| Responsable de plataforma y operaciones | Implementa las acciones sobre monitoreo, copias, despliegue y continuidad                                           |
| Responsable de seguridad                | Conforma las acciones relacionadas con autorización, protección de datos y modelos de amenazas                      |
| Responsable de interfaz web y móvil     | Implementa las acciones sobre los clientes de la plataforma                                                         |
| Arquitecto de software                  | Interviene en acciones derivadas de decisiones de arquitectura y en la prevención de acoplamiento entre componentes |
| Líder de proyecto                       | Aprueba prioridad, fecha compromiso y cierre; presenta el tablero en la revisión de dirección                       |

Los roles se designan sobre las personas del equipo al momento de abrir cada registro; el documento no fija asignaciones permanentes.

## 4.10 Trazabilidad y comunicación de las ACPM

Toda ACPM deja rastro en los artefactos de gestión del proyecto. La trazabilidad mínima es la siguiente:

**Tabla 15**. Trazabilidad mínima entre las ACPM y los elementos del proyecto

| Elemento del proyecto              | Vínculo obligatorio con la ACPM                                         | Efecto sobre el elemento                                                                                      |
| ---------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Requisito funcional o no funcional | Identificación del requisito afectado en la descripción                 | Si la acción modifica un criterio, el requisito se actualiza y se vuelve a verificar                          |
| Historia de usuario                | Registro de la acción en la definición de hecho de la historia afectada | La historia no se considera terminada mientras exista una ACPM de severidad Crítica o Alta abierta sobre ella |
| Caso de prueba                     | Adición o modificación del caso que detectó la no conformidad           | La no conformidad se cierra con una prueba que fallaba antes de la acción y pasa después                      |
| Registro de riesgos                | Apertura o actualización del riesgo asociado                            | El riesgo se marca como materializado cuando la fuente fue un incidente, y se revisa su estrategia            |
| Registro de deuda técnica          | Alta del ítem correspondiente cuando la acción es de mejoramiento       | El ítem asume prioridad y fecha compromiso heredadas de la ACPM                                               |
| Decisiones de arquitectura         | Revisión de la decisión cuando la acción afecta sus consecuencias       | La consecuencia negativa prevista se actualiza o se da por tratada                                            |
| Dependencias del proyecto          | Vínculo cuando la acción está condicionada a un tercero                 | La acción pasa a estado «En espera» con la dependencia registrada                                             |
| Lecciones aprendidas               | Frase con el patrón hallazgo → causa → regla incorporada                | La regla alimenta la definición de hecho y las listas de verificación                                         |

**Comunicación.** La apertura de una acción de prioridad P1 se comunica en la jornada siguiente a su registro a los responsables involucrados y al líder de proyecto. El estado completo del registro se comunica en la revisión semanal de calidad. Las acciones con impacto en usuarios (bordadas, notificaciones, seguimiento en tiempo real) se comunican también al responsable de operaciones para su incorporación en el aviso a usuarios cuando corresponda. Las acciones sin responsable designado o sin fecha compromiso se escalan de oficio por el responsable de calidad en el siguiente ciclo de revisión.

# 5 Registro maestro de ACPM

## 5.1 Criterios de lectura del registro

- El registro se presenta en tres bloques por tipo de acción, con las mismas doce columnas: identificador, fecha de apertura, tipo, origen de detección, descripción, causa raíz, acción, responsable, prioridad, estado, fecha de cierre y evidencia de eficacia.
- Los estados admitidos son: Abierta, En análisis, En ejecución, En espera, Verificación y Cerrada.
- Las entradas identificadas con el prefijo EJ-nn son **escenarios ilustrativos** que ejemplifican el uso del formulario y la redacción de una causa raíz completa; **no registran hechos del proyecto** y se incluyen como plantilla de referencia.
- Las entradas restantes derivan de hallazgos verificados en la revisión de requisitos, en la revisión de la estrategia de pruebas, en el marco de observabilidad, en el marco de continuidad operativa y en el control del proyecto; su causa raíz se declara «En análisis» cuando la etapa 3 aún no se ha completado.
- Un guion largo en fecha de cierre o evidencia indica acción aún no verificada; la evidencia programada describe el método que se aplicará, no un resultado obtenido.

## 5.2 Acciones correctivas

**Tabla 16**. Registro maestro de acciones correctivas

| ID    | Fecha      | Tipo                               | Origen de detección                                    | Descripción                                                                                                                                       | Causa raíz                                                                                                                                        | Acción                                                                                                                                                            | Responsable                             | Prioridad | Estado       | Fecha cierre | Evidencia de eficacia                                                                                               |
| ----- | ---------- | ---------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- | --------- | ------------ | ------------ | ------------------------------------------------------------------------------------------------------------------- |
| C-01  | 25/09/2026 | Correctiva                         | Revisión de la trazabilidad entre requisitos y pruebas | El inicio de sesión emite el token de sesión, pero el control de acceso por rol no está aplicado en las rutas protegidas                          | La autorización por rol y permiso no quedó implementada en la capa de filtros del servicio de identificación y acceso                             | Implementar la verificación de permisos por rol y añadir pruebas de integración que cubran el acceso denegado y el permitido                                      | Responsable técnico de servicios        | P1        | En ejecución | —            | Programada: ejecución de las pruebas de autorización y revisión de código con un revisor mínimo                     |
| C-02  | 28/09/2026 | Correctiva                         | Verificación funcional del panel de administración     | El alta de un bus falla en tiempo de ejecución y el verbo de actualización parcial es bloqueado en el punto de entrada                            | El ruteo del punto de entrada no admite todos los verbos del recurso de flota y la validación del comando de alta no tolera los campos opcionales | Corregir la configuración de ruteo del recurso de flota y la validación del comando de alta; verificar con pruebas de la interfaz de programación de aplicaciones | Responsable técnico de servicios        | P1        | En ejecución | —            | Programada: prueba de alta y de actualización parcial de un bus, con respuesta esperada en cada verbo               |
| C-03  | 28/09/2026 | Correctiva                         | Verificación funcional de asignaciones                 | Al asignar un conductor a un bus, la consulta de perfil se realiza contra el servicio equivocado                                                  | Referencia incorrecta al servicio responsable del perfil en la integración entre gestión de flota y gestión de usuarios                           | Corregir la invocación al servicio correcto y validar el contrato de la consulta de perfil                                                                        | Responsable técnico de servicios        | P1        | En ejecución | —            | Programada: prueba de integración de la asignación conductor–bus con verificación de datos del perfil               |
| C-04  | 30/09/2026 | Correctiva                         | Revisión del contenido de las notificaciones           | La notificación de bajada entrega el nombre de la parada y no las coordenadas de ubicación                                                        | El mensaje se construye únicamente con el catálogo de paradas, sin enriquecerlo con la posición GPS registrada en el trayecto                     | Enriquecer el evento y el mensaje con latitud, longitud y hora de la bajada                                                                                       | Responsable técnico de servicios        | P2        | En ejecución | —            | Programada: comparación del contenido del mensaje recibido frente al criterio del requisito RF-05                   |
| C-05  | 30/09/2026 | Correctiva                         | Revisión de reglas de negocio del viaje                | No se aplican la regla de un solo viaje activo por bus ni la validación de bajadas pendientes al cerrar el viaje                                  | Las reglas de ciclo de viaje quedaron solo en la capa de interfaz, sin la guarda correspondiente en el servicio de rutas                          | Implementar ambas validaciones en el servicio de rutas con pruebas unitarias y de integración                                                                     | Responsable técnico de servicios        | P1        | En ejecución | —            | Programada: prueba de viaje concurrente por bus y de cierre con bajadas pendientes                                  |
| C-06  | 01/10/2026 | Correctiva                         | Revisión de la trazabilidad entre requisitos y pruebas | No existe operación de cambio de estado para inactivar cuentas, pese a que la inactivación debe bloquear el acceso y las asignaciones             | La desactivación se implementó únicamente en la validación de acceso, sin la operación administrativa de estado correspondiente                   | Implementar la operación de cambio de estado con registro en la auditoría de acciones críticas                                                                    | Responsable técnico de servicios        | P2        | Abierta      | —            | Programada: prueba de inactivación con verificación de bloqueo de acceso y de asignaciones activas                  |
| C-07  | 02/10/2026 | Correctiva                         | Auditoría interna del sistema de gestión               | La matriz de validación de los requisitos no funcionales no declara valor, herramienta ni estado de verificación para ninguno de los 8 requisitos | Los criterios de aceptación de calidad no se tradujeron en verificaciones automatizables con valor, herramienta, entorno y responsable            | Definir valor, herramienta, entorno, responsable y estado para cada requisito no funcional, desde el rendimiento hasta la recuperación ante desastres             | Responsable de calidad y pruebas        | P1        | En análisis  | —            | Programada: aprobación de la matriz completada en la revisión de dirección                                          |
| EJ-01 | 07/09/2026 | Correctiva (escenario ilustrativo) | Escenario de plantilla: monitoreo en ventana pico      | La notificación de subida excede el tiempo objetivo de entrega en la ventana de la mañana                                                         | Ausencia de un criterio documentado de escalamiento y priorización por ventana horaria (ver ejemplo de cinco porqués en 4.4.1)                    | Ampliar la capacidad de entrega en la ventana pico y fijar la regla de escalamiento                                                                               | Responsable de plataforma y operaciones | P1        | Cerrada      | 21/09/2026   | Ejemplo de plantilla: el indicador extremo a extremo se mantuvo dentro del objetivo durante cinco días consecutivos |

## 5.3 Acciones preventivas

**Tabla 17**. Registro maestro de acciones preventivas

| ID    | Fecha      | Tipo                               | Origen de detección                                                          | Descripción                                                                                                                                           | Causa raíz                                                                                                                          | Acción                                                                                                                                                    | Responsable                             | Prioridad | Estado       | Fecha cierre | Evidencia de eficacia                                                                                                        |
| ----- | ---------- | ---------------------------------- | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- | --------- | ------------ | ------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| P-01  | 24/09/2026 | Preventiva                         | Revisión de la decisión de base de datos compartida con esquema por servicio | Riesgo de acceso no restringido entre esquemas de distintos servicios al compartir una sola instancia de datos                                        | Anticipación del riesgo: la separación lógica por esquema no se respalda hoy con permisos exclusivos por servicio                   | Crear un inicio de sesión por servicio con permisos únicamente sobre su esquema y documentar la matriz de propiedad; verificarlo en la revisión de código | Responsable de plataforma y operaciones | P2        | En ejecución | —            | Programada: consulta de permisos efectivos por inicio de sesión y verificación de que ningún servicio accede a esquema ajeno |
| P-02  | 26/09/2026 | Preventiva                         | Revisión de la decisión de punto de entrada único                            | El punto de entrada único es componente crítico: una mala configuración de sus complementos podría dejar sin validación de token o sin límite de tasa | Anticipación del riesgo: la validación de seguridad del borde no está cubierta por una comprobación automatizada específica         | Incorporar a las pruebas de integración la verificación de la validación del token y del límite de tasa, y revisar la configuración en cada cambio        | Responsable de seguridad                | P1        | En ejecución | —            | Programada: ejecución de las pruebas de borde con token ausente, token inválido y superación del límite de tasa              |
| P-03  | 29/09/2026 | Preventiva                         | Revisión de la decisión de migraciones de esquema                            | Las migraciones de esquema no generan instrucciones de reversión de forma automática; su ausencia impediría volver a una versión estable              | Anticipación del riesgo: la reversión depende de la disciplina manual, sin verificación que la exija                                | Hacer obligatoria la instrucción de reversión en cada cambio de esquema y verificarla en la revisión de código y en el pipeline                           | Responsable técnico de servicios        | P2        | En ejecución | —            | Programada: verificación de que todo cambio de esquema incorpora reversión y que el pipeline lo rechaza cuando falta         |
| P-04  | 01/10/2026 | Preventiva                         | Revisión del marco de continuidad operativa                                  | No hay responsable designado para el calendario de simulacros de restauración, pese a existir verificación periódica de las copias                    | Anticipación del riesgo: sin responsable ni calendario asignados, la verificación de restauración puede postergarse indefinidamente | Designar responsable, fijar calendario trimestral y registrar el resultado de cada simulacro con su prueba de integridad                                  | Responsable de plataforma y operaciones | P2        | En ejecución | —            | Programada: acta del primer simulacro con tiempo de restauración medido y verificación por esquema                           |
| EJ-02 | 09/09/2026 | Preventiva (escenario ilustrativo) | Escenario de plantilla: revisión de seguridad                                | La llave privada de firma de los tokens no tiene calendario de rotación ni procedimiento de reemplazo                                                 | Anticipación del riesgo: la custodia se resolvió por variable de entorno sin definir el ciclo de vida de la llave                   | Definir calendario de rotación, procedimiento de reemplazo y prueba de continuidad de validación tras la rotación                                         | Responsable de seguridad                | P1        | Cerrada      | 23/09/2026   | Ejemplo de plantilla: rotación ejecutada en entorno de pruebas con validación de token vigente antes y después               |
| EJ-03 | 10/09/2026 | Preventiva (escenario ilustrativo) | Escenario de plantilla: revisión de proceso                                  | Las pruebas manuales de un flujo crítico se ejecutan sin lista de verificación acordada                                                               | Anticipación del riesgo: la verificación manual depende de la memoria de quien ejecuta                                              | Publicar lista de verificación del flujo crítico y exigirla como evidencia en cada ciclo de prueba manual                                                 | Responsable de calidad y pruebas        | P3        | Cerrada      | 24/09/2026   | Ejemplo de plantilla: dos ciclos consecutivos con la lista firmada y hallazgos registrados en el mismo formato               |

## 5.4 Acciones de mejoramiento

**Tabla 18**. Registro maestro de acciones de mejoramiento

| ID    | Fecha      | Tipo                                 | Origen de detección                                           | Descripción                                                                                                                                            | Causa raíz                                                                                                                            | Acción                                                                                                                                                | Responsable                             | Prioridad | Estado       | Fecha cierre | Evidencia de eficacia                                                                                             |
| ----- | ---------- | ------------------------------------ | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- | --------- | ------------ | ------------ | ----------------------------------------------------------------------------------------------------------------- |
| M-01  | 23/09/2026 | Mejoramiento                         | Revisión del panel de observabilidad                          | Los indicadores de negocio documentados usan ejemplos ajenos al dominio del producto, tales como creación de órdenes y valor de orden                  | La plantilla de observabilidad se adoptó sin traducir los indicadores al dominio del transporte escolar                               | Definir los indicadores de negocio del producto: bordadas registradas, notificaciones entregadas, viajes iniciados y finalizados, y alertas atendidas | Responsable de plataforma y operaciones | P2        | En ejecución | —            | Programada: tablero con los cuatro indicadores del dominio publicados y con datos de un ciclo completo            |
| M-02  | 23/09/2026 | Mejoramiento                         | Revisión de la estrategia de pruebas                          | Los flujos prioritarios de pruebas extremo a extremo permanecen como campos de ejemplo sin definir                                                     | No se priorizaron los flujos críticos del producto antes de automatizar la capa superior de la pirámide de pruebas                    | Definir al menos tres flujos críticos: inicio de sesión, escaneo de bordada con notificación al acudiente y seguimiento de ruta en tiempo real        | Responsable de calidad y pruebas        | P2        | En ejecución | —            | Programada: ejecución de los tres flujos automatizados en el entorno de pruebas antes de cada publicación         |
| M-03  | 24/09/2026 | Mejoramiento                         | Registro de deuda técnica de prioridad alta                   | La validación de contratos abiertos no se ejecuta de forma automática en el pipeline de verificación continua                                          | La comprobación quedó esbozada en la definición del pipeline sin llegar a implementarse                                               | Habilitar la comprobación de contratos en el pipeline y bloquear la publicación ante divergencia entre contrato e implementación                      | Responsable técnico de servicios        | P1        | En ejecución | —            | Programada: ejecución del pipeline con una divergencia introducida a propósito y bloqueo comprobado               |
| M-04  | 26/09/2026 | Mejoramiento                         | Registro de deuda técnica y de riesgos técnicos               | El análisis de amenazas de la telemetría de posición capturada sin conexión no está elaborado                                                          | El diseño del flujo sin conexión se postergó durante la construcción y no se retomó                                                   | Elaborar el modelo de amenazas del flujo sin conexión: suplantación del dispositivo, reenvío de posiciones y pérdida de eventos                       | Responsable de seguridad                | P3        | Abierta      | —            | Programada: modelo de amenazas aprobado y medidas derivadas incorporadas al backlog técnico                       |
| M-05  | 30/09/2026 | Mejoramiento                         | Revisión de la decisión de cliente único para todos los roles | Se adoptó una aplicación móvil única para todos los roles; el riesgo registrado es el aumento del costo de mantenimiento a medida que crecen los roles | La segmentación por funciones no se estableció desde el inicio del cliente                                                            | Reorganizar el cliente móvil en módulos por función y mantener las validaciones de negocio únicamente en el servidor                                  | Responsable de interfaz web y móvil     | P3        | Abierta      | —            | Programada: verificación de que un cambio en un rol no modifica los módulos de los demás roles                    |
| M-06  | 03/10/2026 | Mejoramiento                         | Revisión de la estrategia de pruebas                          | Los ejemplos de configuración de los entornos de prueba citan un motor de datos distinto del elegido para la plataforma                                | La estrategia se redactó a partir de plantillas genéricas sin ajustarlas al motor de datos en uso ni a los lenguajes de cada servicio | Ajustar los ejemplos de configuración al motor de datos de la plataforma y a la herramienta de prueba de cada lenguaje                                | Responsable de calidad y pruebas        | P3        | En ejecución | —            | Programada: revisión aprobada con ejemplos coherentes con el motor de datos y con la herramienta de cada servicio |
| EJ-04 | 11/09/2026 | Mejoramiento (escenario ilustrativo) | Escenario de plantilla: revisión de dirección                 | El reporte de métricas de calidad para la revisión de dirección se prepara de forma manual y consume tiempo del equipo                                 | Ausencia de una fuente de datos única para el registro de acciones y sus métricas                                                     | Generar de forma automática el reporte de métricas a partir del registro maestro, con corte por ciclo de revisión                                     | Responsable de calidad y pruebas        | P3        | Cerrada      | 25/09/2026   | Ejemplo de plantilla: dos revisiones consecutivas atendidas con el reporte generado sin preparación manual        |

## 5.5 Síntesis del registro

**Tabla 19**. Síntesis del registro maestro de ACPM por tipo de acción

| Tipo de acción | Registradas | Abiertas o en ejecución | Cerradas | Observación                                                             |
| -------------- | ----------- | ----------------------- | -------- | ----------------------------------------------------------------------- |
| Correctivas    | 8           | 7                       | 1        | El único cierre corresponde a un escenario ilustrativo                  |
| Preventivas    | 6           | 4                       | 2        | Los cierres corresponden a escenarios ilustrativos                      |
| Mejoramiento   | 7           | 6                       | 1        | El cierre corresponde a un escenario ilustrativo                        |
| **Total**      | **21**      | **17**                  | **4**    | 17 acciones derivadas de brechas verificadas; 4 escenarios de plantilla |

**Lectura de la síntesis:** las 17 acciones derivadas de brechas verificadas se encuentran abiertas o en ejecución a la fecha de referencia, con causa raíz confirmada en los casos en que la etapa 3 se completó y declarada «En análisis» en el resto. Ninguna acción real se declara cerrada porque ninguna tiene aún evidencia de eficacia registrada.

# 6 Métricas de gestión ACPM

## 6.1 Definición y fórmula

**Tabla 20**. Métricas de gestión ACPM, fórmula e interpretación

| N.º | Métrica                                   | Fórmula                                                                               | Interpretación                                                                   |
| --- | ----------------------------------------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 1   | Grado de cumplimiento en plazo de cierre  | (ACPM cerradas dentro del plazo de su prioridad ÷ ACPM cerradas en el periodo) × 100  | Capacidad del sistema para responder dentro de los plazos comprometidos          |
| 2   | Tiempo medio de ciclo                     | Σ (fecha de cierre − fecha de apertura) ÷ número de ACPM cerradas, en días naturales  | Velocidad media de respuesta del procedimiento                                   |
| 3   | Tasa de recurrencia                       | (ACPM reabiertas o de causa raíz ya cerrada en otro componente ÷ ACPM cerradas) × 100 | Calidad del análisis de causa raíz: una tasa alta indica que se ataca el síntoma |
| 4   | Grado de verificación de eficacia         | (ACPM cerradas con evidencia de eficacia registrada ÷ ACPM cerradas) × 100            | Integridad del cierre: el valor mínimo admisible es total                        |
| 5   | Distribución por tipo                     | (ACPM de cada tipo ÷ total de ACPM del periodo) × 100                                 | Equilibrio entre corrección, prevención y mejora                                 |
| 6   | Acciones vencidas                         | Número de ACPM con fecha compromiso superada y estado distinto de Cerrada             | Presión acumulada del sistema sobre el equipo                                    |
| 7   | Antigüedad media de las acciones abiertas | Σ (fecha de corte − fecha de apertura) ÷ número de ACPM abiertas, en días naturales   | Riesgo de obsolescencia de la acción y de su causa                               |
| 8   | Cobertura de trazabilidad verificable     | (requisitos funcionales con caso de prueba vinculado ÷ 30) × 100                      | Grado en que la verificación respalda los requisitos del producto                |
| 9   | Carga preventiva                          | (ACPM preventivas registradas en el periodo ÷ total de ACPM del periodo) × 100        | Desplazamiento de la gestión desde la corrección hacia la prevención             |

## 6.2 Metas propuestas y regla de actualización

Los valores que siguen son **criterios de aceptación propuestos** por el equipo; se confirman o ajustan en la primera revisión de dirección y, a partir de ahí, se mantienen como compromiso del ciclo.

**Tabla 21**. Metas propuestas, frecuencia de corte y responsable del cálculo

| Métrica                                   | Meta propuesta                                                      | Frecuencia de corte   | Responsable del cálculo          |
| ----------------------------------------- | ------------------------------------------------------------------- | --------------------- | -------------------------------- |
| 1 Cumplimiento en plazo de cierre       | ≥ 90 %                                                              | Por ciclo de revisión | Responsable de calidad y pruebas |
| 2 Tiempo medio de ciclo                 | ≤ 30 días naturales                                                 | Por ciclo de revisión | Responsable de calidad y pruebas |
| 3 Tasa de recurrencia                   | ≤ 5 %                                                               | Trimestral            | Responsable de calidad y pruebas |
| 4 Verificación de eficacia              | 100 % de los cierres                                                | Por ciclo de revisión | Responsable de calidad y pruebas |
| 5 Distribución por tipo                 | Preventivas ≥ 30 % del total acumulado                              | Trimestral            | Líder de proyecto                |
| 6 Acciones vencidas                     | 0 al cierre de cada ciclo                                           | Por ciclo de revisión | Líder de proyecto                |
| 7 Antigüedad media de abiertas          | ≤ 45 días naturales                                                 | Por ciclo de revisión | Responsable de calidad y pruebas |
| 8 Cobertura de trazabilidad verificable | 100 % de los 30 requisitos funcionales antes de la aceptación final | Mensual               | Responsable de calidad y pruebas |
| 9 Carga preventiva                      | En tendencia creciente respecto del periodo anterior                | Trimestral            | Líder de proyecto                |

**Regla de actualización:** el corte se realiza siempre en la misma fecha de cierre de ciclo; los valores se calculan sobre el registro maestro sin excepciones ni ajustes manuales, y toda métrica fuera de meta genera una entrada en el registro (correctiva si incumple un criterio adoptado, de mejoramiento si se trata de elevar la meta).

**Estado de la métrica 8 al 6 de octubre de 2026:** ningún requisito funcional tiene casos de prueba de aceptación vinculados en la matriz de trazabilidad; la meta se declara como criterio de aceptación propuesto y su seguimiento es prioritario.

## 6.3 Ejemplo de cálculo con el registro vigente

El siguiente corte ilustra la aplicación de las fórmulas con los datos del registro maestro. Es un ejemplo de cálculo, no un resultado histórico consolidado.

**Tabla 22**. Ejemplo de cálculo de las métricas con el registro vigente

| Métrica                                   | Datos del corte                                        | Resultado                | Meta propuesta                     | Valoración                                 |
| ----------------------------------------- | ------------------------------------------------------ | ------------------------ | ---------------------------------- | ------------------------------------------ |
| 4 Verificación de eficacia              | 4 cierres, los 4 con evidencia registrada              | 100 %                    | 100 %                              | Cumple                                     |
| 5 Distribución por tipo                 | Correctivas 8, preventivas 6, mejoramiento 7, total 21 | 38,1 % / 28,6 % / 33,3 % | Preventivas ≥ 30 %                 | No cumple: la carga preventiva debe crecer |
| 6 Acciones vencidas                     | Ninguna acción tiene fecha compromiso superada         | 0                        | 0                                  | Cumple                                     |
| 8 Cobertura de trazabilidad verificable | 0 requisitos con caso de prueba sobre 30               | 0 %                      | 100 % antes de la aceptación final | No cumple: acción de mayor prioridad       |
| 9 Carga preventiva                      | 6 preventivas sobre 21 registros                       | 28,6 %                   | En tendencia creciente             | En observación                             |

Las métricas 1, 2, 3 y 7 requieren al menos dos cierres reales para su cálculo; se reportarán desde el primer corte en el que existan acciones cerradas con evidencia, y su ausencia se declara explícitamente en el tablero para no falsear el estado del sistema de gestión.

# 7 Tablero de indicadores para la revisión de dirección

## 7.1 Composición del tablero

El tablero se presenta íntegro en cada revisión de dirección y combina las métricas ACPM con los indicadores de estado del proyecto que las explican.

**Tabla 23**. Composición del tablero de indicadores para la revisión de dirección

| Indicador                                           | Contenido                                                                                    | Meta propuesta                                                         | Estado al 6 de octubre de 2026                                                                                                                               | Frecuencia | Responsable                             |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------------------------------- |
| ACPM registradas y su distribución                  | Total, y desglose por tipo correctiva, preventiva y de mejoramiento                          | Registro único con el 100 % de las acciones abiertas en este documento | 21 registradas: 8 correctivas, 6 preventivas y 7 de mejoramiento; 17 derivadas de brechas verificadas                                                        | Por ciclo  | Responsable de calidad y pruebas        |
| ACPM abiertas y vencidas                            | Acciones en curso y acciones fuera de plazo                                                  | 0 vencidas                                                             | 17 abiertas; ninguna con cierre declarado sin evidencia                                                                                                      | Por ciclo  | Líder de proyecto                       |
| Verificación de eficacia                            | Cierres con evidencia registrada                                                             | 100 % de los cierres                                                   | Las 4 acciones cerradas corresponden a escenarios de plantilla; ninguna acción real cerrada                                                                  | Por ciclo  | Responsable de calidad y pruebas        |
| Requisitos funcionales implementados                | Avance por estado de la matriz de trazabilidad                                               | 30 de 30 antes de la aceptación final                                  | 11 implementados, 12 en progreso, 7 pendientes, sobre 30 requisitos aplicables                                                                               | Mensual    | Responsable técnico de servicios        |
| Requisitos funcionales con caso de prueba vinculado | Cobertura de la trazabilidad requisito–prueba                                                | 30 de 30                                                               | Ninguno con caso de prueba vinculado                                                                                                                         | Mensual    | Responsable de calidad y pruebas        |
| Requisitos no funcionales con validación declarada  | Valor, herramienta, entorno y responsable por requisito                                      | 8 de 8                                                                 | Ninguno con valor ni herramienta declarados                                                                                                                  | Mensual    | Responsable de calidad y pruebas        |
| Preguntas abiertas del proyecto                     | Decisiones pendientes que pueden bloquear                                                    | 0 preguntas vencidas                                                   | 5 preguntas abiertas, entre ellas la retención de datos de posición, los indicadores de recuperación por esquema y los reportes con datos personales         | Por ciclo  | Líder de proyecto                       |
| Dependencias externas abiertas                      | Proveedores, datos e infraestructura pendientes                                              | 0 dependencias abiertas sin fecha comprometida                         | 4 abiertas: taxonomía de rutas y usos excepcionales, contratos de notificación, carga de datos de instituciones y formato de alimentación de posicionamiento | Por ciclo  | Líder de proyecto                       |
| Deuda técnica sin fecha comprometida                | Ítems del registro técnico sin compromiso de sprint                                          | 0                                                                      | 5 ítems registrados, 4 de prioridad alta y 1 de prioridad media                                                                                              | Por ciclo  | Líder de proyecto                       |
| Disponibilidad y rendimiento                        | Cumplimiento del objetivo de disponibilidad 99,9 % y de latencia percentil 95 menor a 300 ms | 99,9 % y < 300 ms                                                      | En definición: la validación de los requisitos no funcionales aún no está declarada                                                                          | Mensual    | Responsable de plataforma y operaciones |
| Incidentes y su conversión en ACPM                  | Incidentes atendidos con retrospectiva y acciones abiertas                                   | 100 % de los incidentes con retrospectiva en 2 días hábiles            | Sin registros de incidentes en el periodo de referencia                                                                                                      | Por ciclo  | Responsable de plataforma y operaciones |

## 7.2 Rituales de revisión y escalado

**Tabla 24**. Rituales de revisión y escalado, contenido, frecuencia y participantes

| Ritual                             | Contenido                                                                          | Frecuencia | Participantes                                                    |
| ---------------------------------- | ---------------------------------------------------------------------------------- | ---------- | ---------------------------------------------------------------- |
| Revisión del registro ACPM         | Apertura, clasificación, causas raíz en análisis y estado de las acciones en curso | Semanal    | Responsable de calidad, responsables técnicos, líder de proyecto |
| Revisión de dirección              | Tablero completo, métricas contra meta, decisiones sobre prioridad y recursos      | Mensual    | Líder de proyecto, responsables de área, revisión de dirección   |
| Revisión de métricas fuera de meta | Plan de acción específico para cada indicador incumplido                           | Trimestral | Líder de proyecto y responsables con indicador afectado          |
| Reapertura y lecciones aprendidas  | Revisión de acciones no eficaces y de recurrencias                                 | Por evento | Responsable de calidad y equipo involucrado                      |

**Escalado:** cualquier indicador fuera de meta en dos cortes consecutivos se convierte en una entrada de mejoramiento con fecha compromiso, y su persistencia se eleva a la revisión de dirección con propuesta de cambio de prioridad o de recursos.

# 8 Conclusiones

- El procedimiento ACPM ofrece un ciclo completo y auditable: de la detección al cierre con evidencia, con causa raíz obligatoria y verificación realizada por un rol distinto del implementador.
- La clasificación en correctiva, preventiva y de mejoramiento, sustentada en criterios explícitos, evita que la corrección se disfraze de mejora y que una no conformidad se cierre sin causa tratada.
- El registro maestro evidencia que el esfuerzo actual está concentrado en la corrección de brechas verificadas de implementación y de documentación, mientras que la carga preventiva debe crecer a medida que el sistema se estabilice.
- Las métricas y el tablero convierten la calidad en un asunto de dirección y no solo del equipo técnico: la trazabilidad entre requisitos y pruebas, hoy el indicador más débil, es el compromiso de mayor prioridad.
- El valor del sistema de gestión se sostiene en la disciplina de actualización: un registro sin cierres verificados, o una meta sin corte periódico, pierde su capacidad de decisión.

# 9 Anexo A — Plantilla de formulario único

El formulario único es el instrumento de apertura, seguimiento y cierre de toda ACPM. Su uso es obligatorio; los campos vacíos impiden avanzar a la etapa siguiente.

FORMULARIO ÚNICO DE ACCIÓN CORRECTIVA, PREVENTIVA Y DE MEJORAMIENTO

Sistema: Guardian Escolar Versión del formulario: 1.0

────────────────────────────────────────────────────────────────────

1 IDENTIFICACIÓN

ID: ...................... (C-nn | P-nn | M-nn | EJ-nn)

Tipo: .................... (Correctiva | Preventiva | Mejoramiento)

Fecha de apertura: ........

Severidad: ............... (Crítica | Alta | Media | Baja)

Prioridad: ............... (P1 | P2 | P3 | P4)

Estado: .................. (Abierta | En análisis | En ejecución |

En espera | Verificación | Cerrada)

2 DETECCIÓN

Fecha de detección: .......

Fuente de detección: ...... (requisitos | código | pruebas | verificación

funcional | monitoreo | incidente | riesgo |

deuda técnica | auditoría)

Detector y rol: ...........

Descripción objetiva del hallazgo (reproducible sin explicación oral):

....................................................................

Requisito, historia o componente afectado: (RF-nn | NFR-nnn | HU-... )

Evidencia de la detección: ..................

3 ANÁLISIS DE CAUSA RAÍZ

Causas inmediatas: ........

Técnica de los cinco porqués:

1 ¿Por qué? .............

2 ¿Por qué? .............

3 ¿Por qué? .............

4 ¿Por qué? .............

5 ¿Por qué? .............

Diagrama de Ishikawa (6M):

Mano de obra .............

Método ...................

Máquina ..................

Material .................

Medida ...................

Medio ambiente ...........

Causa raíz confirmada: .....

Factores contribuyentes (sin asignación de culpa): ................

4 DEFINICIÓN DE LA ACCIÓN

Acción definida: ..........

Alcance (qué cubre y qué queda fuera): ............................

Responsable (rol y persona): ....................................

Fecha compromiso: ..........

Recursos necesarios: .......

Criterio de eficacia (medible, definido antes de implementar):

....................................................................

Método de verificación: ....

Fecha prevista de verificación:

Aprobación del líder de proyecto: ........ Fecha: ...............

Conformidad de seguridad (si aplica): ..... Fecha: ...............

5 IMPLEMENTACIÓN

Fecha de implementación: ...

Cambios realizados: ........

Evidencia de ejecución: ....

6 VERIFICACIÓN DE EFICACIA

Fecha de verificación: .....

Verificador (rol distinto del implementador): .....................

Resultado: ................. (Eficaz | No eficaz)

Evidencia de eficacia: .....

Si no es eficaz: reapertura conservando el ID, elevación de

prioridad un nivel y retorno a la etapa 3: ........................

7 CIERRE

Fecha de cierre: ...........

Aprobación del cierre: ..... Fecha: .......................

Lección aprendida (hallazgo -> causa -> regla incorporada):

....................................................................

Métricas y tablero actualizados: (Sí | No)

Próxima revisión programada: ......

8 REAPERTURA (solo si corresponde)

Motivo: ....................

Nueva causa raíz: ..........

Nueva fecha compromiso: .....

────────────────────────────────────────────────────────────────────

Nota: el identificador «EJ-nn» corresponde a escenarios ilustrativos de

plantilla y no registra hechos del proyecto.

**Figura 3**. Estructura del formulario único de ACPM