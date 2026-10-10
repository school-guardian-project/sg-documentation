
**Análisis del Software: Vistas Dinámicas**

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

# 1 Introducción

## 1.1 Propósito

El presente documento especifica las **vistas dinámicas** del software Guardian Escolar: aquellas descripciones del sistema que representan el comportamiento a lo largo del tiempo mediante intercambios de mensajes entre participantes, secuencias de acciones con puntos de decisión y ciclos de vida de los elementos persistentes. Guardian Escolar es una plataforma web y móvil para el monitoreo y la seguridad del transporte escolar, con geolocalización GPS, escaneo QR de subida y bajada de estudiantes, notificaciones y alertas en tiempo real.

Las vistas dinámicas complementan las vistas estáticas del sistema (requisitos, modelo de datos y arquitectura) al responder tres preguntas de análisis: qué mensajes intercambian los participantes y en qué orden; qué reglas y decisiones gobiernan un proceso de negocio; y en qué estados puede encontrarse cada entidad y qué transiciones lo provocan.

## 1.2 Alcance

El alcance de la modelización dinámica comprende:

- Los **casos de uso generales** de la plataforma con sus actores, requisitos funcionales e historias de usuario asociadas.
- Los **diagramas de secuencia** de los flujos críticos: autenticación, alta de usuarios con eventos, administración institucional, asignaciones de transporte, planificación de rutas, ejecución de trayectos, control de bordadas mediante QR, telemetría GPS y entrega de alertas.
- Los **diagramas de actividad** de los procesos de negocio con reglas de validación, incluido el flujo de usos excepcionales y el procedimiento de auditoría.
- Las **máquinas de estados** de los objetos con ciclo de vida relevante: sesión de usuario, ejecución de ruta, alerta, trayecto del estudiante, dispositivo GPS y perfil de acceso.

Quedan fuera de alcance las vistas estáticas (estructura de clases, modelo de datos detallado e interfaces gráficas), así como los procedimientos de instalación y operación, que se tratan por separado.

## 1.3 Notación UML empleada

La notación se limita a los tres tipos de diagramas de comportamiento definidos en el lenguaje unificado de modelado (UML 2.5), empleados con la siguiente intención:

**Tabla 2**. Tipos de diagrama y elementos de la notación empleada

| Tipo de diagrama | Elementos empleados                                                                                                                          | Uso en el documento                                                                |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Secuencia        | Participantes, lifelines, flechas de mensaje numeradas y respuestas                                                                          | Flujo de mensajes cliente–gateway–servicios–broker                                 |
| Actividad        | Swimlanes (carriles de responsabilidad), nodos de actividad, nodos de decisión y bifurcación, nodos inicial y final, guardas entre corchetes | Procesos de negocio con reglas de validación                                       |
| Estados          | Estados, transiciones etiquetadas con evento y guarda, estado inicial y final                                                                | Ciclo de vida de sesiones, ejecuciones, alertas, bordadas, dispositivos y perfiles |

Los diagramas se presentan en notación gráfica ASCII. La numeración de los mensajes de cada secuencia corresponde exactamente a las filas de su tabla de mensajes.

## 1.4 Convención de diagramas y mensajes

- **Participantes**: se emplean nombres funcionales de rol o de componente. La puerta de enlace se representa siempre como el punto de entrada único de los clientes.
- **Tipos de mensaje**: REST (síncrono cliente–gateway–servicio), gRPC (comunicación síncrona entre servicios, canal persistente en el puerto 5001), evento (mensajería asíncrona en el broker de eventos, con entrega al menos una vez) y push (notificación al dispositivo móvil).
- **Guardas**: expresiones entre corchetes, por ejemplo \[guarda: perfil ACTIVE\]; si la guarda no se cumple, la transición no se ejecuta y se describe el resultado de error en las notas de excepción.
- **Estados y valores de enumeración**: se escriben en mayúsculas (ACTIVE, INACTIVE, ON_BOARD, OFF_BOARD), coherentes con los valores del modelo de datos.
- **Contrato de respuesta**: JSON con códigos HTTP estándar; los errores emplean una estructura consistente con marcas de tiempo, mensaje y detalle.
- **Identificadores**: RF-nn.mm designa un requisito dentro del grupo RF-nn; HU-XXX-nnn designa una historia de usuario; EP-nnn designa una épica; UC-nn, SEQ-nn, ACT-nn y EST-nn designan los artefactos de este documento.

## 1.5 Participantes recurrentes

**Tabla 3**. Participantes recurrentes y tecnologías asociadas

| Participante                    | Función en los diagramas                                          | Tecnología                                      |
| ------------------------------- | ----------------------------------------------------------------- | ----------------------------------------------- |
| Aplicación web                  | Cliente administrativo: formularios, tablas y tablero             | Angular 21 con Angular Material 3               |
| Aplicación móvil (conductor)    | Ruta del día, escáner QR, reporte de incidentes                   | React + Expo                                    |
| Aplicación móvil (acudiente)    | Seguimiento de ruta, notificaciones, historial                    | React + Expo                                    |
| Aplicación móvil (estudiante)   | Visualización del código QR personal                              | React + Expo                                    |
| Puerta de enlace                | Termina TLS, limita tráfico, valida el token e inyecta identidad  | Kong OSS, puerto 8000                           |
| Servicios de dominio            | Lógica de negocio y esquema propio de datos                       | Java 21 (IAM) y C#/.NET 10 (resto), puerto 8080 |
| Broker de eventos               | Publicación y suscripción de eventos, entrega al menos una vez    | Apache Kafka 4.1.0 (KRaft), puerto 9092         |
| Canal gRPC                      | Consultas síncronas entre servicios y telemetría de baja latencia | gRPC, puerto 5001                               |
| Base de datos relacional        | Persistencia con esquema por servicio                             | SQL Server 2022, puerto 1433                    |
| Servicio de notificaciones push | Entrega de avisos a los dispositivos móvil                        | Notificaciones Expo (DeviceToken)               |
| Dispositivo GPS                 | Emisión periódica de posición a bordo del bus                     | Dispositivo embarcado con IMEI registrado       |

# 2 Diagrama general de casos de uso

## 2.1 Descripción general

Los actores primarios corresponden a los cinco roles del sistema (SUPER_ADMIN, ADMIN, DRIVER, PARENT, STUDENT); los actores secundarios son los componentes técnicos que prestan un servicio al caso de uso (puerta de enlace, servicios de dominio, broker de eventos y proveedor de notificaciones). No existe autorregistro de usuarios: el administrador crea todas las cuentas y el servicio de identificación y acceso se limita a autenticarlas.

La tabla siguiente resume la vista de casos de uso del sistema: cada fila es un caso de uso (UC), cada columna un actor, y la marca ● indica participación primaria o secundaria del actor en ese caso.

## 2.2 Matriz de casos de uso

**Tabla 4**. Matriz de casos de uso y actores participantes

| ID    | Caso de uso                                                         | SUPER_ADMIN | ADMIN | DRIVER | PARENT | STUDENT | RF asociados                       | HU                   |
| ----- | ------------------------------------------------------------------- | ----------- | ----- | ------ | ------ | ------- | ---------------------------------- | -------------------- |
| UC-01 | Autenticarse, renovar token y cerrar sesión                         | ●           | ●     | ●      | ●      | ●       | RF-02.1, RF-02.2                   | HU-IAM-001           |
| UC-02 | Gestionar cuentas de usuarios (crear, editar, activar o desactivar) |             | ●     |        |        |         | RF-01.1, RF-01.2, RF-01.3          | HU-USER-004          |
| UC-03 | Vincular estudiantes a la cuenta de un acudiente                    |             | ●     |        |        |         | RF-01.4                            | HU-USER-003          |
| UC-04 | Administrar institución, sedes y cursos                             | ●           | ●     |        |        |         | —                                  | —                    |
| UC-05 | Registrar y administrar la flota de buses                           |             | ●     |        |        |         | RF-03.6                            | HU-FLEET-001         |
| UC-06 | Asignar y desasignar un conductor a un bus                          |             | ●     |        |        |         | RF-03.7                            | HU-BUS-002           |
| UC-07 | Crear, editar y eliminar rutas con paradas ordenadas                |             | ●     |        |        |         | RF-03.1, RF-03.2, RF-03.3, RF-03.4 | HU-ROUTE-001         |
| UC-08 | Asignar un bus a una ruta                                           |             | ●     |        |        |         | RF-03.5                            | HU-ROUTE-005         |
| UC-09 | Asignar estudiantes a una ruta con su parada de subida y bajada     |             | ●     |        |        |         | RF-03.8                            | HU-ROUTE-008         |
| UC-10 | Programar la ruta (día de la semana, dirección y hora de inicio)    |             | ●     |        |        |         | RF-03.1                            | HU-ROUTE-001         |
| UC-11 | Iniciar y finalizar la ejecución de la ruta                         |             |       | ●      |        |         | RF-07.1                            | HU-ROUTE-003         |
| UC-12 | Registrar la subida del estudiante mediante escaneo QR              |             |       | ●      |        |         | RF-04.2, RF-05.1                   | HU-QR-001            |
| UC-13 | Registrar la bajada del estudiante mediante escaneo QR              |             |       | ●      |        |         | RF-04.3                            | HU-QR-002            |
| UC-14 | Visualizar el código QR personal                                    |             |       |        |        | ●       | RF-04.1                            | HU-QR-004            |
| UC-15 | Recibir notificación de subida y bajada                             |             |       |        | ●      |         | RF-05.2, RF-05.3                   | HU-NOTIF-001         |
| UC-16 | Recibir alerta por detención prolongada del bus                     |             |       |        | ●      |         | RF-05.4                            | HU-NOTIF-005         |
| UC-17 | Reportar un incidente durante el trayecto                           |             |       | ●      |        |         | RF-08.1                            | HU-INCIDENT-001      |
| UC-18 | Consultar la ruta asignada con paradas y horarios (conductor)       |             |       | ●      |        |         | RF-09.2                            | HU-ROUTE-002         |
| UC-19 | Consultar la ruta del estudiante (acudiente)                        |             |       |        | ●      |         | RF-09.1                            | HU-ROUTE-004         |
| UC-20 | Seguir la ubicación del bus en el mapa                              |             | ●     |        | ●      |         | RF-06.1, RF-06.2, RF-06.3          | HU-GPS-002           |
| UC-21 | Registrar y validar un uso excepcional de ruta o bus                |             | ●     | ●      |        |         | —                                  | —                    |
| UC-22 | Auditar acciones críticas y consultar el registro                   | ●           | ●     |        |        |         | —                                  | —                    |
| UC-23 | Personalizar el tema de colores y el idioma de la interfaz          | ●           | ●     | ●      | ●      | ●       | RF-10.1, RF-10.2                   | HU-UI-001, HU-UI-002 |

Los casos sin requisito formal asociado (UC-04, UC-21 y UC-22) cubren capacidades de soporte requeridas por el dominio: los datos maestros institucionales que las rutas y la flota necesitan como referencia, el registro de usos fuera del patrón operativo autorizado y la trazabilidad de acciones críticas exigida por el control de la plataforma.

## 2.3 Relaciones entre casos de uso

**Tabla 5**. Relaciones de inclusión y extensión entre casos de uso

| Relación  | Caso base                           | Caso asociado                         | Naturaleza                                               |
| --------- | ----------------------------------- | ------------------------------------- | -------------------------------------------------------- |
| Inclusión | UC-11 Iniciar y finalizar ejecución | UC-01 Autenticarse                    | Toda operación exige un token vigente                    |
| Inclusión | UC-12 y UC-13 Escaneo QR            | UC-11 Iniciar ejecución               | El escaneo requiere una ejecución de ruta activa         |
| Extensión | UC-20 Seguimiento en mapa           | UC-16 Alerta por detención prolongada | La alerta se dispara cuando se excede el umbral          |
| Extensión | UC-11 Ejecución de ruta             | UC-17 Reporte de incidente            | El incidente solo puede ocurrir durante el trayecto      |
| Inclusión | UC-08 Asignar bus a ruta            | UC-06 Asignar conductor a bus         | La cobertura de la ruta deriva del conductor del bus     |
| Extensión | UC-02 Gestión de cuentas            | UC-22 Auditoría                       | La desactivación de cuentas genera registro de auditoría |

# 3 Diagramas de secuencia

Los diagramas de esta sección representan los flujos de mensajes entre clientes, puerta de enlace, servicios de dominio y broker de eventos. Cada diagrama se acompaña de su tabla de mensajes paso a paso y de las notas de excepción que describen los resultados alternativos y de error.

## 3.1 SEQ-01 — Inicio de sesión y renovación del token

Aplicación web Puerta de enlace Servicio IAM Esquema Iam

| | | |

|-- 1. POST /api/v1/auth/login ------------->| |

| |-- 2. reenvío y límite de tráfico ------>|

| | |-- 3. verifica hash|

| | | y estado \[ |

| | | guarda: ACTIVE\]|

| |<-- 4. 200 {accessToken, refreshToken} --|

|<-- 5. par de tokens (access 15 min / refresh 30 días) --------|

| | | |

|-- 6. POST /api/v1/auth/refresh ----------->| |

| |-- 7. reenvío ------------------------->|

| | |-- 8. rota refresh |

| | | \[guarda: vigente\]

| |<-- 9. 200 nuevo par de tokens ---------|

|<-- 10. tokens renovados --------------------------------------|

| | | |

|-- 11. POST /api/v1/auth/logout ----------->| |

| | |-- 12. revoca jti |

| | | y sesión |

|<-- 13. 204 sin contenido -------------------------------------|

**Figura 1**. Diagrama de secuencia de inicio de sesión y renovación del token

**Tabla 6**. Mensajes del diagrama de secuencia de inicio de sesión y renovación del token

| Nº  | Emisor                              | Receptor                            | Tipo    | Descripción                                                                                  |
| --- | ----------------------------------- | ----------------------------------- | ------- | -------------------------------------------------------------------------------------------- |
| 1   | Aplicación web                      | Puerta de enlace                    | REST    | Envío de credenciales (correo electrónico y clave) al recurso de autenticación.              |
| 2   | Puerta de enlace                    | Servicio de identificación y acceso | REST    | Aplicación del límite de intentos por dirección IP y reenvío del recurso público.            |
| 3   | Servicio de identificación y acceso | Esquema Iam                         | Interno | Verificación del hash de la clave de acceso y del estado del perfil.                         |
| 4   | Servicio de identificación y acceso | Puerta de enlace                    | REST    | Devolución del par de tokens firmado con RS256; solo el servicio IAM posee la llave privada. |
| 5   | Puerta de enlace                    | Aplicación web                      | REST    | Entrega del access token (15 minutos) y del refresh token (30 días).                         |
| 6   | Aplicación web                      | Puerta de enlace                    | REST    | Solicitud de renovación con el token de refresco.                                            |
| 7   | Puerta de enlace                    | Servicio de identificación y acceso | REST    | Reenvío de la solicitud de renovación.                                                       |
| 8   | Servicio de identificación y acceso | Esquema Iam                         | Interno | Rotación del refresh token de uso único y actualización de SessionProfile.                   |
| 9   | Servicio de identificación y acceso | Puerta de enlace                    | REST    | Devolución del nuevo par de tokens.                                                          |
| 10  | Puerta de enlace                    | Aplicación web                      | REST    | Entrega de los tokens renovados al cliente.                                                  |
| 11  | Aplicación web                      | Puerta de enlace                    | REST    | Cierre de sesión del usuario.                                                                |
| 12  | Servicio de identificación y acceso | Esquema Iam                         | Interno | Incorporación del identificador del access token a la lista negra y revocación de la sesión. |
| 13  | Servicio de identificación y acceso | Aplicación web                      | REST    | Confirmación del cierre sin cuerpo de respuesta.                                             |

Notas de excepción:

- Credenciales inválidas: respuesta 401 con mensaje genérico que no revela si la cuenta existe.
- Renovación con refresh token ya usado: respuesta 401 y revocación de toda la familia de sesiones por posible robo.
- Perfil en estado INACTIVE: el inicio de sesión se rechaza y las sesiones existentes se revocan.
- Superado el límite de intentos de autenticación desde una misma dirección: la puerta de enlace responde 429 sin invocar al servicio.

## 3.2 SEQ-02 — Alta de usuario con evento \*.created

Aplicación web Puerta de enlace Gestión de usuarios Broker Servicio IAM

| | | | |

|-- 1. POST /api/v1/persons {datos, rol} -------------->| |

| |-- 2. validación de permiso -------->| |

| | |-- 3. gRPC: ¿ | |

| | | identificación | |

| | | duplicada? ----|----------->|

| | |<-- 4. resultado | |

| | |-- 5. INSERT Person |

| |<-- 6. 201 Created -------------------| |

|<-- 7. confirmación del alta ---------------------------| |

| | |-- 8. evento: student.created |

| | |---------------->| |

| | | |-- 9. consume (grupo iam)

| | | |--> 10. crea Profile + hash

| | | |--> 11. evento profile.created

**Figura 2**. Diagrama de secuencia de alta de usuario con eventos

**Tabla 7**. Mensajes del diagrama de secuencia de alta de usuario con eventos

| Nº  | Emisor                              | Receptor                            | Tipo    | Descripción                                                                                                                            |
| --- | ----------------------------------- | ----------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Aplicación web                      | Servicio de gestión de usuarios     | REST    | Alta de una persona con sus datos y el rol asignado (student, driver, parent o admin).                                                 |
| 2   | Puerta de enlace                    | Servicio de gestión de usuarios     | REST    | Validación de permisos e inyección de los encabezados de identidad.                                                                    |
| 3   | Servicio de gestión de usuarios     | Servicio de identificación y acceso | gRPC    | Verificación de que no exista un perfil con el mismo número de identificación.                                                         |
| 4   | Servicio de identificación y acceso | Servicio de gestión de usuarios     | gRPC    | Respuesta sobre la existencia de la identificación.                                                                                    |
| 5   | Servicio de gestión de usuarios     | Esquema UserManagement              | Interno | Persistencia de la persona.                                                                                                            |
| 6   | Servicio de gestión de usuarios     | Puerta de enlace                    | REST    | Devolución del recurso creado.                                                                                                         |
| 7   | Puerta de enlace                    | Aplicación web                      | REST    | Confirmación del alta visible en la lista de cuentas.                                                                                  |
| 8   | Servicio de gestión de usuarios     | Broker de eventos                   | evento  | Publicación de student.created, driver.created, admin.created o parent.created según el rol, con la clave de ordenamiento por entidad. |
| 9   | Broker de eventos                   | Servicio de identificación y acceso | evento  | Entrega del mensaje al grupo de consumo de identificación.                                                                             |
| 10  | Servicio de identificación y acceso | Esquema Iam                         | Interno | Creación del perfil con hash de la clave de acceso, rol y estado ACTIVE (consumo idempotente).                                         |
| 11  | Servicio de identificación y acceso | Broker de eventos                   | evento  | Publicación de profile.created como confirmación del alta de credenciales.                                                             |

Notas de excepción:

- Número de identificación duplicado: rechazo con código 422 y sin publicación de eventos.
- Rol no permitido para el solicitante: respuesta 403 por denegación por defecto.
- Fallo del broker: el alta se confirma solo cuando el evento queda comprometido; la publicación se realiza de forma atómica respecto a la transacción de persistencia y se reintenta en caso de indisponibilidad.
- Desactivación posterior de la cuenta (RF-01.3): bloquea el acceso y la posibilidad de asignaciones sin eliminar el registro de la persona.

## 3.3 SEQ-03 — Creación de sede y curso

Aplicación web Puerta de enlace Gestión escolar Esquema School Transporte

| | | | |

|-- 1. POST /api/v1/schools --------------------------------->| |

| |-- 2. valida SUPER_ADMIN ----------------->| |

| | |-- 3. INSERT School | |

|<-- 4. 201 institución registrada ----------------------------| |

|-- 5. POST /api/v1/campuses {sede, ciudad} ------------------>| |

| |-- 6. valida ADMIN ------------------------>| |

| | |-- 7. INSERT SchoolCampus

|<-- 8. 201 sede creada ---------------------------------------| |

|-- 9. POST /api/v1/courses {sede, nombre} ------------------->| |

| | |-- 10. INSERT Course | |

|<-- 11. 201 curso creado -------------------------------------| |

| | |-- 12. evento SchoolCreated

| | |------------------------>|----------->

**Figura 3**. Diagrama de secuencia de creación de sede y curso

**Tabla 8**. Mensajes del diagrama de secuencia de creación de sede y curso

| Nº  | Emisor                      | Receptor                    | Tipo    | Descripción                                                                                                                   |
| --- | --------------------------- | --------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 1   | Aplicación web              | Servicio de gestión escolar | REST    | Registro de la institución educativa con sus datos de contacto y tema.                                                        |
| 2   | Puerta de enlace            | Servicio de gestión escolar | REST    | Validación del rol SUPER_ADMIN requerido para el recurso.                                                                     |
| 3   | Servicio de gestión escolar | Esquema School              | Interno | Persistencia de la institución.                                                                                               |
| 4   | Servicio de gestión escolar | Aplicación web              | REST    | Confirmación de la creación.                                                                                                  |
| 5   | Aplicación web              | Servicio de gestión escolar | REST    | Alta de una sede asociada a la institución y a una ciudad.                                                                    |
| 6   | Puerta de enlace            | Servicio de gestión escolar | REST    | Validación del rol ADMIN y del alcance de la institución.                                                                     |
| 7   | Servicio de gestión escolar | Esquema School              | Interno | Persistencia de la sede.                                                                                                      |
| 8   | Servicio de gestión escolar | Aplicación web              | REST    | Confirmación de la sede creada.                                                                                               |
| 9   | Aplicación web              | Servicio de gestión escolar | REST    | Alta de un curso asociado a la sede.                                                                                          |
| 10  | Servicio de gestión escolar | Esquema School              | Interno | Persistencia del curso.                                                                                                       |
| 11  | Servicio de gestión escolar | Aplicación web              | REST    | Confirmación del curso creado.                                                                                                |
| 12  | Servicio de gestión escolar | Contextos de transporte     | evento  | Difusión del evento de dominio SchoolCreated para que los contextos de rutas y flota actualicen su referencia de institución. |

Notas de excepción:

- Ciudad de destino inexistente: rechazo con código 404 en la creación de la sede.
- Sede o curso fuera del alcance de la institución del solicitante: respuesta 403.
- Duplicidad de nombre de curso en la misma sede: rechazo con código 422.
- El evento de alta institucional es de corta difusión y no interrumpe el flujo principal si algún consumidor no está disponible.

## 3.4 SEQ-04 — Asignación conductor–bus–ruta

Aplicación web Puerta de enlace Servicio flota Servicio rutas Esquema Fleet

| | | | |

|-- 1. POST /api/v1/buses/{id}/driver {perfil} ----------->| |

| |-- 2. valida ADMIN y permiso ----------->| |

| | |-- 3. \[guarda: sin | |

| | | asignación vigente|

| | |-- 4. INSERT DriverAssignemts

|<-- 5. 201 conductor responsable del bus ------------------| |

| | | | |

|-- 6. POST /api/v1/routes/{id}/bus {busId} -------------->| |

| |-- 7. valida ADMIN ---------------------->| |

| | |<-- 8. gRPC: vigencia del bus y de su conductor

| | |-- 9. bus y conductor ACTIVE |

| | | \[guarda: ruta sin bus y bus sin ruta\]

| | |-- 10. INSERT RouteBusAssignments |

|<-- 11. 201 bus asignado a la ruta ------------------------| |

**Figura 4**. Diagrama de secuencia de asignación de conductor, bus y ruta

**Tabla 9**. Mensajes del diagrama de secuencia de asignación de conductor, bus y ruta

| Nº  | Emisor            | Receptor          | Tipo    | Descripción                                                                                           |
| --- | ----------------- | ----------------- | ------- | ----------------------------------------------------------------------------------------------------- |
| 1   | Aplicación web    | Servicio de flota | REST    | Asignación de un conductor a un bus con vigencia de la asignación.                                    |
| 2   | Puerta de enlace  | Servicio de flota | REST    | Validación del rol ADMIN y del permiso de administración de flota.                                    |
| 3   | Servicio de flota | Servicio de flota | Interno | Evaluación de la guarda de unicidad: el conductor no puede responder a otro bus de forma concurrente. |
| 4   | Servicio de flota | Esquema Fleet     | Interno | Persistencia de la asignación conductor–bus.                                                          |
| 5   | Servicio de flota | Aplicación web    | REST    | Confirmación del conductor responsable.                                                               |
| 6   | Aplicación web    | Servicio de rutas | REST    | Asignación del bus a la ruta.                                                                         |
| 7   | Puerta de enlace  | Servicio de rutas | REST    | Validación del rol ADMIN y del permiso sobre rutas.                                                   |
| 8   | Servicio de rutas | Servicio de flota | gRPC    | Consulta de vigencia del bus, de su conductor y de su estado operativo.                               |
| 9   | Servicio de flota | Servicio de rutas | gRPC    | Respuesta con la condición del bus y del conductor.                                                   |
| 10  | Servicio de rutas | Esquema Route     | Interno | Persistencia de la asignación ruta–bus (un bus por ruta y una ruta por bus).                          |
| 11  | Servicio de rutas | Aplicación web    | REST    | Confirmación de la asignación; la cobertura de la ruta se deriva del bus y su conductor.              |

Notas de excepción:

- Bus ya asignado a otra ruta: rechazo con mensaje «La ruta ya tiene un bus asignado» y código 422.
- Conductor ya responsable de otro bus: rechazo con mensaje «El conductor ya está asignado a un bus».
- Bus o conductor en estado INACTIVE: rechazo de la asignación; la desactivación impide nuevas asignaciones.
- Fallo del canal gRPC: la asignación no se realiza; el servicio aplica tiempo de espera y cortacircuitos antes de responder con error 503.
- La desasignación de bus a ruta es un requisito del grupo RF-03 y su operación complementaria debe estar disponible en el mismo recurso.

## 3.5 SEQ-05 — Planificación y programación de ruta

Aplicación web Puerta de enlace Servicio rutas Esquema Route Transporte

| | | | |

|-- 1. POST /api/v1/routes {sede, sector, nombre} -------->| |

| |-- 2. valida ADMIN ---------------------->| |

| | |-- 3. \[guarda: nombre único, RF-03.2\]

| | |-- 4. INSERT Route | |

|<-- 5. 201 ruta creada -------------------------------------| |

|-- 6. POST /api/v1/routes/{id}/stops \[orden del itinerario\]

| | |-- 7. INSERT RouteStop (orden) |

|<-- 8. paradas registradas --------------------------------| |

|-- 9. registro de programación: día, dirección y hora ----->| |

| | |-- 10. INSERT RouteSchedule |

|<-- 11. 201 programación registrada ------------------------| |

| | |-- 12. eventos RouteCreated y RouteStopAdded

| | |--------------------->|--------------->|

**Figura 5**. Diagrama de secuencia de planificación y programación de ruta

**Tabla 10**. Mensajes del diagrama de secuencia de planificación y programación de ruta

| Nº  | Emisor            | Receptor                | Tipo    | Descripción                                                                                           |
| --- | ----------------- | ----------------------- | ------- | ----------------------------------------------------------------------------------------------------- |
| 1   | Aplicación web    | Servicio de rutas       | REST    | Creación de la ruta con sus datos de cabecera.                                                        |
| 2   | Puerta de enlace  | Servicio de rutas       | REST    | Validación del rol ADMIN y del permiso de creación de rutas.                                          |
| 3   | Servicio de rutas | Servicio de rutas       | Interno | Evaluación de la guarda de unicidad de nombre antes de persistir.                                     |
| 4   | Servicio de rutas | Esquema Route           | Interno | Persistencia de la ruta.                                                                              |
| 5   | Servicio de rutas | Aplicación web          | REST    | Confirmación de la ruta creada.                                                                       |
| 6   | Aplicación web    | Servicio de rutas       | REST    | Registro de las paradas con su orden de itinerario.                                                   |
| 7   | Servicio de rutas | Esquema Route           | Interno | Persistencia de la asociación ruta–parada con el orden indicado.                                      |
| 8   | Servicio de rutas | Aplicación web          | REST    | Confirmación de las paradas ordenadas.                                                                |
| 9   | Aplicación web    | Servicio de rutas       | REST    | Registro de la programación semanal: día de la semana, dirección y hora de inicio.                    |
| 10  | Servicio de rutas | Esquema Route           | Interno | Persistencia de la programación asociada a la asignación de bus.                                      |
| 11  | Servicio de rutas | Aplicación web          | REST    | Confirmación de la programación registrada.                                                           |
| 12  | Servicio de rutas | Contextos de transporte | evento  | Difusión de RouteCreated y RouteStopAdded para los consumidores de flota, geolocalización y bordadas. |

Notas de excepción:

- Nombre de ruta duplicado: rechazo con mensaje «La ruta ya existe» y código 422 (RF-03.2).
- Parada que no pertenece al ámbito de la institución: rechazo con código 400.
- Programación sin bus asignado: la ruta queda planificada y la programación se completa después de la asignación.
- Eliminación de la ruta con una ejecución activa: bloqueada conforme a RF-03.4.
- Fallo de alguno de los consumidores de eventos: no afecta la creación de la ruta; la recuperación se realiza por reintentos y reproceso.

## 3.6 SEQ-06 — Inicio y finalización de la ejecución de la ruta

Móvil conductor Puerta de enlace Servicio rutas Servicio flota Notificaciones

| | | | |

|-- 1. POST /api/v1/executions {ruta, bus, hora} --------->| |

| |-- 2. valida DRIVER ------------------->| |

| | |-- 3. gRPC: bus y conductor válidos ->|

| | |<-- 4. condición del bus -----------|

| | |-- 5. \[guarda: sin viaje activo para el bus\]

| | |-- 6. INSERT RouteExecution (inicio)

|<-- 7. 201 ejecución iniciada ----------------------------| |

| | | | |

| ... registro de bordadas durante el trayecto ... | |

| | | | |

|-- 8. cierre de la ejecución (fin del trayecto) --------->| |

| |-- 9. valida DRIVER ------------------->| |

| | |-- 10. \[guarda: sin bordadas de |

| | | bajada pendientes, RF-07.2\] |

| | |-- 11. UPDATE fin en RouteExecution |

|<-- 12. 200 trayecto finalizado --------------------------| |

| | |-- 13. evento del ciclo del viaje --->|

**Figura 6**. Diagrama de secuencia de inicio y finalización de la ejecución de la ruta

**Tabla 11**. Mensajes del diagrama de secuencia de inicio y finalización de la ejecución de la ruta

| Nº  | Emisor                       | Receptor                     | Tipo    | Descripción                                                                                  |
| --- | ---------------------------- | ---------------------------- | ------- | -------------------------------------------------------------------------------------------- |
| 1   | Aplicación móvil (conductor) | Servicio de rutas            | REST    | Inicio del trayecto con la ruta, el bus y la hora de partida.                                |
| 2   | Puerta de enlace             | Servicio de rutas            | REST    | Validación del rol DRIVER y de la asignación vigente.                                        |
| 3   | Servicio de rutas            | Servicio de flota            | gRPC    | Verificación de que el bus y el conductor respondan a la ruta.                               |
| 4   | Servicio de flota            | Servicio de rutas            | gRPC    | Devolución de la condición operativa.                                                        |
| 5   | Servicio de rutas            | Servicio de rutas            | Interno | Evaluación de la guarda de unicidad: un solo viaje activo por bus (RF-07.1).                 |
| 6   | Servicio de rutas            | Esquema Route                | Interno | Persistencia de la ejecución con fecha y hora de inicio.                                     |
| 7   | Servicio de rutas            | Aplicación móvil (conductor) | REST    | Confirmación del inicio; la aplicación muestra la ruta en curso.                             |
| 8   | Aplicación móvil (conductor) | Servicio de rutas            | REST    | Solicitud de finalización del trayecto.                                                      |
| 9   | Puerta de enlace             | Servicio de rutas            | REST    | Validación del rol DRIVER.                                                                   |
| 10  | Servicio de rutas            | Servicio de rutas            | Interno | Evaluación de la guarda de bordadas pendientes (RF-07.2).                                    |
| 11  | Servicio de rutas            | Esquema Route                | Interno | Actualización de la fecha y hora de fin de la ejecución.                                     |
| 12  | Servicio de rutas            | Aplicación móvil (conductor) | REST    | Confirmación del cierre del trayecto.                                                        |
| 13  | Servicio de rutas            | Notificación y auditoría     | evento  | Difusión del ciclo de vida del viaje para que los consumidores cierren sus flujos asociados. |

Notas de excepción:

- Solicitud de inicio con otro viaje activo para el bus: rechazo con mensaje «Ya existe un viaje activo para este bus» y código 422.
- Finalización con bordadas de bajada pendientes: rechazo con código 422 e indicación de los estudiantes sin baja registrada.
- Conductor sin bus asignado o con asignación vencida: respuesta 403.
- Pérdida de conectividad durante el trayecto: la aplicación conserva la operación pendiente y reintenta al recuperar el enlace.

## 3.7 SEQ-07 — Escaneo QR de subida y bajada (student.scanned)

Móvil conductor Puerta de enlace Notificaciones Servicio rutas Broker Móvil acudiente

| | | | | |

|-- 1. POST /api/v1/boarding {qr, tipo, ruta, parada} ------>| |

| |-- 2. valida DRIVER y JWT ----------------->| |

| | |-- 3. gRPC: estudiante | |

| | | asignado a la ruta y | |

| | | ejecución activa ---->| |

| | |<-- 4. resultado ---------| |

| | |-- 5. \[guarda: subida: asignado; |

| | | bajada: bordada ON_BOARD activa\] |

| | |-- 6. evento student.scanned |

| | |--------------------------->| |

| | | | |-- 7. consume

| | | | |-- 8. INSERT Boarding

| | | | |-- 9. crea aviso al acudiente

| | | | |-------------->|

|<-- 10. 201 bordada registrada -----| | | |

**Figura 7**. Diagrama de secuencia de escaneo QR de subida y bajada

**Tabla 12**. Mensajes del diagrama de secuencia de escaneo QR de subida y bajada

| Nº  | Emisor                       | Receptor                     | Tipo    | Descripción                                                                                        |
| --- | ---------------------------- | ---------------------------- | ------- | -------------------------------------------------------------------------------------------------- |
| 1   | Aplicación móvil (conductor) | Servicio de notificaciones   | REST    | Envío del código escaneado con el tipo de movimiento (ON_BOARD u OFF_BOARD), la ruta y la parada.  |
| 2   | Puerta de enlace             | Servicio de notificaciones   | REST    | Validación del token del conductor.                                                                |
| 3   | Servicio de notificaciones   | Servicio de rutas            | gRPC    | Verificación de que el estudiante esté asignado a la ruta y de que exista una ejecución activa.    |
| 4   | Servicio de rutas            | Servicio de notificaciones   | gRPC    | Respuesta con la condición de asignación y de viaje.                                               |
| 5   | Servicio de notificaciones   | Servicio de notificaciones   | Interno | Evaluación de las guardas de subida y de bajada antes de aceptar el movimiento.                    |
| 6   | Servicio de notificaciones   | Broker de eventos            | evento  | Publicación de student.scanned con la identidad del estudiante y el tipo de movimiento.            |
| 7   | Broker de eventos            | Servicio de notificaciones   | evento  | Entrega del mensaje al grupo de consumo de escaneos.                                               |
| 8   | Servicio de notificaciones   | Esquema Notification         | Interno | Registro de la bordada con fecha, hora y parada (ON_BOARD u OFF_BOARD).                            |
| 9   | Servicio de notificaciones   | Aplicación móvil (acudiente) | push    | Entrega de la notificación de subida o bajada con la hora y la parada, usando el token registrado. |
| 10  | Servicio de notificaciones   | Aplicación móvil (conductor) | REST    | Confirmación del movimiento registrado en la aplicación del conductor.                             |

Notas de excepción:

- Estudiante asignado a otra ruta: rechazo con mensaje «El estudiante no está asignado a esta ruta» y código 422.
- Bajada sin bordada activa en el trayecto: rechazo con mensaje «No existe bordada activa para el estudiante».
- Escaneo sin ejecución de ruta activa: rechazo con código 409; el flujo exige haber iniciado el trayecto.
- Código vencido o perfil del estudiante INACTIVE: respuesta con la indicación de que el código no está disponible.
- Escaneo repetido: el consumo idempotente del evento evita bordadas duplicadas.
- Intermitencia de red: la aplicación almacena el escaneo en una cola local y reintenta sin perder el orden.

## 3.8 SEQ-08 — Recepción de posición GPS mediante gRPC

Dispositivo GPS Canal gRPC :5001 Geolocalización Esquema Gps Notificaciones Web acudiente

| | | | | |

|-- 1. transmisión de posición {imei, lat, lon, velocidad, rumbo, marca de tiempo}

|------------------>| | | | |

| |-- 2. reenvío ---------------------->| | |

| | |-- 3. \[guarda: IMEI registrado y ACTIVE\] |

| | |-- 4. INSERT GpsLocation | |

| | |-- 5. UPDATE última conexión | |

| | |-- 6. evento GpsLocationReceived | |

| | |---------------------------------->|------------->|

| | | | |-- 7. \[guarda: parada prolongada\]

|<-- 8. canal abierto (confirmación de trama) --------------| | |

| | | | | |

| | |<-- 9. GET /api/v1/locations (última posición) ---|

| | |-- 10. 200 posición y marca de disponibilidad -----|

**Figura 8**. Diagrama de secuencia de recepción de posición GPS mediante gRPC

**Tabla 13**. Mensajes del diagrama de secuencia de recepción de posición GPS mediante gRPC

| Nº  | Emisor                      | Receptor                    | Tipo    | Descripción                                                                                |
| --- | --------------------------- | --------------------------- | ------- | ------------------------------------------------------------------------------------------ |
| 1   | Dispositivo GPS             | Canal gRPC                  | gRPC    | Envío continuo de la posición del bus por el canal persistente de baja latencia.           |
| 2   | Canal gRPC                  | Servicio de geolocalización | gRPC    | Reenvío de la trama al servicio responsable del esquema de geolocalización.                |
| 3   | Servicio de geolocalización | Servicio de geolocalización | Interno | Evaluación de la guarda de dispositivo registrado y activo.                                |
| 4   | Servicio de geolocalización | Esquema Gps                 | Interno | Persistencia de la posición con latitud, longitud, velocidad, rumbo y marca de tiempo.     |
| 5   | Servicio de geolocalización | Esquema Gps                 | Interno | Actualización de la última conexión del dispositivo.                                       |
| 6   | Servicio de geolocalización | Suscriptores                | evento  | Difusión de GpsLocationReceived hacia notificaciones y tableros de seguimiento.            |
| 7   | Servicio de notificaciones  | Servicio de notificaciones  | Interno | Evaluación de la detención prolongada con el umbral configurado.                           |
| 8   | Servicio de geolocalización | Dispositivo GPS             | gRPC    | Confirmación de recepción de la trama para el control del enlace.                          |
| 9   | Aplicación web (acudiente)  | Servicio de geolocalización | REST    | Consulta de la última posición de la ruta, sin necesidad de refresco manual en el cliente. |
| 10  | Servicio de geolocalización | Aplicación web (acudiente)  | REST    | Devolución de la posición con la marca de disponibilidad de la señal.                      |

Notas de excepción:

- IMEI no registrado o dispositivo INACTIVE: la muestra se rechaza y se registra el intento.
- Ausencia de posiciones durante el umbral configurado: se emite el evento de pérdida de señal y la aplicación muestra la última posición conocida con la marca «sin señal» (RF-06.3).
- Interrupción del canal: el dispositivo reintenta la conexión; el servicio conserva la última conexión registrada para calcular la disponibilidad.
- Volumen alto de muestras: la difusión de posiciones es optativa y se destina a modelos de lectura, de modo que no se acoplen consumidores en la ruta síncrona principal.

## 3.9 SEQ-09 — Generación y entrega de alerta con notificación push

Disparador Notificaciones Esquema Notification Broker Móvil acudiente Auditoría

| | | | | |

|-- 1. condición: detención prolongada, incidente o pérdida de señal

|---------------->| | | | |

| |-- 2. INSERT Alert con AlertType y nivel de urgencia |

| |-- 3. INSERT AlertRecipient por cada destinatario |

| |-- 4. evento AlertCreated ------------------>| |

| | | | |-- 5. consume |

| |-- 6. push con la alerta y su ubicación --------------------->|

| | | | |--> 7. el acudiente lee

| |<-- 8. PATCH /api/v1/alerts/{id}/status (lectura) ------------|

| |-- 9. actualiza lectura y atencimiento |

| | | | | |-- 10. registro de la acción

**Figura 9**. Diagrama de secuencia de generación y entrega de alerta con notificación push

**Tabla 14**. Mensajes del diagrama de secuencia de generación y entrega de alerta con notificación push

| Nº  | Emisor                       | Receptor                     | Tipo    | Descripción                                                                                        |
| --- | ---------------------------- | ---------------------------- | ------- | -------------------------------------------------------------------------------------------------- |
| 1   | Disparador                   | Servicio de notificaciones   | evento  | Llegada de la condición de negocio: detención prolongada, reporte de incidente o pérdida de señal. |
| 2   | Servicio de notificaciones   | Esquema Notification         | Interno | Creación de la alerta con su tipo y su nivel de urgencia.                                          |
| 3   | Servicio de notificaciones   | Esquema Notification         | Interno | Creación de un registro de destinatario por cada perfil notificado.                                |
| 4   | Servicio de notificaciones   | Broker de eventos            | evento  | Difusión de AlertCreated hacia los consumidores suscritos.                                         |
| 5   | Broker de eventos            | Servicio de auditoría        | evento  | Consumo del evento para dejar constancia de la acción.                                             |
| 6   | Servicio de notificaciones   | Aplicación móvil (acudiente) | push    | Entrega del aviso con la ubicación del bus, usando el token registrado en el dispositivo.          |
| 7   | Aplicación móvil (acudiente) | Servicio de notificaciones   | Interno | Registro de la lectura del destinatario con su fecha y hora.                                       |
| 8   | Aplicación móvil (acudiente) | Servicio de notificaciones   | REST    | Confirmación del estado de la alerta desde la aplicación.                                          |
| 9   | Servicio de notificaciones   | Esquema Notification         | Interno | Actualización de la lectura y del momento de atencimiento de la alerta.                            |
| 10  | Servicio de auditoría        | Esquema Audit                | Interno | Registro de la acción con fecha, descripción, dirección IP y aplicación de origen.                 |

Notas de excepción:

- Reporte de incidente por el conductor: se envía al recurso de alertas con tipo, descripción opcional y ubicación, y la notificación alcanza a la institución y a los acudientes de los estudiantes a bordo (RF-08.1).
- Acudiente sin token de notificación registrado: la alerta queda como pendiente de entrega y se completa al registrarse el dispositivo.
- Fallo del proveedor de notificaciones: se aplica reintento con espera creciente y se registra el error para su seguimiento.
- Alerta no leída dentro del plazo: permanece en estado pendiente de atención y es visible en el tablero administrativo.

# 4 Diagramas de actividad

Los diagramas de actividad describen procesos de negocio completos, con los carriles de responsabilidad de cada participante, los puntos de decisión y las guardas que condicionan el avance del flujo.

## 4.1 ACT-01 — Proceso de subida y bajada de estudiantes con validaciones

| Conductor (app móvil) | Servicio de notificaciones | Servicio de rutas | Acudiente |

|---|---|---|---|

| (inicio) | | | |

| O | | | |

| \[1\] escanear código | | | |

| | | | | |

| +---------> \[2\] ¿ejecución activa? ---- no ----> \[X\] rechazo 409 | |

| | | sí | | |

| | v | | |

| | \[3\] consulta gRPC <------------+ | |

| | | | |

| | \[4\] ¿estudiante asignado? -- no --> \[X\] rechazo | |

| | | sí | |

| | v | |

| | \[5\] ¿tipo de movimiento? | |

| | / \\ | |

| | ON_BOARD OFF_BOARD | |

| | | | | |

| | | \[6\] ¿bordada activa? -- no --> \[X\] rechazo

| | | | sí | |

| | \[7\] publicar student.scanned | |

| | \[8\] registrar bordada (fecha, hora, parada) | |

| +--------+-------------> \[9\] entregar notificación --->|

| | | (push) |

| v v |

| \[10\] (fin) (fin) |

**Figura 10**. Diagrama de actividad de subida y bajada de estudiantes con validaciones

**Tabla 15**. Elementos del diagrama de actividad de subida y bajada de estudiantes

| Nº  | Elemento                          | Tipo       | Responsable                | Condición o resultado                                                            |
| --- | --------------------------------- | ---------- | -------------------------- | -------------------------------------------------------------------------------- |
| 1   | Escaneo del código del estudiante | Actividad  | Conductor                  | Punto de entrada del proceso en la aplicación móvil.                             |
| 2   | ¿Ejecución de ruta activa?        | Decisión   | Servicio de notificaciones | Guarda: existe un trayecto en curso; en caso negativo se responde con error 409. |
| 3   | Consulta de asignación            | Actividad  | Servicio de rutas          | Verificación por gRPC de la asignación del estudiante a la ruta.                 |
| 4   | ¿Estudiante asignado a la ruta?   | Decisión   | Servicio de notificaciones | Guarda de RF-04.2; en caso negativo se rechaza con código 422.                   |
| 5   | ¿Tipo de movimiento?              | Decisión   | Servicio de notificaciones | bifurcación hacia subida (ON_BOARD) o bajada (OFF_BOARD).                        |
| 6   | ¿Bordada activa?                  | Decisión   | Servicio de notificaciones | Guarda de RF-04.3: solo se registra la bajada si existe subida activa.           |
| 7   | Publicación del evento            | Actividad  | Servicio de notificaciones | Emisión de student.scanned en el broker de eventos.                              |
| 8   | Registro de la bordada            | Actividad  | Servicio de notificaciones | Persistencia con fecha, hora y parada en el esquema de notificaciones.           |
| 9   | Entrega de la notificación        | Actividad  | Servicio de notificaciones | Aviso push al acudiente con hora y parada.                                       |
| 10  | Fin del proceso                   | Nodo final | —                          | El conductor observa la confirmación del movimiento.                             |

## 4.2 ACT-02 — Proceso de atención de una alerta

| Sistema / Disparador | Servicio de notificaciones | Proveedor push | Acudiente | Tablero administrativo |

|----------------------|----------------------------|----------------|-----------|------------------------|

| (inicio) | | | | |

| O | | | | |

| | | | | | |

| \[1\] condición | | | | |

| +--------------> \[2\] crear alerta y destinatarios| | | |

| | \[3\] difundir AlertCreated | | | |

| | \[4\] entregar aviso -------->| | | |

| | | ¿entregado?| | | |

| | +-- no --> \[5\] reintento con espera | | |

| | | sí | | | |

| | +-------------------------------------> \[6\] lectura

| | | | | |

| | <------- \[7\] marcar leída y atendida --------+ | |

| | | |

| | +--> \[8\] registro en auditoría ----------------->|

| | |

| | \[9\] ¿urgencia alta sin atención? -- sí --> \[10\] escalar |

| | (fin)

**Figura 11**. Diagrama de actividad de atención de una alerta

| Nº | Elemento | Tipo | Responsable | Condición o resultado | | 1 | Condición de disparo | Actividad | Sistema | Detención prolongada, incidente reportado o pérdida de señal. | | 2 | Creación de la alerta | Actividad | Servicio de notificaciones | Registra el tipo de alerta, el nivel de urgencia y los destinatarios. | | 3 | Difusión del evento | Actividad | Servicio de notificaciones | Publicación de AlertCreated para los suscriptores. | | 4 | Entrega del aviso | Actividad | Proveedor de notificaciones | Envío mediante el token registrado en el dispositivo del acudiente. | | 5 | Reintento de entrega | Actividad | Servicio de notificaciones | Se aplica cuando el proveedor no confirma la entrega. | | 6 | Lectura por el acudiente | Actividad | Acudiente | La aplicación registra la fecha y hora de lectura del destinatario. | | 7 | Marca de atención | Actividad | Servicio de notificaciones | Actualiza la lectura y el momento en que la alerta fue atendida. | | 8 | Registro de auditoría | Actividad | Servicio de auditoría | Deja constancia de la acción con fecha, IP y aplicación de origen. | | 9 | ¿Urgencia alta sin atención? | Decisión | Servicio de notificaciones | Guarda: el nivel de urgencia del tipo de alerta supera el umbral de escalamiento. | | 10 | Escalamiento | Actividad | Tablero administrativo | La alerta pendiente se muestra al administrador para su gestión. |

## 4.3 ACT-03 — Alta y edición de usuario con eventos

| Administrador (web) | Gestión de usuarios | Broker | Servicio IAM | Esquema Iam |

|---------------------|---------------------|--------|--------------|-------------|

| (inicio) | | | | |

| O | | | | |

| \[1\] formulario | | | | |

| | | | | | |

| \[2\] enviar datos -->| | | | |

| | \[3\] validar datos | | | |

| | | | | | |

| | \[4\] ¿identificación | | | |

| | duplicada? -- sí --> \[X\] rechazo 422 | |

| | | no | | | |

| | \[5\] persistir persona| | | |

| | | | | | |

| | \[6\] publicar \*.created ---->| | |

| | | | \[7\] consumir (grupo iam) |

| | | | |--> \[8\] crear Profile + hash |

| | | | |--> \[9\] publicar profile.created

| <--- \[10\] 201 alta confirmada <-------------+--------+--------------| |

| \[11\] editar cuenta --> \[12\] PUT /api/v1/persons/{id} (sin nuevo evento de alta) |

| | | | | |

| \[13\] desactivar ----> \[14\] PATCH /api/v1/profiles/{id}/status --> \[15\] revocar sesiones

**Figura 12**. Diagrama de actividad de alta y edición de usuario con eventos

**Tabla 16**. Elementos del diagrama de actividad de alta y edición de usuarios con eventos

| Nº      | Elemento                   | Tipo      | Responsable                                         | Condición o resultado                                                       |
| ------- | -------------------------- | --------- | --------------------------------------------------- | --------------------------------------------------------------------------- |
| 1       | Captura del formulario     | Actividad | Administrador                                       | Datos de la persona, el rol y la vinculación familiar.                      |
| 2       | Envío del alta             | Actividad | Administrador                                       | Envío al recurso de personas.                                               |
| 3       | Validación de datos        | Actividad | Servicio de gestión de usuarios                     | Formato de correo, número de identificación y teléfono.                     |
| 4       | ¿Identificación duplicada? | Decisión  | Servicio de gestión de usuarios                     | Guarda de unicidad de documento; en caso afirmativo se rechaza.             |
| 5       | Persistencia de la persona | Actividad | Servicio de gestión de usuarios                     | Registro en el esquema de gestión de usuarios.                              |
| 6       | Publicación del evento     | Actividad | Servicio de gestión de usuarios                     | Emisión de student.created, driver.created, admin.created o parent.created. |
| 7       | Consumo del evento         | Actividad | Servicio de identificación y acceso                 | Grupo de consumo con procesamiento idempotente.                             |
| 8       | Creación del perfil        | Actividad | Servicio de identificación y acceso                 | Hash de la clave de acceso, rol y estado ACTIVE.                            |
| 9       | Confirmación del perfil    | Actividad | Servicio de identificación y acceso                 | Publicación de profile.created.                                             |
| 10      | Alta confirmada            | Actividad | Administrador                                       | Respuesta 201 visible en la lista de cuentas.                               |
| 11 y 12 | Edición de la cuenta       | Actividad | Administrador y servicio de gestión de usuarios     | Actualización de datos sin reemitir el evento de alta.                      |
| 13 a 15 | Desactivación de la cuenta | Actividad | Administrador y servicio de identificación y acceso | Guarda: el perfil pasa a INACTIVE y sus sesiones se revocan (RF-01.3).      |

## 4.4 ACT-04 — Uso excepcional de ruta o bus

| Solicitante (web) | Usos excepcionales | Servicio rutas | Servicio flota | Notificaciones / Auditoría |

|-------------------|--------------------|----------------|----------------|----------------------------|

| (inicio) | | | | |

| O | | | | |

| \[1\] registrar uso | | | | |

| | con motivo y ventana de tiempo | | | |

| +-------------> \[2\] validar datos | | | |

| | | | | | |

| | \[3\] consulta gRPC ---> \[4\] ¿ruta existente? -- no --> \[X\] rechazo

| | | | | sí | | |

| | \[5\] consulta gRPC -----------------> \[6\] ¿bus y conductor

| | | | | asignados? -- no --> \[X\] rechazo

| | | | | | sí | |

| | \[7\] persistir uso excepcional | | |

| | | | | | |

| | \[8\] difundir evento de excepción ---------------------------> \[9\] alerta y registro

| | | | | | |

| | \[10\] ¿ventana vencida? -- sí --> \[11\] cierre del uso (fin)

| | | no |

| | +----> permanece vigente hasta el cierre |

**Figura 13**. Diagrama de actividad de uso excepcional de ruta o bus

**Tabla 17**. Elementos del diagrama de actividad de uso excepcional de ruta o bus

| Nº    | Elemento                    | Tipo                 | Responsable                    | Condición o resultado                                                            |
| ----- | --------------------------- | -------------------- | ------------------------------ | -------------------------------------------------------------------------------- |
| 1     | Registro de la solicitud    | Actividad            | Solicitante                    | Indica el motivo y la ventana de tiempo del uso fuera del patrón operativo.      |
| 2     | Validación de datos         | Actividad            | Servicio de usos excepcionales | Verificación de formato y de la ventana indicada.                                |
| 3 y 4 | Consulta de la ruta         | Actividad y decisión | Servicio de rutas              | Guarda: la ruta debe existir y estar activa.                                     |
| 5 y 6 | Consulta de bus y conductor | Actividad y decisión | Servicio de flota              | Guarda: el bus debe estar activo y el conductor debe ser su responsable vigente. |
| 7     | Persistencia del uso        | Actividad            | Servicio de usos excepcionales | Registro del uso de bus por terceros o del desvío de ruta con su motivo.         |
| 8     | Difusión del evento         | Actividad            | Servicio de usos excepcionales | Emisión del evento de dominio de excepción detectada.                            |
| 9     | Alerta y registro           | Actividad            | Notificaciones y auditoría     | Generación de la alerta correspondiente y constancia de la acción.               |
| 10    | ¿Ventana vencida?           | Decisión             | Servicio de usos excepcionales | Guarda: comparación entre la ventana declarada y la hora actual.                 |
| 11    | Cierre del uso              | Actividad            | Servicio de usos excepcionales | Finalización del uso excepcional registrado.                                     |

## 4.5 ACT-05 — Procedimiento de auditoría de acciones críticas

| Cualquier servicio | Broker de eventos | Servicio de auditoría | Esquema Audit | SUPER_ADMIN (web) |

|--------------------|-------------------|-----------------------|---------------|-------------------|

| (inicio) | | | | |

| O | | | | |

| \[1\] acción crítica | | | | |

| | (autenticación, alta o baja de | | | |

| | cuentas, cambio de rol, borrado)| | | |

| | | | | | |

| +--> \[2\] emitir evento de dominio ->| | | |

| | | | | | |

| | +--> \[3\] consumir de forma asíncrona ------>| | |

| | | \[4\] validar payload | | |

| | | \[5\] INSERT registro --+ | |

| | | | (acción, fecha, IP, aplicación) | |

| | | +--> \[6\] consulta paginada <------|

| | | | | \[7\] filtrar por |

| | | | | fecha, rol y tipo |

| | | \[8\] ¿fallo de consumo? -- sí --> \[9\] reintento e idempotencia

| | | | no | |

| | | (fin) | |

**Figura 14**. Diagrama de actividad de auditoría de acciones críticas

**Tabla 18**. Elementos del diagrama de actividad de auditoría de acciones críticas

| Nº    | Elemento                  | Tipo      | Responsable           | Condición o resultado                                                |
| ----- | ------------------------- | --------- | --------------------- | -------------------------------------------------------------------- |
| 1     | Acción crítica            | Actividad | Cualquier servicio    | Hecho del dominio que exige trazabilidad.                            |
| 2     | Emisión del evento        | Actividad | Servicio de origen    | Publicación del evento de dominio correspondiente.                   |
| 3     | Consumo asíncrono         | Actividad | Servicio de auditoría | El registro nunca bloquea la ruta principal de negocio.              |
| 4     | Validación del contenido  | Actividad | Servicio de auditoría | Verificación de los campos obligatorios del evento.                  |
| 5     | Persistencia del registro | Actividad | Servicio de auditoría | Escritura con acción, fecha, descripción, dirección IP y aplicación. |
| 6 y 7 | Consulta del registro     | Actividad | SUPER_ADMIN           | Consulta paginada con filtros por fecha, rol y tipo de acción.       |
| 8     | ¿Fallo de consumo?        | Decisión  | Servicio de auditoría | Guarda: el mensaje no fue procesado.                                 |
| 9     | Reintento                 | Actividad | Servicio de auditoría | Reproceso con identificador de evento para garantizar idempotencia.  |

# 5 Diagramas de estados

Las máquinas de estados siguientes describen el ciclo de vida de los objetos persistentes. Cada transición se etiqueta con el evento que la provoca y se condiciona por una guarda cuando la regla de negocio lo exige.

## 5.1 EST-01 — Sesión de usuario

+-----------------+

| SIN AUTENTICAR |

+--------+--------+

|

login \[guarda: perfil ACTIVE\]

|

v

+--------------------------+

| ACTIVA |<-----+

+--+--------+--------+-----+ |

| | | |

| logout | | desactivación expiración con

| | | del perfil refresh vigente

v | v (rotación)

+---------+ | +----------+ +---------+

| CERRADA | | | REVOCADA | | VENCIDA |

+----+----+ | +----------+ +----+----+

| | | |

+----------+--------+------------------+

|

nuevo login \[guarda: perfil ACTIVE\] -> ACTIVA

**Figura 15**. Máquina de estados de la sesión de usuario

**Tabla 19**. Transiciones de la máquina de estados de la sesión de usuario

| Estado         | Evento                                                            | Guarda                                   | Acción                                                                          | Destino        |
| -------------- | ----------------------------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------- | -------------- |
| SIN AUTENTICAR | Solicitud de inicio de sesión                                     | Perfil ACTIVE y credenciales válidas     | Emisión del par de tokens (access 15 min, refresh 30 días) y registro de sesión | ACTIVA         |
| SIN AUTENTICAR | Solicitud de inicio de sesión                                     | Perfil INACTIVE o credenciales inválidas | Respuesta 401 sin revelar el motivo                                             | SIN AUTENTICAR |
| ACTIVA         | Expiración del access token                                       | Refresh token vigente (hasta 30 días)    | Rotación del refresh y emisión de un nuevo access token                         | ACTIVA         |
| ACTIVA         | Expiración del access token                                       | Refresh token vencido o reutilizado      | Revocación de la familia de sesiones                                            | VENCIDA        |
| ACTIVA         | Cierre de sesión                                                  | —                                        | Incorporación del jti a la lista negra y revocación de la sesión                | CERRADA        |
| ACTIVA         | Desactivación del perfil o restablecimiento de la clave de acceso | —                                        | Revocación inmediata de todas las sesiones                                      | REVOCADA       |
| VENCIDA        | Solicitud de inicio de sesión                                     | Perfil ACTIVE                            | Emisión de un nuevo par de tokens                                               | ACTIVA         |
| CERRADA        | Solicitud de inicio de sesión                                     | Perfil ACTIVE                            | Emisión de un nuevo par de tokens                                               | ACTIVA         |
| REVOCADA       | Reactivación del perfil y nuevo inicio                            | Perfil en estado ACTIVE                  | Emisión de un nuevo par de tokens                                               | ACTIVA         |

## 5.2 EST-02 — Ejecución de la ruta

+---------------+

+-->| PROGRAMADA |

| +-------+-------+

| |

| inicio de trayecto

| \[guarda: sin viaje activo para el bus\]

| |

| v

| +------------------+

| | EN EJECUCIÓN |<----+ registro de bordadas

| +--------+---------+ +---------------------------------+

| |

| | fin de trayecto

| | \[guarda: sin bordadas de bajada pendientes\]

| |

| v

| +------------------+

| | FINALIZADA |

| +--------+---------+

| |

+------------+ nueva programación de la ruta

v

(fin)

**Figura 16**. Máquina de estados de la ejecución de la ruta

**Tabla 20**. Transiciones de la máquina de estados de la ejecución de la ruta

| Estado       | Evento                               | Guarda                                                | Acción                                                                            | Destino      |
| ------------ | ------------------------------------ | ----------------------------------------------------- | --------------------------------------------------------------------------------- | ------------ |
| PROGRAMADA   | Inicio del trayecto por el conductor | No existe otra ejecución activa para el bus (RF-07.1) | Persistencia de la ejecución con hora de inicio y notificación a los consumidores | EN EJECUCIÓN |
| PROGRAMADA   | Inicio del trayecto por el conductor | Ya existe un viaje activo para el bus                 | Respuesta de rechazo «Ya existe un viaje activo para este bus»                    | PROGRAMADA   |
| EN EJECUCIÓN | Escaneo de subida o bajada           | Ejecución activa y asignación válida del estudiante   | Registro de la bordada y notificación al acudiente                                | EN EJECUCIÓN |
| EN EJECUCIÓN | Fin de trayecto                      | Sin bordadas de bajada pendientes (RF-07.2)           | Registro de la fecha y hora de fin y difusión del ciclo del viaje                 | FINALIZADA   |
| EN EJECUCIÓN | Fin de trayecto                      | Existencia de bordadas de bajada pendientes           | Respuesta de rechazo con el detalle de los estudiantes pendientes                 | EN EJECUCIÓN |
| FINALIZADA   | Nueva programación de la ruta        | La ruta permanece activa                              | Generación de una nueva ejecución programada                                      | PROGRAMADA   |

## 5.3 EST-03 — Alerta

+-------------+

+-->| GENERADA |<---------------------+

| +------+------+ |

| | |

| | entrega push aceptada | reintento de entrega

| | \[guarda: token registrado\] | (espera creciente)

| v |

| +--------------+ |

| | ENTREGADA | |

| +------+-------+ |

| | |

| | lectura y atención |

| | por el destinatario |

| v |

| +--------------+ fallo ---------+

| | ATENDIDA |--------------------> (fin)

| +--------------+

**Figura 17**. Máquina de estados de la alerta

**Tabla 21**. Transiciones de la máquina de estados de la alerta

| Estado    | Evento                                 | Guarda                                                | Acción                                                                      | Destino   |
| --------- | -------------------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------- | --------- |
| GENERADA  | Condición de negocio detectada         | —                                                     | Creación de la alerta con su tipo, su nivel de urgencia y sus destinatarios | GENERADA  |
| GENERADA  | Solicitud de entrega push              | El destinatario tiene token de dispositivo registrado | Envío del aviso con la ubicación del bus                                    | ENTREGADA |
| GENERADA  | Solicitud de entrega push              | Sin token registrado o fallo del proveedor            | Dejar la alerta como pendiente de entrega y programar el reintento          | GENERADA  |
| ENTREGADA | Lectura confirmada por el destinatario | —                                                     | Registro de la fecha y hora de lectura del destinatario                     | ATENDIDA  |
| ENTREGADA | Atención por parte de la institución   | —                                                     | Registro del momento de atencimiento de la alerta                           | ATENDIDA  |
| ATENDIDA  | Cierre del proceso                     | —                                                     | Conservación del historial para la consulta y la auditoría                  | ATENDIDA  |

## 5.4 EST-04 — Trayecto del estudiante (bordada)

+---------------+

+-->| SIN BORDADA |

| +-------+-------+

| |

| escaneo ON_BOARD

| \[guarda: asignado a la ruta y ejecución activa\]

| |

| v

| +---------------+

| | A BORDO |

| +-------+-------+

| |

| escaneo OFF_BOARD

| \[guarda: bordada activa en la ejecución\]

| |

| v

| +---------------+

| | BAJADO |

| +-------+-------+

| |

+-----------+ inicio de una nueva ejecución

v

(fin)

**Figura 18**. Máquina de estados del trayecto del estudiante

**Tabla 22**. Transiciones de la máquina de estados del trayecto del estudiante

| Estado      | Evento                                   | Guarda                                                              | Acción                                                                       | Destino     |
| ----------- | ---------------------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ----------- |
| SIN BORDADA | Escaneo de subida (ON_BOARD)             | El estudiante está asignado a la ruta y existe una ejecución activa | Registro de la bordada con fecha, hora y parada; notificación al acudiente   | A BORDO     |
| SIN BORDADA | Escaneo de subida (ON_BOARD)             | El estudiante pertenece a otra ruta o no hay trayecto activo        | Respuesta de rechazo sin registro                                            | SIN BORDADA |
| A BORDO     | Escaneo repetido de subida               | Ya existe bordada activa del mismo estudiante                       | Consumo idempotente del evento: no se genera un registro duplicado           | A BORDO     |
| A BORDO     | Escaneo de bajada (OFF_BOARD)            | Existe la bordada activa del estudiante en la ejecución             | Registro de la bajada con fecha, hora y ubicación; notificación al acudiente | BAJADO      |
| BAJADO      | Inicio de una nueva ejecución de la ruta | —                                                                   | Reinicio del trayecto para el siguiente servicio                             | SIN BORDADA |

## 5.5 EST-05 — Dispositivo GPS

+-----------------+

+-->| REGISTRADO |

| +--------+--------+

| |

| primera posición recibida por gRPC

| |

| v

| +-----------------+

| | EN LÍNEA |<-------------------+

| +--------+--------+ |

| | |

| | nueva posición | posición recibida

| | (actualiza la conexión) | tras una interrupción

| +--> EN LÍNEA |

| | |

| umbral sin posiciones excedido |

| (evento de pérdida de señal) |

| | |

| v |

| +-----------------+ |

| | SIN SEÑAL |-------------------+

| +--------+--------+

| |

| desactivación lógica del dispositivo (baja lógica)

| |

| v

| +-----------------+

| | INACTIVO |--> REGISTRADO (reactivación)

| +-----------------+

|

+---- conservación del registro histórico y de las posiciones

**Figura 19**. Máquina de estados del dispositivo GPS

**Tabla 23**. Transiciones de la máquina de estados del dispositivo GPS

| Estado               | Evento                              | Guarda                                                  | Acción                                                                              | Destino    |
| -------------------- | ----------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------- |
| REGISTRADO           | Primera posición recibida           | IMEI registrado y perfil de dispositivo ACTIVE          | Persistencia de la posición y actualización de la última conexión                   | EN LÍNEA   |
| REGISTRADO           | Posición recibida                   | IMEI desconocido o dispositivo inactivo                 | Rechazo de la muestra y registro del intento                                        | REGISTRADO |
| EN LÍNEA             | Nueva posición                      | Canal gRPC disponible                                   | Actualización del historial de posiciones y de la marca de última conexión          | EN LÍNEA   |
| EN LÍNEA             | Expiración del umbral de posiciones | Ninguna posición recibida dentro del umbral configurado | Emisión del evento de pérdida de señal y marca «sin señal» en la consulta (RF-06.3) | SIN SEÑAL  |
| SIN SEÑAL            | Posición recibida                   | —                                                       | Reanudación del historial y levantamiento de la marca                               | EN LÍNEA   |
| EN LÍNEA o SIN SEÑAL | Desactivación del dispositivo       | Baja lógica del dispositivo                             | Suspensión de la ingesta de posiciones sin borrar el historial                      | INACTIVO   |
| INACTIVO             | Reactivación del dispositivo        | Dispositivo ACTIVE nuevamente                           | Reanudación de la recepción de posiciones                                           | REGISTRADO |

## 5.6 EST-06 — Perfil de acceso con ACTIVE e INACTIVE

+--------------------------------------+

| ACTIVE |

+---+-------------------+--------------+

| |

| desactivación | inicio de sesión

| por el | y uso de la plataforma

| administrador |

| \[guarda: revoca |

| sesiones\] v

| (operación válida)

v

+--------------+ reactivación

| INACTIVE |----------------------------------> ACTIVE

+------+-------+

|

| persistencia del registro

| (baja lógica, sin borrado físico)

v

(fin)

**Figura 20**. Máquina de estados del perfil de acceso

**Tabla 24**. Transiciones de la máquina de estados del perfil de acceso

| Estado              | Evento                                             | Guarda                                       | Acción                                                                | Destino  |
| ------------------- | -------------------------------------------------- | -------------------------------------------- | --------------------------------------------------------------------- | -------- |
| Creación del perfil | Alta de usuario confirmada por el evento de perfil | Datos válidos y sin identificación duplicada | Asignación de rol, hash de la clave de acceso y estado ACTIVE         | ACTIVE   |
| ACTIVE              | Desactivación por el administrador                 | Revocación de todas las sesiones vigentes    | Bloqueo del acceso y de nuevas asignaciones (RF-01.3)                 | INACTIVE |
| ACTIVE              | Inicio de sesión y operación                       | Perfil ACTIVE y token vigente                | Emisión o renovación de tokens y ejecución de la operación solicitada | ACTIVE   |
| INACTIVE            | Intento de inicio de sesión                        | —                                            | Respuesta 401 con mensaje genérico                                    | INACTIVE |
| INACTIVE            | Reactivación por el administrador                  | Datos de la cuenta íntegros                  | Restablecimiento de la capacidad de autenticación                     | ACTIVE   |
| INACTIVE            | Eliminación lógica definitiva                      | Conservación del registro histórico          | Mantenimiento del registro sin borrado físico para auditoría          | INACTIVE |

# 6 Trazabilidad dinámica ↔ RF/HU

La tabla siguiente vincula cada artefacto dinámico con los requisitos funcionales e historias de usuario que modela, de modo que toda vista dinámica quede justificada y todo requisito con comportamiento relevante aparezca representado.

**Tabla 25**. Trazabilidad de los artefactos dinámicos con requisitos e historias de usuario

| Artefacto                        | Denominación                                         | RF                                 | HU                   | Épica  |
| -------------------------------- | ---------------------------------------------------- | ---------------------------------- | -------------------- | ------ |
| UC-01 / SEQ-01                   | Autenticación, renovación y cierre de sesión         | RF-02.1, RF-02.2                   | HU-IAM-001           | EP-004 |
| UC-02 / SEQ-02 / ACT-03 / EST-06 | Alta y edición de cuentas con eventos                | RF-01.1, RF-01.2, RF-01.3          | HU-USER-004          | EP-004 |
| UC-03 / SEQ-02                   | Vinculación de estudiantes al acudiente              | RF-01.4                            | HU-USER-003          | EP-004 |
| UC-04 / SEQ-03                   | Administración de institución, sedes y cursos        | —                                  | —                    | —      |
| UC-05                            | Administración de la flota de buses                  | RF-03.6                            | HU-FLEET-001         | EP-003 |
| UC-06 / SEQ-04                   | Asignación de conductor al bus                       | RF-03.7                            | HU-BUS-002           | EP-003 |
| UC-07 / SEQ-05                   | Creación y gestión de rutas con paradas              | RF-03.1, RF-03.2, RF-03.3, RF-03.4 | HU-ROUTE-001         | EP-003 |
| UC-08 / SEQ-04                   | Asignación de bus a la ruta                          | RF-03.5                            | HU-ROUTE-005         | EP-003 |
| UC-09                            | Asignación de estudiantes a la ruta con paradas      | RF-03.8                            | HU-ROUTE-008         | EP-003 |
| UC-10 / SEQ-05                   | Programación semanal de la ruta                      | RF-03.1                            | HU-ROUTE-001         | EP-003 |
| UC-11 / SEQ-06 / EST-02          | Inicio y fin de la ejecución de la ruta              | RF-07.1                            | HU-ROUTE-003         | EP-003 |
| UC-12 / SEQ-07 / ACT-01 / EST-04 | Registro de subida por escaneo QR                    | RF-04.2, RF-05.1                   | HU-QR-001            | EP-001 |
| UC-13 / SEQ-07 / ACT-01 / EST-04 | Registro de bajada por escaneo QR                    | RF-04.3                            | HU-QR-002            | EP-001 |
| UC-14                            | Visualización del código QR personal                 | RF-04.1                            | HU-QR-004            | EP-001 |
| UC-15 / SEQ-07                   | Notificación de subida y bajada al acudiente         | RF-05.2, RF-05.3                   | HU-NOTIF-001         | EP-002 |
| UC-16 / SEQ-09 / ACT-02 / EST-03 | Alerta por detención prolongada                      | RF-05.4                            | HU-NOTIF-005         | EP-002 |
| UC-17 / SEQ-09                   | Reporte de incidente y notificación automática       | RF-08.1                            | HU-INCIDENT-001      | EP-006 |
| UC-18                            | Consulta de la ruta asignada por el conductor        | RF-09.2                            | HU-ROUTE-002         | EP-003 |
| UC-19                            | Consulta de la ruta del estudiante por el acudiente  | RF-09.1                            | HU-ROUTE-004         | EP-003 |
| UC-20 / SEQ-08 / EST-05          | Captura, consulta y visualización de la posición GPS | RF-06.1, RF-06.2, RF-06.3          | HU-GPS-002           | EP-005 |
| UC-21 / ACT-04                   | Uso excepcional de ruta o bus                        | —                                  | —                    | —      |
| UC-22 / ACT-05                   | Auditoría de acciones críticas                       | —                                  | —                    | —      |
| UC-23                            | Personalización de tema e idioma                     | RF-10.1, RF-10.2                   | HU-UI-001, HU-UI-002 | EP-007 |

## 6.1 Cobertura por grupo de requisitos

**Tabla 26**. Cobertura de los grupos de requisitos funcionales por artefactos dinámicos

| Grupo RF                          | Requisitos del grupo | Artefactos dinámicos que lo modelan | Cobertura                                                                         |
| --------------------------------- | -------------------- | ----------------------------------- | --------------------------------------------------------------------------------- |
| RF-01 Gestión de usuarios         | 4                    | SEQ-02, ACT-03, EST-06              | Completa                                                                          |
| RF-02 Autenticación               | 2                    | SEQ-01, EST-01                      | Completa                                                                          |
| RF-03 Rutas, flota y asignaciones | 8                    | SEQ-03, SEQ-04, SEQ-05              | Completa                                                                          |
| RF-04 Escaneo QR                  | 3                    | SEQ-07, ACT-01, EST-04              | Completa                                                                          |
| RF-05 Notificaciones de bordada   | 4                    | SEQ-07, SEQ-09, ACT-02, EST-03      | Completa                                                                          |
| RF-06 Rastreo en tiempo real      | 3                    | SEQ-08, EST-05                      | Completa                                                                          |
| RF-07 Gestión de viajes           | 1                    | SEQ-06, EST-02                      | Completa                                                                          |
| RF-08 Incidentes                  | 1                    | SEQ-09, ACT-02, EST-03              | Completa                                                                          |
| RF-09 Consulta de rutas           | 2                    | SEQ-06, SEQ-08                      | Parcial: la consulta es de solo lectura y se describe en la tabla de casos de uso |
| RF-10 Personalización             | 2                    | UC-23                               | Solo cliente: no interviene en servicios ni en eventos                            |

## 6.2 Observaciones de trazabilidad

- Los casos UC-04, UC-21 y UC-22 carecen de requisito formal propio: cubren datos maestros institucionales, usos excepcionales de transporte y auditoría, que el dominio exige aunque no estén numerados entre los requisitos funcionales. Sus procesos se modelan íntegramente en ACT-04 y ACT-05.
- La personalización de tema e idioma se resuelve en el cliente, por lo que no genera intercambio de mensajes con los servicios de dominio ni eventos en el broker.
- El consumo de eventos se realiza con procesamiento idempotente y entrega al menos una vez; los flujos que dependen de la confirmación de un consumidor (bordadas y notificaciones) describen explícitamente el reintento en sus notas de excepción.
- La comunicación síncrona entre servicios se realiza mediante gRPC con tiempos de espera y cortacircuitos; la comunicación con los clientes pasa siempre por la puerta de enlace, que valida el token e inyecta la identidad antes de invocar a cualquier servicio.