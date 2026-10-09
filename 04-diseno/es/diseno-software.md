# Diseño del Software

---

>** Proyecto Guardian Escolar**

| Campo | Detalle |
| --- | --- |
|** Autores**| Johan Smith Santamaría Fernández • Sharik Dayanna Rojas Ibargüen • Juan Pablo Chala Ramírez |
|** Institución**| SENA — Servicio Nacional de Aprendizaje |
|** Ficha**| Ficha de programación 3145556 |
|** Centro**| Centro de formación La Industria, La Empresa y Los Servicios |
|** Ubicación**| Neiva, Huila, Colombia |
|** Fecha**| Octubre 2026 |

---

## Índice de Contenidos


  - [1. Introducción y principios de diseño](#introduccion-y-principios-de-diseno)
  - [2. Modelo arquitectónico](#modelo-arquitectonico)
  - [2.1. Vista de contexto](#vista-de-contexto)
  - [2.2. Vista de contenedores](#vista-de-contenedores)
  - [2.3. Vista de despliegue](#vista-de-despliegue)
  - [2.4. Vista de capas internas de un servicio](#vista-de-capas-internas-de-un-servicio)
  - [2.5. Estilos arquitectónicos adoptados y justificación](#estilos-arquitectonicos-adoptados-y-justificacion)
  - [2.6. Preocupaciones transversales](#preocupaciones-transversales)
  - [3. Patrones de diseño aplicados](#patrones-de-diseno-aplicados)
  - [3.1. Patrones verificados](#patrones-verificados)
  - [3.2. Patrones evaluados no adoptados](#patrones-evaluados-no-adoptados)
  - [4. Interfaces gráficas](#interfaces-graficas)
  - [4.1. Principios de interfaz](#principios-de-interfaz)
  - [4.2. Interfaces web por rol](#interfaces-web-por-rol)
  - [4.3. Interfaces móviles](#interfaces-moviles)
  - [4.4. Mapa de navegación](#mapa-de-navegacion)
  - [4.5. Design tokens](#design-tokens)
  - [4.6. Estados de interfaz](#estados-de-interfaz)
  - [4.7. Accesibilidad y adaptación](#accesibilidad-y-adaptacion)
  - [5. Modelo de datos](#modelo-de-datos)
  - [5.1. Diagrama ER global](#diagrama-er-global)
  - [5.2. Esquema Iam — 5 tablas](#esquema-iam-5-tablas)
  - [5.3. Esquema UserManagement — 4 tablas](#esquema-usermanagement-4-tablas)
  - [5.4. Esquema School — 4 tablas](#esquema-school-4-tablas)
  - [5.5. Esquema Geographic — 1 tabla](#esquema-geographic-1-tabla)
  - [5.6. Esquema Fleet — 4 tablas](#esquema-fleet-4-tablas)
  - [5.7. Esquema Route — 7 tablas](#esquema-route-7-tablas)
  - [5.8. Esquema Gps — 2 tablas](#esquema-gps-2-tablas)
  - [5.9. Esquema Notification — 5 tablas](#esquema-notification-5-tablas)
  - [5.10. Esquema Exceptional — 2 tablas](#esquema-exceptional-2-tablas)
  - [5.11. Esquema Settings — 2 tablas](#esquema-settings-2-tablas)
  - [5.12. Esquema Audit — 2 tablas](#esquema-audit-2-tablas)
  - [5.13. Enums y datos semilla](#enums-y-datos-semilla)
  - [5.14. Índices y restricciones](#indices-y-restricciones)
  - [5.15. Reglas de negocio y retención](#reglas-de-negocio-y-retencion)
  - [6. Contratos de interfaz](#contratos-de-interfaz)
  - [6.1. Recursos REST](#recursos-rest)
  - [6.2. Eventos de dominio (Kafka)](#eventos-de-dominio-kafka)
  - [6.3. Flujo gRPC](#flujo-grpc)
  - [7. Decisiones y trade-offs de diseño](#decisiones-y-trade-offs-de-diseno)

---


- [1. Introducción y principios de diseño](#introduccion-y-principios-de-diseno)
- [2. Modelo arquitectónico](#modelo-arquitectonico)
- [2.1. Vista de contexto](#vista-de-contexto)
- [2.2. Vista de contenedores](#vista-de-contenedores)
- [2.3. Vista de despliegue](#vista-de-despliegue)
- [2.4. Vista de capas internas de un servicio](#vista-de-capas-internas-de-un-servicio)
- [2.5. Estilos arquitectónicos adoptados y justificación](#estilos-arquitectonicos-adoptados-y-justificacion)
- [2.6. Preocupaciones transversales](#preocupaciones-transversales)
- [3. Patrones de diseño aplicados](#patrones-de-diseno-aplicados)
- [3.1. Patrones verificados](#patrones-verificados)
- [3.2. Patrones evaluados no adoptados](#patrones-evaluados-no-adoptados)
- [4. Interfaces gráficas](#interfaces-graficas)
- [4.1. Principios de interfaz](#principios-de-interfaz)
- [4.2. Interfaces web por rol](#interfaces-web-por-rol)
- [4.3. Interfaces móviles](#interfaces-moviles)
- [4.4. Mapa de navegación](#mapa-de-navegacion)
- [4.5. Design tokens](#design-tokens)
- [4.6. Estados de interfaz](#estados-de-interfaz)
- [4.7. Accesibilidad y adaptación](#accesibilidad-y-adaptacion)
- [5. Modelo de datos](#modelo-de-datos)
- [5.1. Diagrama ER global](#diagrama-er-global)
- [5.2. Esquema Iam — 5 tablas](#esquema-iam-5-tablas)
- [5.3. Esquema UserManagement — 4 tablas](#esquema-usermanagement-4-tablas)
- [5.4. Esquema School — 4 tablas](#esquema-school-4-tablas)
- [5.5. Esquema Geographic — 1 tabla](#esquema-geographic-1-tabla)
- [5.6. Esquema Fleet — 4 tablas](#esquema-fleet-4-tablas)
- [5.7. Esquema Route — 7 tablas](#esquema-route-7-tablas)
- [5.8. Esquema Gps — 2 tablas](#esquema-gps-2-tablas)
- [5.9. Esquema Notification — 5 tablas](#esquema-notification-5-tablas)
- [5.10. Esquema Exceptional — 2 tablas](#esquema-exceptional-2-tablas)
- [5.11. Esquema Settings — 2 tablas](#esquema-settings-2-tablas)
- [5.12. Esquema Audit — 2 tablas](#esquema-audit-2-tablas)
- [5.13. Enums y datos semilla](#enums-y-datos-semilla)
- [5.14. Índices y restricciones](#indices-y-restricciones)
- [5.15. Reglas de negocio y retención](#reglas-de-negocio-y-retencion)
- [6. Contratos de interfaz](#contratos-de-interfaz)
- [6.1. Recursos REST](#recursos-rest)
- [6.2. Eventos de dominio (Kafka)](#eventos-de-dominio-kafka)
- [6.3. Flujo gRPC](#flujo-grpc)
- [7. Decisiones y trade-offs de diseño](#decisiones-y-trade-offs-de-diseno)

---


### 1. Introducción y principios de diseño


El presente documento especifica el diseño del software de la plataforma Guardian Escolar: el modelo arquitectónico que estructura el sistema, los patrones de diseño empleados, las interfaces gráficas web y móvil, el modelo de datos persistente y los contratos de integración que enlazan todas las piezas. El diseño traduce los requisitos funcionales y no funcionales del sistema en decisiones concretas y verificables de construcción.
La plataforma está compuesta por una aplicación web Angular 21 con Angular Material 3, una aplicación móvil React + Expo para conductor y acudiente, un API Gateway (Kong OSS) en el borde, nueve microservicios de dominio, un bus de eventos Apache Kafka 4.1.0 (modo KRaft), un canal gRPC de baja latencia y una base de datos relacional única SQL Server 2022 organizada en esquemas.
** Tabla 2. Contenido del documento por sección**
Elemento de diseño
Contenido del documento
Sección
Modelo arquitectónico
Vistas de contexto, contenedores, despliegue y capas; estilos aplicados con justificación
2
Patrones de diseño
Catálogo de patrones verificados y patrones no adoptados
3
Interfaces gráficas
Pantallas web por rol, pantallas móviles, mapa de navegación y design tokens
4
Modelo de datos
Diagrama ER global, diccionario de datos por esquema, enums, semillas, índices y reglas
5
Contratos de interfaz
Recursos REST, eventos Kafka y flujo gRPC
6
Decisiones
Decisiones de diseño, alternativas descartadas y trade-offs
7

Los principios siguientes rigen toda decisión de diseño del sistema y se evalúan en la revisión técnica de cada incremento.
** Tabla 3. Principios de diseño y su aplicación**
Principio
Aplicación concreta en el sistema
Acoplamiento bajo y cohesión alta
Cada microservicio encapsula un contexto acotado (identidad, escolar, flota, rutas, notificaciones, auditoría, configuración, usos excepcionales) con su propio esquema; los servicios se relacionan mediante contratos, nunca mediante lecturas directas de datos ajenos
Responsabilidad única (SRP)
Un caso de uso atiende una operación nominal; los controladores limitan su trabajo a traducir HTTP a caso de uso y devolver la respuesta
Abertura para ampliación, cerrazón para modificación (OCP)
La búsqueda se extiende agregando estrategias nuevas sin modificar los casos de uso; los tipos de alerta y los parámetros de configuración se agregan como datos, no como código
Sustitución de Liskov (LSP)
Las implementaciones de los puertos de datos y los adaptadores remotos cumplen exactamente el contrato de su interfaz de dominio, de modo que las pruebas con dobles en memoria sustituyen al almacén real sin cambio de comportamiento
Principios de interfaz segregada (ISP)
Interfaces de puerto pequeñas y específicas: un puerto por operación de caso de uso (ICreateRouteUseCase, IListRouteUseCase, IStopRepository) en lugar de puertos monolíticos
Inversión de dependencias (DIP)
El dominio declara los puertos; la infraestructura los implementa y el contenedor de inyección de dependencias los enlaza en el arranque
Regla de dependencia
Las dependencias apuntan hacia el interior: infraestructura → aplicación → dominio; el dominio no conoce marcos web, motores de persistencia ni clientes de mensajería
Diseño por contratos
El contrato REST se define antes de la implementación y el código nunca publica un recurso ausente de la definición; los eventos y los contratos RPC se versionan de forma aditiva
Fallop rápido, recupérese con elegancia
Validación de entrada en el borde y en el servicio, comprobación de salud por componente, consumidores idempotentes y respuestas 503 cuando una dependencia crítica no responde
Mínimo privilegio
Autorización por rol y permisos con denegación por defecto; cada servicio de base de datos accede únicamente a su propio esquema
Observabilidad desde el diseño
Identificador de correlación de extremo a extremo, registros estructurados y métricas por componente
Accesibilidad y consistencia visual
Contraste WCAG AA, operación por teclado, indicadores de foco visibles y consumo exclusivo de los design tokens del sistema de diseño
#
** Tabla 4. Convenciones de diseño**
Área
Convención
Almacén relacional
Esquema y tabla en PascalCase; columna en PascalCase; columna de clave ajena <Entidad>Id
Identificadores
UNIQUEIDENTIFIER en las entidades de negocio; TINYINT autoincremento en catálogos; BIGINT autoincremento en auditoría; INT autoincremento en sesiones
Restricciones
Prefijos PK, FK_<Tabla>_<TablaRef>, UQ_<Tabla>_<Columna>, CK_<Tabla>_<regla>, DF_<Tabla>_<Columna>
Ciclo de vida
Baja lógica con Status en valores Active/Inactive; los contratos de interfaz los exponen como ACTIVE/INACTIVE
Superficies de interfaz
Rutas REST bajo /api/v1, sustantivos en plural y kebab-case; payloads en camelCase; identificadores uuid
Eventos
Nombre <entidad>.<acción>; clave de mensaje = identificador de la entidad; carga útil JSON aditiva y sin datos sensibles
Puertos de servicio
Interfaz de caso de uso I<Verbo><Entidad>UseCase; interfaz de almacén I<Entidad>Repository; implementación en la capa de infraestructura
Tiempo
Marcas de fecha y hora en DATETIME2 con huso horario UTC en los flujos nuevos
Comunicación
REST versionado en el borde, gRPC para llamadas internas de baja latencia y Kafka para desacoplamiento asincrónico; diagramas en representación ASCII orientada a lectura en texto plano

### 2. Modelo arquitectónico

La arquitectura de Guardian Escolar se describe en cuatro vistas complementarias: contexto, contenedores, despliegue y capas internas. Las tres primeras siguen la notación de vistas de arquitectura (contexto, contenedores y componentes) adaptada a diagramas ASCII, de modo que el documento sea legible en cualquier medio de texto y no dependa de imágenes ni de notaciones gráficas propietarias.
### 2.1. Vista de contexto
La vista de contexto sitúa el sistema frente a sus actores y sistemas externos. Guardian Escolar se interpone entre la comunidad educativa y dos proveedores externos de nivel gratuito: un proveedor de correo electrónico transaccional y un proveedor de mensajería instantánea.
COMUNIDAD EDUCATIVA
+----------------+  +----------------+  +----------------+  +----------------+
|  Super admin.  |  | Administrador  |  |   Conductor    |  |  Acudiente /   |
|                |  |   del colegio  |  |                |  |   Estudiante   |
+-------+--------+  +-------+--------+  +-------+--------+  +-------+--------+
|                   |                   |                   |
+-------------------+---------+---------+-------------------+
|
v
+--------------------------------------------+
|        GUARDIAN ESCOLAR (plataforma)       |
|  Operación del servicio de transporte      |
|  escolar: identificación, usuarios, flota, |
|  rutas, eventos, notificaciones y auditoría|
+-------------------+------------------------+
|
+-------------------+-------------------+
|                                       |
v                                       v
+--------------------+                 +--------------------+
| Proveedor de correo|                 | Proveedor de       |
| transaccional      |                 | mensajería (WhatsApp)|
+--------------------+                 +--------------------+
** Figura 1. Vista de contexto de la plataforma**
Actores y responsabilidades resumidas:
** Tabla 5. Actores y su interacción con la plataforma**
Actor
Interacción principal con la plataforma
Super administrador
Administra la instancia global: organizaciones, usuarios, permisos, catálogos y parámetros del sistema.
Administrador del colegio
Configura el colegio y sus sedes, cursos, familias, conductores, rutas y horarios; consulta reportes y estados.
Conductor
Consulta su ruta asignada, registra eventos de bordada y estados de viaje, ejecuta las validaciones de parada y consulta mensajes.
Acudiente
Consulta la información de su familia, las rutas y estados de viaje de sus hijos, y envía solicitudes de uso excepcional.
Estudiante
Accede a un conjunto mínimo de consultas propias y de su ruta.
Sistemas externos de mensajería
Reciben las plantillas de notificación por correo y por mensajería instantánea.
### 2.2. Vista de contenedores
La vista de contenedores identifica las unidades desplegables de la plataforma. El sistema es una arquitectura de microservicios compuesta por nueve servicios de dominio y de plataforma, un paso de enrutamiento y agregación de API, un bus de eventos, una base de datos relacional compartida por esquemas lógicos, un almacenamiento de caché y de tokens de refresco y un servicio de mensajería para plantillas de notificación.
+---------------------------+
|  Paso de API y enrutador  |
|      (puerto 8000)        |
+-------------+-------------+
|
+-----------+-----------+--------+-------+-----------+-----------+
|           |           |                |           |           |
v           v           v                v           v           v
+-----------+ +---------+ +---------+   +------------+ +---------+ +-----------+
|    IAM    | | Usuarios| |Escolar  |   |   Flota    | | Rutas   | |Notificac. |
| (8080)    | | (8080)  | | (8080)  |   |   (8080)   | | (8080)  | |  (8080)   |
+-----+-----+ +----+----+ +----+----+   +------+------+ +----+----+ +-----+-----+
|            |           |               |             |           |
+-----------+ +---------+ +---------+                           v           |
|  Usos     | |Auditoría| |Config.  |                    +------------+     |
|excepcional| | (8080)  | | (8080)  |                    | Bus de     |     |
|  (8080)   | +---------+ +---------+                    | eventos    |     |
+-----------+                                             | Kafka      |     |
|                                                   | (9092)     |     |
|                                                   +------+-----+     |
|                                                          |           |
+----------------------+-----------------------------------+-----------+
|
v
+---------------------------------+
|    Base de datos relacional     |
|      SQL Server (1433)          |
|   11 esquemas, 38 tablas        |
+---------------------------------+
Almacenamiento complementario: caché y refresco de sesión.
Canal de plantillas de notificación: servicio externo de mensajería.
** Figura 2. Vista de contenedores de la arquitectura**
Descripción de los contenedores:
** Tabla 6. Contenedores de la arquitectura y su rol**
Contenedor
Rol en la arquitectura
Paso de API y enrutador
Punto de entrada único, enrutamiento hacia los servicios, terminación de TLS, limitación de tasa y política de CORS.
Servicio de identificación y acceso
Emisión de tokens, inicio y cierre de sesión, refresco de token, recuperación de la contraseña conforme a la política vigente y bloqueo de cuenta.
Servicio de gestión de usuarios
Personas, roles, permisos, perfiles y relación acudiente-estudiante.
Servicio de gestión escolar
Colegios, sedes, cursos, asignaciones, horarios y relaciones familiares.
Servicio de flota
Buses, marcas, modelos, mantenimientos y estados de flota.
Servicio de rutas
Rutas, paradas, orden de paradas, ejecuciones, eventos de bordada, estados de viaje y reportes.
Servicio de notificaciones
Plantillas, cola de entrega, destinatarios, reintentos y estado de envío.
Servicio de usos excepcionales
Solicitudes, aprobaciones, límites y registros de escoltas.
Servicio de auditoría
Eventos de auditoría, consulta filtrada y exportación.
Servicio de configuración
Configuración del sistema y catálogos de referencia.
Bus de eventos
Canal de eventos entre servicios (topicos de creación de personas, cambios de estado, eventos de viaje y eventos de telemetría).
Base de datos relacional
Persistencia transaccional, con un esquema lógico por servicio y claves foráneas de apoyo entre esquemas.
Almacenamiento de caché y canal de plantillas
Caché de datos de referencia y refresco de token con latencia baja; salida de los mensajes de notificación hacia los dispositivos finales.
### 2.3. Vista de despliegue
La vista de despliegue muestra cómo se distribuyen los contenedores en los recursos de ejecución. Cada microservicio se empaqueta en su propio contenedor, con su propio ciclo de vida y su propia configuración; la orquestación agrupa los contenedores y reparte las peticiones entre instancias.
INTERNET
|
v
+------------------------+
|   Balanceador y        |
|   paso de API          |
|   (8080 -> 8000)       |
+-----------+------------+
|
+-----------------------+------------------------+
|                       |                        |
v                       v                        v
+--------------+      +--------------+         +--------------+
| Contenedores |      | Contenedores |         | Contenedores |
| de servicio  |      | de servicio  |         | de servicio  |
| dominio A..C |      | dominio D..F |         | plataforma   |
+------+-------+      +------+-------+         +------+-------+
|                       |                      |
+-----------+-----------+----------------------+
|
v
+-----------------------------+       +-----------------------------+
| Base de datos relacional    |       | Bus de eventos              |
| (1433) con volúmenes        |       | (9092) con discos           |
| persistentes                |       | persistentes                |
+-----------------------------+       +-----------------------------+
Caché y refresco: instancia de memoria; secretos: almacén del entorno de ejecución;
observabilidad: métricas y trazas recolectadas de cada contenedor.
** Figura 3. Vista de despliegue de la plataforma**
Consideraciones de despliegue:
Cada servicio expone su propio puerto de aplicación (8080) y solo el paso de API es accesible desde el exterior.
La base de datos y el bus de eventos se despliegan con almacenamiento persistente y no se exponen a Internet.
La escalabilidad horizontal se aplica por servicio según su carga: los servicios de rutas y notificaciones escalan por volumen de eventos; los servicios de lectura escalan por número de peticiones.
Las credenciales y secretos (base de datos, claves de firma de tokens, credenciales de correo y mensajería) se inyectan desde un almacén de secretos y nunca se incluyen en el código ni en los artefactos de despliegue; los estados del viaje y los indicadores de servicio se publican con marcas de tiempo de expiración, de modo que un corte de energía no deje indicadores obsoletos.
### 2.4. Vista de capas internas de un servicio
Cada servicio adopta internamente una arquitectura hexagonal (puertos y adaptadores) organizada en capas. La dependencia apunta siempre hacia el interior: las capas de dominio y de aplicación no conocen a la infraestructura.
+-------------------------------------------------------------------+
|  Adaptadores de entrada (primarios)                               |
|  Controladores REST, manejadores de eventos, servidores gRPC,     |
|  procesos programados                                            |
+-------------------------------+-----------------------------------+
|  invoca casos de uso
v
+-------------------------------------------------------------------+
|  Aplicación (casos de uso)                                        |
|  Orquestación, transacciones, reglas de proceso, autorización     |
+-------------------------------+-----------------------------------+
|  puertos de salida
v
+-------------------------------------------------------------------+
|  Dominio                                                          |
|  Entidades, valores, invariantes, errores de dominio              |
+-------------------------------+-----------------------------------+
|  implementado por
v
+-------------------------------------------------------------------+
|  Adaptadores de salida (secundarios)                              |
|  Accesos a datos, unidades de trabajo, clientes de eventos,       |
|  clientes gRPC, adaptadores de correo y mensajería                |
+-------------------------------------------------------------------+
** Figura 4. Vista de capas internas de un servicio**
Convención de nombres internos: los puertos de salida terminan en Repository o Gateway, los casos de uso terminan en UseCase, las operaciones de consulta terminan en Query y las operaciones de cambio terminan en Command. Los errores de dominio se capturan en la capa de API y se traducen a respuestas de problema con el estado HTTP correspondiente.
### 2.5. Estilos arquitectónicos adoptados y justificación
** Tabla 7. Estilos arquitectónicos adoptados y justificación**
Estilo
Dónde se aplica
Justificación
Microservicios
Conjunto de la plataforma: nueve servicios con ciclo de vida propio
Permite desplegar, escalar y fallar de forma independiente cada dominio; acota el impacto de un defecto a un solo servicio.
Arquitectura hexagonal (puertos y adaptadores)
Interior de cada servicio
Separa reglas de negocio de infraestructura; habilita pruebas unitarias del dominio sin base de datos ni red.
Orientación a eventos
Flujos entre servicios (creación de personas, cambios de estado, eventos de viaje, telemetría)
Desacopla productores y consumidores en el tiempo; permite añadir consumidores nuevos sin tocar al productor.
Capas (n-tiers) dentro de cada servicio
Adaptadores de entrada, aplicación, dominio, adaptadores de salida
Mantiene una dirección única de dependencia y una regla de negocio localizable.
API unificada bajo un paso de enrutador
Exposición externa
Ofrece un único punto de entrada, política de seguridad y límite de tasa comunes, y evita exponer servicios internos.
Base de datos por servicio con esquemas lógicos
Persistencia
Da autonomía de datos a cada servicio y, a la vez, permite integridad referencial de apoyo dentro del motor relacional.
Estilos descartados y por qué: la arquitectura monolítica modular se descartó porque la plataforma prevé crecimiento diferenciado por dominio (rutas y notificaciones crecen con el volumen de viajes; auditoría crece con el volumen de acciones); el servidor sin estado puro (stateless) se matizó porque la sesión de usuario y la caché de referencia requieren estado compartido; la arquitectura orientada a servicios (SOAP) se descartó por su sobrecarga de contrato y su menor adecuación a clientes web y móviles ligeros.
### 2.6. Preocupaciones transversales
Las siguientes preocupaciones se resuelven de forma transversal y no quedan confinadas a un servicio:
** Tabla 8. Preocupaciones transversales y criterios de diseño**
Preocupación
Criterio de diseño
Seguridad
Autenticación por token, autorización por rol y permiso, transporte cifrado, secretos en almacén dedicado y validación de toda entrada.
Disponibilidad
Cada servicio puede reiniciarse sin afectar a los demás; las llamadas entre servicios tienen tiempo límite y reintento acotado; los fallos de envío de notificaciones se reintentan con política de retroceso exponencial.
Observabilidad
Identificador de correlación propagado en peticiones y eventos, métricas por servicio, trazas de extremo a extremo y registro estructurado.
Consistencia
Transacciones locales dentro de cada servicio y consistencia eventual entre servicios mediante eventos compensatorios cuando una operación distribuida falla.
Evolución de contratos
Versionado explícito de la API en la ruta (/api/v1), compatibilidad hacia atrás en los tópicos de eventos y versionado del servicio de RPC en su definición.
Rendimiento
Índices alineados a las consultas de listado, paginación obligatoria en listas, proyección de solo los campos necesarios y caché para datos de referencia poco cambiantes.
Accesibilidad
Contraste mínimo de 4.5:1 para texto, 3:1 para elementos gráficos, navegación completa por teclado y soporte de lector de pantalla.
Trazabilidad de la operación
Toda acción relevante genera un evento de auditoría con actor, acción, entidad, marca de tiempo y origen.

### 3. Patrones de diseño aplicados

Los patrones de esta sección fueron verificados en la implementación del sistema. La tabla los presenta con su intención, el lugar donde se aplica y el mecanismo concreto que lo materializa.
### 3.1. Patrones verificados
** Tabla 9. Patrones de diseño verificados**
Patrón
Intención en el sistema
Aplicación verificada
Arquitectura hexagonal (puertos y adaptadores)
Mantener el dominio independiente de la infraestructura
Cada servicio separa puertos de entrada y de salida de sus adaptadores; el dominio no depende de marcos ni de bases de datos.
Repository (acceso a datos)
Centralizar el acceso a datos y abstraer el motor de persistencia
Puertos de salida tipados por entidad, con implementaciones sobre el ORM del servicio; las consultas de listado están en las implementaciones.
Unidad de trabajo
Garantizar atomicidad en operaciones que tocan varias entidades
Interfaz de unidad de trabajo con confirmación explícita; los casos de uso abren transacción, ejecutan y confirman.
DTO y mapeador
Desacoplar el contrato público del modelo interno
Objetos de transferencia de datos de entrada y de salida con perfiles de mapeo automático entre DTO, dominio y modelo de presentación.
Manejador de problemas (ProblemDetails)
Devolver errores normalizados y documentados
Punto único de traducción de errores internos a respuestas de problema, con estado HTTP, tipo, detalle y extensión de campos.
Estrategia de búsqueda
Componer consultas filtrables sin condicionales dispersos
Cadenas de criterios encadenables (colegio, sede, curso, estado, fechas, texto libre) combinables según la necesidad de cada listado.
Fachada de casos de uso
Exponer una operación única y testeable por caso de negocio
Cada caso de uso (crear persona, asignar ruta, aprobar uso excepcional, registrar bordada) es una clase con una operación pública.
Observador / eventos de dominio
Reaccionar a cambios sin acoplar al productor
La creación de personas, el cambio de estado de viaje y los eventos de telemetría se publican en tópicos y son consumidos por otros servicios.
Adaptador de cliente remoto
Ocultar el transporte de un servicio a otro tras una interfaz
Clientes tipados del servicio de RPC con canal compartido, reintentos y transformación de errores a errores de dominio.
Inyección de dependencias / fábrica gestionada
Componer objetos sin fábricas manuales
El contenedor de ejecución construye controladores, casos de uso, accesos a datos y clientes; los clientes se registran con política de vida compartida.
Servicio alojado (hosted service)
Ejecutar trabajos en segundo plano dentro del ciclo de vida del servicio
Procesos en segundo plano para envío de notificaciones y procesamiento de cola, con cancelación ordenada al detener el servicio.
Interfaz de adaptador de salida
Sustituir infraestructura sin tocar reglas
Puertos propios para envío de correo, envío de mensajería y publicación de eventos, con implementaciones intercambiables por entorno.
Compensación en operación distribuida
Deshacer efectos parciales cuando una operación entre servicios falla
Secuencias con pasos compensatorios ante fallos de llamada entre servicios y confirmación explícita al final.
Punto de extensión por catálogo
Permitir crecer en comportamiento sin modificar código
Los catálogos de configuración (tipos de alerta, estados, motivos) se parametrizan y se consumen como datos, no como condición fija.
### 3.2. Patrones evaluados no adoptados
** Tabla 10. Patrones de diseño evaluados no adoptados**
Patrón
Motivo de no adopción
Decorator
La decoración transversal de peticiones se resuelve en el paso de API (limitación de tasa, cabeceras, CORS) y en middleware del servicio, sin envolver dominio.
CQRS (separación explícita de mando y consulta)
El volumen de lectura no justifica duplicar modelo y persistencia; las consultas se optimizan con proyecciones, índices y paginación.
Saga orquestada con componente dedicado
Las pocas operaciones distribuidas se resuelven con eventos y compensación local; introducir un orquestador añadiría un punto único de fallo.
Malla de servicio
El número de servicios y el tráfico actual no justifican la sobrecarga operativa de un sidecar en cada contenedor.
Modelo de resultado enriquecido (envoltorio de resultado)
Los errores se devuelven con el estado HTTP propio de la operación y un cuerpo de problema normalizado, sin envoltorio adicional en el cuerpo de éxito.

### 4. Interfaces gráficas

La plataforma se entrega en dos superficies: una aplicación web de gestión y una aplicación móvil de consulta y registro en campo. La web se dirige a los perfiles administrativos (super administrador y administrador del colegio) y a las pantallas públicas; la móvil se dirige a conductor, acudiente y estudiante, que operan desde el vehículo o desde el camino. Ambas superficies comparten tokens, componentes y reglas de validación, de modo que la identidad visual y el comportamiento sean un solo sistema.
### 4.1. Principios de interfaz
** Tabla 11. Principios de interfaz y consecuencias de diseño**
Principio
Consecuencia de diseño
Primero la operación
En las pantallas de campo, la acción principal (iniciar viaje, registrar bordada) siempre está a un toque y con tamaño de destino mínimo de 44 píxeles.
Información escaneable
Las listas y paneles muestran el dato decisivo primero (estado, hora, ruta) y el detalle en modal o vista de información.
Confirmación antes de destruir
Toda eliminación o cambio irreversible pasa por un diálogo de confirmación con la consecuencia explícita.
Errores accionables
Cada error se muestra junto al control afectado y explica cómo corregirlo; nunca se informa solo el código del problema.
Accesibilidad desde el diseño
Etiqueta asociada a cada control, orden de tabulación predecible, contraste mínimo cumplido y estados nunca comunicados solo por color.
Consistencia entre superficies
Mismos tokens, mismas reglas de validación y mismos patrones de confirmación en web y móvil.
### 4.2. Interfaces web por rol
4.2.1. Espacio público (sin sesión)
La superficie pública comprende el inicio (presentación del servicio) y el acceso. El inicio comunica las capacidades del producto (monitoreo en tiempo real, alertas, control de asistencia, panel de administración, aplicación para familias, datos seguros) y el registro de institución.
+--------------------------------------------------------------+
| [≡]  GUARDIAN ESCOLAR           [ Contacto ]  [ Iniciar sesión ]  <- 56px
+--------------------------------------------------------------+
| La seguridad escolar en tiempo real                           |
|   GPS en tiempo real · Alertas instantáneas · Asistencia      |
|   Panel de administración · App para familias · Datos seguros |
|   Registro: [Nombre de la institución] [Correo] [Mensaje]     |
|                                              [ Registrar ]    |
+--------------------------------------------------------------+
** Figura 5. Boceto de la página de inicio pública**
+--------------------------------------------------------------------------------+
|                       [ ← ]  GUARDIAN ESCOLAR                                  |
+--------------------------------------------------------------------------------+
|              +--------------------------------+                                 |
|              |  Iniciar sesión               |                                 |
|              |  Correo                        |                                 |
|              |  [________________________]   |                                 |
|              |  Contraseña                    |  <- política de contraseñas      |
|              |  [________________________]   |                                 |
|              |  ¿Olvidaste tu contraseña?     |  <- enlace a recuperación        |
|              |        [ Iniciar sesión ]      |  <- var(--button-apply)          |
|              |  (mensaje de credenciales      |                                 |
|              |   inválidas, si aplica)        |                                 |
|              +--------------------------------+                                 |
+--------------------------------------------------------------------------------+
** Figura 6. Boceto del formulario de inicio de sesión**
4.2.2. Rol super administrador
El panel de super administración concentra la vista global de la instancia: organizaciones, colegios, administradores y alertas recientes.
+--------------------------------------------------------------------------------+
| GUARDIAN ESCOLAR   [ Tema ▾ ] [ Idioma ▾ ] [ Perfil ▾ ]          56px          |
+-------------------+------------------------------------------------------------+
|                   |  Bienvenido                                               |
|   PANEL           |                                                           |
|   +---------------+----------------------------------+---------------------+ |
|   | Colegios      |  [ Tarjetas: colegios, usuarios,    |                     | |
|   | Administrador |        sedes y rutas ]              |                     | |
|   |               |  Últimas alertas                  |                     | |
|   |               |  +----------------------------+  |                     | |
|   |               |  | hora | tipo | colegio      |  |                     | |
|   |               |  +----------------------------+  |                     | |
|   |               |  | ...  | ...  | ...          |  |                     | |
|   |               |  +----------------------------+  |                     | |
|   |               |  [ Ver todo ]                     |                     | |
|   +---------------+-----------------------------------+---------------------+ |
|   200px           sidebar con ítems activos resaltados                        |
+-------------------+------------------------------------------------------------+
** Figura 7. Boceto del panel de super administración**
Gestión de colegios y de administradores repite la misma plantilla de módulo (lista + modal de información + modal de edición + diálogo de eliminación):
+--------------------------------------------------------------------------------+
| [←]  Colegios                                    [ + Nuevo colegio ]           |
+--------------------------------------------------------------------------------+
|  [ Buscar por nombre.......... ]   Filtro: [Estado ▾]   Resultados: 12         |
+--------------------------------------------------------------------------------+
|  Nombre            | Ciudad      | Sedes | Estado    | Acciones               |
|  San José           | Bogotá      |   3   | Activo    | [i] [✎] [🗑]           |
|  Norte Abierto      | Trujillo    |   1   | Inactivo  | [i] [✎] [🗑]           |
+--------------------------------------------------------------------------------+
|  < 1  2  3 ... 6 >                                                             |
+--------------------------------------------------------------------------------+
[i] -> modal de información (RecordInformation);  [✎] -> modal de edición;  [🗑] -> diálogo de eliminación (DeleteRecord)
** Figura 8. Boceto del módulo de gestión de colegios**
4.2.3. Rol administrador del colegio
El panel administrativo agrupa las operaciones del colegio: personas (usuarios, padres, conductores, familias), transporte (buses, paradas, rutas) y perfil.
+--------------------------------------------------------------------------------+
| GUARDIAN ESCOLAR   [ Tema ▾ ] [ Idioma ▾ ] [ Perfil ▾ ]                       |
+-------------------+------------------------------------------------------------+
|   USUARIOS        |  Bienvenido, administrador                                |
|   - Usuarios      |                                                           |
|   - Padres        |  +----------------+  +----------------+                    |
|   - Conductores   |  | Estudiantes    |  | Buses          |                    |
|   - Familias      |  |      120       |  |       20       |                    |
|   ----------------|  +----------------+  +----------------+                    |
|   TRANSPORTE      |  +----------------+  +----------------+                    |
|   - Buses         |  | Rutas          |  | Paradas        |                    |
|   - Paradas       |  |       20       |  |       60       |                    |
|   - Rutas         |  +----------------+  +----------------+                    |
|   ----------------|                                                           |
|                   |  Últimas alertas del colegio                               |
|                   |  +----------------------------------------------------+  |
|                   |  | 08:12  Desvío de ruta      Ruta 04     [Ver]       |  |
|                   |  | 07:55  Bordada no registrada  Ruta 02  [Ver]       |  |
|   +---------------+--------------------------------------------------------+  |
+-------------------+------------------------------------------------------------+
** Figura 9. Boceto del panel del administrador del colegio**
Ficha de registro de estudiante (plantilla de formulario de la aplicación; los demás formularios comparten estructura):
+--------------------------------------------------------------------------------+
|  Register student                                                              |
|  +--------------------------------------------------------------------------+  |
|  | Nombre* [________________]      | Apellido* [________________] |  |
|  | Tipo de doc.* [Documento      ▾]      | N.º de doc.  [________________] |  |
|  | Fecha nac.* [aaaa-mm-dd     ]       | Género       [___________ ▾]   |  |
|  | Colegio* [Colegio        ▾]      | Sede* [Sede        ▾]   |  |
|  | Curso* [Curso          ▾]      | Grupo        [Grupo       ▾]   |  |
|  | Acudiente* [Buscar persona ....]   | Estado* [Activo      ▾]   |  |
|  |                                                                          |  |
|  |                [ Cancelar ]   [ Guardar ]                                |  |
|  +--------------------------------------------------------------------------+  |
|  (errores en línea bajo cada campo, solo cuando el control fue tocado)        |
+--------------------------------------------------------------------------------+
** Figura 10. Boceto del formulario de registro de estudiante**
4.2.4. Perfiles con acceso web limitado (conductor, acudiente, estudiante)
Los perfiles de campo conservan en la web un espacio de sesión reducido: información personal, cambio de datos de cuenta, idioma y tema. Las operaciones de consulta en ruta y los registros de campo se resuelven en la aplicación móvil.
+--------------------------------------------------------------------------------+
| [←]  Mi perfil                                                                 |
+--------------------------------------------------------------------------------+
|  +---------------------------+  +------------------------------------------+   |
|  |  (avatar)                 |  |  Nombre completo                         |   |
|  |                           |  |  Correo electrónico                       |   |
|  |  Nombre Apellido          |  |  Teléfono de contacto                     |   |
|  |  Rol                      |  |  Estado de la cuenta                     |   |
|  +---------------------------+  |  [ Editar información ]                   |   |
|                                 +------------------------------------------+   |
|  Seguridad de la cuenta                                                        |
|  - Cambio de correo electrónico      -> flujo en cuatro pasos con código       |
|  - Cambio de contraseña              -> flujo en tres pasos con código de longitud seis |
|  - Cambio de teléfono o contacto     -> flujo en cuatro pasos con código       |
|  Preferencias:  Idioma [Español ▾]        Tema [Azul claro ▾]                |
+--------------------------------------------------------------------------------+
** Figura 11. Boceto del perfil con acceso web limitado**
Flujo de cambio de un dato sensible (correo, teléfono) en cuatro pasos con confirmación visible y retorno al perfil; el cambio de contraseña aplica la política de contraseñas y usa tres pasos:
Paso 1               Paso 2                Paso 3             Paso 4
+--------------+    +------------------+    +---------------+    +----------------+
| Solicitar    | -> | Validar código   | -> | Ingresar el   | -> | Validar código  |
| el cambio    |    | de seis         |    | nuevo valor   |    | final           |
| (dato actual)|    | caracteres      |    |               |    |                 |
+--------------+    +------------------+    +---------------+    +----------------+
** Figura 12. Flujo de cambio de un dato sensible en cuatro pasos**
4.2.5. Matriz de componentes y validaciones por formulario
** Tabla 12. Matriz de componentes y validaciones por formulario**
Pantalla / formulario
Componentes de interfaz
Validaciones aplicadas
Inicio de sesión
Campo de correo, campo de contraseña, botón primario, mensaje en línea
Correo con formato válido; la política de contraseñas exige como mínimo ocho caracteres con mayúscula, minúscula, número y carácter especial; el error de credenciales se muestra en la vista y no revela cuál de los dos datos falló.
Recuperación de acceso (paso 1)
Campo de correo, botón primario
Correo con formato válido; respuesta genérica para existencia de la cuenta.
Recuperación de acceso (paso 2)
Seis campos segmentados, botón primario
Código alfanumérico de longitud seis; foco y orden de tabulación preservados entre los seis campos; error en línea.
Recuperación de acceso (paso 3)
Campo de contraseña, campo de confirmación, botón primario
coincidencia de ambas contraseñas; se aplica la política de contraseñas con un mínimo de ocho caracteres, mayúscula, minúscula, número y carácter especial; validación al enviar con todos los controles marcados como tocados.
Registro de estudiante
Campos de identidad, selectores de colegio/sede/curso/grupo, buscador de acudiente, selector de estado
Obligatoriedad de los campos marcados con asterisco; documento sin duplicados en el colegio; acudiente existente y con rol de padre; fecha de nacimiento no futura.
Registro de conductor
Campos de identidad, licencia, teléfono, selector de estado
Obligatoriedad; teléfono en formato internacional; vigencia de la licencia informada antes de aceptar.
Registro de bus
Placa, marca, modelo, capacidad, número interno
Obligatoriedad; placa sin duplicados; capacidad entera positiva.
Registro de ruta
Nombre, origen, destino, hora de inicio, hora de fin, paradas en orden
Obligatoriedad; hora de fin posterior a la hora de inicio; al menos una parada; orden de paradas único.
Registro de parada
Nombre, dirección, latitud, longitud
Obligatoriedad; latitud entre −90 y 90; longitud entre −180 y 180.
Cambio de correo o contacto
Campo de correo o teléfono, campos de código
Formato del dato nuevo; código de longitud seis; verificación en dos vueltas para evitar el uso del dato anterior.
Cambio de contraseña autenticado
Campo de contraseña actual, nuevas contraseñas, código
Política de contraseñas aplicada a la nueva contraseña; no coincidir con la contraseña actual; código de longitud seis.
Eliminación de registro
Diálogo de confirmación
Confirmación explícita de la acción destructiva; en registros con dependencias, aviso de las entidades afectadas.
### 4.3. Interfaces móviles
La aplicación móvil se organiza en tres zonas: barra superior, área de contenido y navegación inferior. Las pantallas cubren el acceso, la operación de ruta del conductor, la consulta del acudiente y la administración de la cuenta.
Acceso:
+---------------------------+   +---------------------------+
|  GUARDIAN ESCOLAR         |   |  Recuperar acceso         |
|  Correo                   |   |  Correo                   |
|  [___________________]   |   |  [___________________]   |
|                           |   |        [ Continuar ]      |
|  Contraseña               |   |                           |  <- política de contraseñas
|  [___________________]   |   |  +---------------------+  |
|                           |   |  | | | | | | |         |  |
|  ¿Olvidaste tu           |   |  |código de seis       |  |
|  contraseña?              |   |  |caracteres segmentado|  |  <- longitud seis
|        [ Iniciar sesión ] |   |  +---------------------+  |
|                           |   |        [ Validar ]        |
+---------------------------+   +---------------------------+
** Figura 13. Bocetos móviles de acceso y recuperación**
Conductor: operación de ruta con registro de bordada y aviso de sincronización en línea:
+---------------------------+   +---------------------------+
| [≡]  Mi ruta        [🔔] |   | [≡]  Mi ruta        [🔔] |
+---------------------------+   +---------------------------+
|  Ruta Centro               |   |  Ruta Centro               |
|  Salida 07:10   4 de 12    |   |  Próxima parada             |
|  +---------------------+   |   |  +-----------------------+  |
|  | Próxima parada:     |   |   |  | Colegio San José      |  |
|  | Colegio San José    |   |   |  | 07:42   (falta 6 min) |  |
|  | 07:42    1.2 km     |   |   |  +-----------------------+  |
|  +---------------------+   |   |                             |
|                             |   |  Estudiantes en esta parada |
|  Lista de paradas           |   |  [x] Juan García   07:41    |
|  1. Av. Norte      07:15 OK |   |  [ ] María Ruiz    --       |
|  2. Calle 12       07:28 OK |   |  [ ] Carlos Silva   --       |
|  3. Colegio S.J.   07:42 .. |   |                             |
|  4. La Finlandia   07:55    |   |  [ Registrar bordada ]      |
|  [ Registrar bordada ]      |   |  (botón principal, 44px)     |
+---------------------------+   +---------------------------+
** Figura 14. Boceto móvil de la operación de ruta del conductor**
Estado sin conexión del conductor y monitor de viaje:
+---------------------------+   +---------------------------+
| [←]  Mi ruta   [Sin red]  |   | [←]  Viaje en curso  [⏸]  |
|      3 cambios pendientes |   |                           |
+---------------------------+   |  Estado: EN RUTA          |
|                             |   |  Inicio: 07:10            |
|  Los registros se guardarán |   |  Paradas completadas: 4   |
|  y se sincronizarán cuando  |   |  Estudiantes a bordo: 18  |
|  vuelva la conexión.        |   |                           |
|                             |   |  [ Pausar ]  [ Finalizar ]|
|  Pendientes:                |   |                           |
|  - Bordada 07:42 Colegio SJ |   |  (la finalización exige   |
|  - Estado 07:55 La Finlandia|   |   confirmación)           |
|                             |   +---------------------------+
|  [ Reintentar sincronizar ] |
+---------------------------+
** Figura 15. Boceto móvil sin conexión y monitor de viaje**
Acudiente: familia y ruta en vivo:
+---------------------------+   +---------------------------+
| [≡]  Mi familia     [🔔] |   | [≡]  Ruta        [🔔]    |
+---------------------------+   +---------------------------+
|  Titular                   |   |  Ruta Centro              |
|  +---------------------+   |   |  +---------------------+  |
|  | Ana Gómez   Acudiente|   |   |  |  Mapa del recorrido |  |
|  +---------------------+   |   |  |  (con resumen en     |  |
|                             |   |  |   texto alternativo) |  |
|  Otros miembros            |   |  +---------------------+  |
|  +---------------------+   |   |                           |
|  | Juan García  Estudiante|  |   |  Próxima llegada: 07:52  |
|  | Ruta Centro  [Ver]    |   |   |  Parada: Calle 12        |
|  +---------------------+   |   |  Estado: EN RUTA          |
|  +---------------------+   |   |                           |
|  | María Ruiz  Estudiante|  |   |  Paradas                  |
|  | Sin ruta     --      |   |   |  1. Av. Norte     07:15 ✓ |
|  +---------------------+   |   |  2. Calle 12       07:28 ✓ |
|                             |   |  3. Colegio S.J.  07:42   |
|  +---------------------+   |   +---------------------------+
|  | Solicitud de uso     |   |
|  | excepcional [Enviar] |   |
|  +---------------------+   |
+---------------------------+
** Figura 16. Boceto móvil de familia y ruta del acudiente**
### 4.4. Mapa de navegación
WEB
|
+-- / (redirección) -------------------------------------> /home
|
+-- /home ................................ público  (+ /contact)
|
+-- /auth
|     +-- /login ..................................... público
|     +-- /recuperar-acceso
|           +-- /email ...... solicitud (política de contraseñas)
|           +-- /code ....... validación de código de longitud seis
|           +-- /reset ...... nueva contraseña (política de contraseñas)
|
+-- /panel-administrador ................... SUPER_ADMIN, ADMIN
|     +-- /admin/usuarios, /padres ................. ADMIN
|     +-- /admin/conductores, /familias ............ ADMIN
|     +-- /admin/buses, /admin/paradas, /admin/rutas ......... ADMIN
|     +-- /admin/perfil .................... SUPER_ADMIN, ADMIN
|           +-- /cambio-correo y /cambio-contacto: dato -> /code-first -> /reset -> /code-second
|           +-- /cambio-clave: /email -> /code -> /reset
|
+-- /panel-superadministrador .............. SUPER_ADMIN
|     +-- /superadmin/admins y /superadmin/colegios  SUPER_ADMIN
|
+-- /* (ruta no definida) ------------------------------> /home
MÓVIL
|
+-- Acceso: login y recuperar acceso (email -> código -> política de contraseñas)
|
+-- Conductor
|     +-- Mi ruta (paradas, próxima parada, bordada)
|     +-- Viaje en curso y sin conexión (pendientes, reintento)
|     +-- Notificaciones
|
+-- Acudiente
|     +-- Mi familia (titular, miembros, uso excepcional)
|     +-- Ruta (mapa con resumen de texto, paradas, estado)
|     +-- Notificaciones y mis datos
|
+-- Estudiante: consulta de ruta y de datos propios
|
+-- Cuenta compartida: perfil, seguridad, apariencia, idioma, acerca de,
política de privacidad y cierre de sesión con confirmación
** Figura 17. Mapa de navegación de la plataforma**
Reglas de navegación: la raíz redirige al inicio; una ruta no definida redirige al inicio; los flujos de cambio de datos son secuencias paso a paso con retorno explícito al perfil; el panel del administrador y el del super administrador son rutas distintas con menú lateral propio; el acceso a cada panel se decide por rol en el lado del servidor, no solo por la interfaz.
### 4.5. Design tokens
Los design tokens son la única fuente de valores visuales: los componentes referencian variables, no colores ni tamaños sueltos.
Paleta base:
** Tabla 13. Paleta base de design tokens**
Token
Valor
Uso
--bg-color
#F0F4F8
Fondo general de las vistas
--card-bg
#FFFFFF
Fondo de tarjetas y modales
--card-secondary-bg
#F8FAFC
Fondo secundario de tarjetas
--text-color
#111827
Texto principal
--title-color
#1A2E4A
Títulos
--text-secondary
#64748B
Texto secundario y metadatos
--border-color
#E2E8F0
Bordes y separadores
--color-navbar
#1A56DB
Barra superior y panel lateral
--accent-color
#003AB8
Enlaces y énfasis
--hover-color
#2061EC
Estado de paso del puntero
--button-apply
#1A56DB
Botón primario (aplicar, guardar, continuar)
--button-cancel
#FFFFFF
Botón secundario (cancelar, volver)
--alert-bg
#FEF2F2
Fondo de alertas
--alert-title
#7F1D1D
Título de alerta
--alert-text
#374151
Texto de alerta
--alert-icon
#DC2626
Icono de alerta
--input-bg
#FFFFFF
Fondo de campos de formulario
--input-border
#CBD5E1
Borde de campos de formulario
--modal-bg
#FFFFFF
Fondo de diálogos
--mat-sys-surface
#FFFFFF
Superficie del sistema Material
Temas (ocho combinaciones; el token de barra y el del botón primario cambian juntos):
** Tabla 14. Combinaciones de tema y tokens de color**
Tema
--color-navbar
--button-apply
light-theme-blue
#1A56DB
#1A56DB
light-theme-green
#16A34A
#16A34A
light-theme-yellow
#D4A017
#D4A017
light-theme-red
#B42318
#B42318
dark-theme-blue
#0F2E6B
#3B82F6
dark-theme-green
#0B4A24
#22C55E
dark-theme-yellow
#B8860B
#FACC15
dark-theme-red
#5B1215
#DC2626
Tipografía:
** Tabla 15. Valores de tipografía**
Uso
Valor
Familia base
"Segoe UI", Arial, sans-serif
Familia Material
Roboto
Título de barra
20px, peso 500
Título de formulario
1.5rem
Texto secundario
0.85rem a 1rem, color --text-secondary
Etiquetas
0.78rem, peso 500, mayúsculas
Iconos de ilustración
1.5rem
Espaciado, radios y sombras:
** Tabla 16. Espaciado, radios y sombras**
Uso
Valor
Escala de espaciado
4px, 8px, 12px, 16px, 24px, 32px
Controles compactos
radio de 5px a 10px
Tarjetas y modales
radio de 16px
Botones de formulario existentes
radio de 50px
Tarjetas
sombra 0 4px 12px rgba(0, 0, 0, 0.08)
Modales
sombra propia de modal (--shadow-modal)
Estructura y comportamiento:
** Tabla 17. Estructura y comportamiento de los componentes**
Elemento
Valor
Barra superior
altura 56px, fondo --color-navbar, título 20px
Panel lateral
ancho 200px, fondo --color-navbar; ítems con radio 8px y separación 2px; paso con superposición blanca al 8%; activo con superposición blanca al 15% y peso 500
Densidad Material
0
Paleta Material
primaria tipo Azure, terciaria tipo azul, tipografía Roboto
Contraste
4.5:1 para texto y 3:1 para elementos gráficos
Objetivo táctil (móvil)
mínimo 44px
Punto de quiebre del panel lateral público
se oculta por encima de 1024px
Punto de quiebre del panel de información en flujos de sesión
se oculta por debajo de 1000px
Formularios y tarjetas
pasan de dos columnas a una en pantallas pequeñas
### 4.6. Estados de interfaz
** Tabla 18. Estados de interfaz y patrones visuales**
Situación
Patrón visual
Error de formulario
Mensaje en línea bajo el campo, borde inválido y bloque de error tipado
Credenciales inválidas
Mensaje visible en la vista de acceso sin distinguir qué dato falló
Confirmación
Diálogo de confirmación con la consecuencia de la acción
Carga y éxito
Indicador de progreso o control deshabilitado durante la petición; mensaje visible antes de la navegación de retorno.
Sin conexión (móvil)
Insignia visible, contador de cambios pendientes y acción de reintentar sincronización
Vacío (listas)
Estado vacío con la acción de creación como siguiente paso
### 4.7. Accesibilidad y adaptación
Todo control tiene etiqueta asociada; el texto de relleno no sustituye a la etiqueta.
El orden de tabulación sigue el orden visual y se preserva en los campos segmentados de código.
Ningún estado se comunica solo por color: siempre acompaña texto o icono.
El mapa en vivo de la ruta se acompaña de un resumen textual de la posición, para no depender de la representación gráfica.
Los iconos sueltos reciben etiqueta accesible o mensaje emergente.
No se permite desplazamiento horizontal ni controles superpuestos en ningún punto de quiebre, y el texto se puede ampliar sin pérdida de contenido ni solapamientos.

### 5. Modelo de datos

La plataforma se persiste en una base de datos relacional única sobre SQL Server 2022 con once esquemas lógicos, uno por contexto de dominio, y treinta y ocho tablas en total. Cada servicio accede únicamente a su esquema; las claves foráneas que cruzan esquemas existen para garantizar integridad referencial dentro del motor, no para habilitar lecturas directas entre servicios. El esquema físico se mantiene con migraciones versionadas que se aplican en orden determinista en cada arranque.
Convenciones del modelo:
** Tabla 19. Convenciones del modelo de datos**
Convención
Regla aplicada
Llave primaria
Identificador único global UNIQUEIDENTIFIER en las entidades de negocio; TINYINT autoincremental en catálogos pequeños; BIGINT autoincremental en la bitácora de auditoría y en el registro de errores.
Llaves foráneas
Declaradas como restricción de integridad referencial; algunas se crean junto con la tabla y otras se añaden por separado para respetar el orden de creación entre esquemas.
Fechas y horas
DATETIME2 para marcas de fecha y hora, DATE para vencimientos y TIME para horarios.
Coordenadas geográficas
DECIMAL(10,7) para latitud y longitud, con precisión suficiente para seguimiento vehicular.
Ciclo de vida
Columna Status con valores ACTIVE e INACTIVE (baja lógica) en casi todas las entidades; el borrado físico se reserva para datos de prueba.
Nulabilidad
Solo se declara NULL cuando el dato es opcional por naturaleza (fecha de fin de vigencia, sitio web, segundo código de confirmación).
Restricciones de dominio
Comprobaciones en línea para estados, tipos de documento, días de la semana y tipo de bordada; unicidad sobre documentos, placas, IMEI, códigos de dispositivo y números de licencia.
### 5.1. Diagrama ER global
+--------------------------------------------------------------+
| Geographic: City                                              |
+----------------------------+---------------------------------+
| CityId (City 1 : N School)
+----------------------------v---------------------------------+
| School: School, SchoolCampus, Course, SchoolAdmin            |
+-------------------+--------------------+---------------------+
| CampuseId (1 : N)   | PersonId (1 : 1)
+-------------------v--------------------v---------------------+
| UserManagement: Person, Family, FamilyMember, DriverLicense  |
+-------------------+--------------------+---------------------+
| PersonId (1 : 1)    | ProfileId (1 : N)
+-------------------v--------------------v---------------------+
| Iam: Profile, Role, Permissions, PermissionsRole, SessionProfile
|     FK: PersonId, CampuseId, RoleId                          |
+-------------------+--------------------+---------------------+
| ProfileId (1 : N)   | CampuseId (1 : N)
+-------------------v--------------------v---------------------+
| Fleet: Brand, Model, Bus, DriverAssignemts                   |
|     FK: ModelId, CampuseId, GpsDeviceId, ProfileId           |
+-------------------+--------------------+---------------------+
| GpsDeviceId (1 : 1) | BusId y RouteId
+-------------------v--------------------v---------------------+
| Gps: GpsDevice, GpsLocation                                  |
+--------------------------------------------------------------+
| Route: Route, Stop, RouteStop, RouteBusAssignments,          |
|        RouteStudentAssignments, RouteSchedule, RouteExecution|
|     FK: CampuseId, CityId, SchoolId, BusId, ProfileId        |
+----------------------------+---------------------------------+
| RouteExecutionId (1 : N)
+----------------------------v---------------------------------+
| Notification: Boarding, Alert, AlertType, AlertRecipient,    |
|               DeviceToken                                    |
|     FK: RouteExecutionId, RouteStopId, AlertTypeId, ProfileId|
+--------------------------------------------------------------+
| Exceptional: ExceptionalDriverUsage, ExceptionalRouteUsage   |
|     FK: BusId, RouteId, ProfileId                            |
| Audit: Audit (FK ProfileId), LogErrors                       |
| Settings: PasswordPolicies (política de contraseñas),        |
|           SecuritySettings — sin llave foránea               |
+--------------------------------------------------------------+
** Figura 18. Diagrama ER global del modelo de datos**
Lectura del diagrama: la ciudad afilia a los colegios; el colegio despliega sedes y cursos; la sede es el punto de anclaje de perfiles, buses y rutas; el perfil (identidad y rol) es el nodo que conecta personas, familias, licencias, conductores, asignaciones, sesiones, alertas, bordadas, usos excepcionales y auditoría; la ruta se compone de paradas ordenadas y de asignaciones de bus y estudiantes; la ejecución de ruta es el contenedor de las bordadas y de las alertas de un trayecto real; y el dispositivo de geolocalización entrega las posiciones que alimentan la ejecución.
### 5.2. Esquema Iam — 5 tablas
** Tabla 20. Columnas del esquema Iam**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
Role
Id
TINYINT identity
NOT NULL
PK
Identificador del rol
Role
Name
VARCHAR(20)
NOT NULL
UNIQUE
Nombre del rol
Role
Description
VARCHAR(100)
NOT NULL
—
Alcance del rol
Role
Status
VARCHAR(20)
NOT NULL
—
Ciclo de vida del rol
Permissions
Id
TINYINT identity
NOT NULL
PK
Identificador del permiso
Permissions
TypePermissions
VARCHAR(100)
NOT NULL
—
Código del permiso (recurso.acción)
Permissions
Description
VARCHAR(150)
NOT NULL
—
Descripción legible del permiso
PermissionsRole
Id
TINYINT identity
NOT NULL
PK
Identificador de la asignación
PermissionsRole
PermissionsId
TINYINT
NOT NULL
FK → Permissions
Permiso asignado
PermissionsRole
RoleId
TINYINT
NOT NULL
FK → Role
Rol receptor
Profile
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identidad de acceso
Profile
PersonId
UNIQUEIDENTIFIER
NOT NULL
FK → Person
Persona titular del perfil
Profile
CampuseId
UNIQUEIDENTIFIER
Sí
FK → SchoolCampus
Sede a la que pertenece el perfil
Profile
PasswordHash
VARCHAR(100)
NOT NULL
—
Resumen protegido de la contraseña (política de contraseñas)
Profile
RoleId
TINYINT
NOT NULL
FK → Role
Rol vigente del perfil
Profile
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
SessionProfile
Id
INT identity
NOT NULL
PK
Identificador de la sesión
SessionProfile
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Perfil de la sesión
SessionProfile
DateStart
DATETIME2
NOT NULL
—
Inicio de la sesión
SessionProfile
DateEnd
DATETIME2
NOT NULL
—
Cierre de la sesión
SessionProfile
IPAddress
VARCHAR(50)
NOT NULL
—
Origen de la sesión
SessionProfile
StatusSession
VARCHAR(50)
NOT NULL
—
Estado de la sesión
### 5.3. Esquema UserManagement — 4 tablas
** Tabla 21. Columnas del esquema UserManagement**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
Person
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identidad de la persona
Person
Name
VARCHAR(50)
NOT NULL
—
Nombre
Person
LastName
VARCHAR(50)
NOT NULL
—
Apellido
Person
IdentificationType
VARCHAR(20)
NOT NULL
CHECK (TI, CC)
Tipo de documento
Person
IdentificationNumber
VARCHAR(20)
NOT NULL
UNIQUE
Número de documento
Person
Email
VARCHAR(50)
NOT NULL
—
Correo electrónico
Person
Phone
INT
NOT NULL
—
Teléfono de contacto
Person
ResidenceAddress
VARCHAR(50)
NOT NULL
—
Dirección de residencia
Person
DateBirth
DATE
NOT NULL
—
Fecha de nacimiento
Person
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
Family
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del grupo familiar
Family
Name
VARCHAR(50)
NOT NULL
—
Nombre del grupo
Family
Observations
VARCHAR(100)
NOT NULL
—
Observaciones del grupo
Family
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
FamilyMember
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del vínculo
FamilyMember
FamilyId
UNIQUEIDENTIFIER
NOT NULL
FK → Family
Grupo familiar
FamilyMember
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Perfil vinculado
FamilyMember
RelationshipType
VARCHAR(20)
NOT NULL
CHECK (Parent, Student)
Relación dentro del grupo
FamilyMember
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
DriverLicense
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la licencia
DriverLicense
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Conductor titular
DriverLicense
LicenseNumber
VARCHAR(20)
NOT NULL
UNIQUE
Número de licencia
DriverLicense
LicenseExpirationDate
DATE
NOT NULL
—
Fecha de vencimiento
DriverLicense
Status
VARCHAR(20)
NOT NULL
—
Vigente o no vigente
### 5.4. Esquema School — 4 tablas
** Tabla 22. Columnas del esquema School**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
School
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del colegio
School
CityId
UNIQUEIDENTIFIER
NOT NULL
FK → City
Ciudad de ubicación
School
Logo
VARBINARY(MAX)
NOT NULL
—
Logotipo institucional
School
Name
VARCHAR(30)
NOT NULL
—
Nombre del colegio
School
Address
VARCHAR(30)
NOT NULL
—
Dirección
School
Phone
INT
NOT NULL
—
Teléfono de contacto
School
Email
VARCHAR(50)
NOT NULL
—
Correo institucional
School
Website
VARCHAR(100)
Sí
—
Sitio web institucional
School
Theme
VARCHAR(20)
Sí
—
Tema visual institucional
School
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
SchoolCampus
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la sede
SchoolCampus
SchoolId
UNIQUEIDENTIFIER
NOT NULL
FK → School
Colegio propietario
SchoolCampus
Name
VARCHAR(30)
NOT NULL
—
Nombre de la sede
SchoolCampus
Address
VARCHAR(30)
NOT NULL
—
Dirección de la sede
SchoolCampus
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
Course
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del curso
Course
CampusId
UNIQUEIDENTIFIER
NOT NULL
FK → SchoolCampus
Sede que ofrece el curso
Course
Name
VARCHAR(10)
NOT NULL
—
Nombre del curso
Course
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
SchoolAdmin
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la asignación
SchoolAdmin
SchoolId
UNIQUEIDENTIFIER
NOT NULL
FK → School
Colegio administrado
SchoolAdmin
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile, UNIQUE
Administrador asignado (uno por perfil)
SchoolAdmin
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
### 5.5. Esquema Geographic — 1 tabla
** Tabla 23. Columnas del esquema Geographic**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
City
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la ciudad
City
Name
VARCHAR(40)
NOT NULL
—
Nombre de la ciudad
City
Country
VARCHAR(30)
NOT NULL
—
País de la ciudad
City
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
### 5.6. Esquema Fleet — 4 tablas
** Tabla 24. Columnas del esquema Fleet**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
Brand
Id
TINYINT identity
NOT NULL
PK
Identificador de la marca
Brand
Name
VARCHAR(30)
Sí
—
Nombre de la marca
Brand
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
Model
Id
TINYINT identity
NOT NULL
PK
Identificador del modelo
Model
BrandId
TINYINT
NOT NULL
FK → Brand
Marca propietaria
Model
Name
VARCHAR(30)
Sí
—
Nombre del modelo
Model
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
Bus
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del bus
Bus
CampuseId
UNIQUEIDENTIFIER
NOT NULL
FK → SchoolCampus
Sede a la que pertenece
Bus
SoatValidity
DATE
NOT NULL
—
Vigencia del SOAT
Bus
GpsDeviceId
UNIQUEIDENTIFIER
NOT NULL
FK → GpsDevice, UNIQUE
Dispositivo de geolocalización
Bus
Capacity
TINYINT
NOT NULL
—
Capacidad de pasajeros
Bus
Plate
VARCHAR(15)
NOT NULL
UNIQUE
Placa del vehículo
Bus
ModelId
TINYINT
NOT NULL
FK → Model
Modelo del vehículo
Bus
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
DriverAssignemts
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la asignación
DriverAssignemts
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Conductor asignado
DriverAssignemts
BusId
UNIQUEIDENTIFIER
NOT NULL
FK → Bus
Bus asignado
DriverAssignemts
AssignedFrom
DATE
NOT NULL
—
Inicio de la vigencia
DriverAssignemts
AssignedTo
DATE
Sí
—
Fin de la vigencia
### 5.7. Esquema Route — 7 tablas
** Tabla 25. Columnas del esquema Route**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
Route
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la ruta
Route
CampuseId
UNIQUEIDENTIFIER
NOT NULL
FK → SchoolCampus
Sede que opera la ruta
Route
Name
VARCHAR(30)
NOT NULL
—
Nombre de la ruta
Route
TargetSector
VARCHAR(30)
NOT NULL
—
Sector o zona de cobertura
Route
StartTime
TIME
NOT NULL
—
Hora de inicio programada
Route
EndTime
TIME
NOT NULL
—
Hora de cierre programado
Route
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
Stop
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la parada
Stop
CityId
UNIQUEIDENTIFIER
NOT NULL
FK → City
Ciudad de la parada
Stop
SchoolId
UNIQUEIDENTIFIER
NOT NULL
FK → School
Colegio de destino
Stop
Name
VARCHAR(30)
NOT NULL
—
Nombre de la parada
Stop
Address
VARCHAR(100)
NOT NULL
—
Dirección de la parada
Stop
Longitude
DECIMAL(10,7)
NOT NULL
—
Longitud de la parada
Stop
Latitude
DECIMAL(10,7)
NOT NULL
—
Latitud de la parada
Stop
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
RouteStop
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del paso
RouteStop
RouteId
UNIQUEIDENTIFIER
NOT NULL
FK → Route
Ruta
RouteStop
StopId
UNIQUEIDENTIFIER
NOT NULL
FK → Stop
Parada
RouteStop
OrderSequence
INT
NOT NULL
—
Orden dentro del recorrido
RouteStop
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
RouteBusAssignments
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la asignación
RouteBusAssignments
BusId
UNIQUEIDENTIFIER
NOT NULL
FK → Bus
Bus asignado
RouteBusAssignments
RouteId
UNIQUEIDENTIFIER
NOT NULL
FK → Route
Ruta asignada
RouteBusAssignments
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
RouteStudentAssignments
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la asignación
RouteStudentAssignments
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Estudiante asignado
RouteStudentAssignments
RouteStopId
UNIQUEIDENTIFIER
NOT NULL
FK → RouteStop
Parada de bajada
RouteStudentAssignments
Status
VARCHAR(20)
NOT NULL
—
Activa o inactiva
RouteSchedule
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del programado
RouteSchedule
RouteBusAssignmentsId
UNIQUEIDENTIFIER
NOT NULL
FK → RouteBusAssignments
Asignación que se programa
RouteSchedule
DayOfWeek
VARCHAR(10)
NOT NULL
CHECK (lunes a viernes)
Día de la semana
RouteSchedule
Direction
VARCHAR(10)
NOT NULL
—
Sentido del recorrido
RouteSchedule
StartTime
TIME
NOT NULL
—
Hora de salida
RouteSchedule
Status
VARCHAR(20)
NOT NULL
—
Activo o inactivo
RouteExecution
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la ejecución
RouteExecution
RouteId
UNIQUEIDENTIFIER
Sí
FK → Route
Ruta ejecutada
RouteExecution
BusId
UNIQUEIDENTIFIER
NOT NULL
FK → Bus
Bus utilizado
RouteExecution
DriverId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Conductor del trayecto
RouteExecution
StartDateTime
DATETIME2
NOT NULL
—
Inicio real del trayecto
RouteExecution
EndDateTime
DATETIME2
Sí
—
Fin real del trayecto
RouteExecution
Status
VARCHAR(20)
NOT NULL
—
Estado de la ejecución
### 5.8. Esquema Gps — 2 tablas
** Tabla 26. Columnas del esquema Gps**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
GpsDevice
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del dispositivo
GpsDevice
Imei
VARCHAR(20)
NOT NULL
UNIQUE
Identificador del equipo
GpsDevice
LastConnection
DATETIME2
NOT NULL
—
Última comunicación recibida
GpsDevice
GpsStatus
BIT
NOT NULL
—
Estado operativo del dispositivo
GpsLocation
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la posición
GpsLocation
GpsDeviceId
UNIQUEIDENTIFIER
NOT NULL
FK → GpsDevice
Dispositivo que reporta
GpsLocation
Latitude
DECIMAL(10,7)
NOT NULL
—
Latitud reportada
GpsLocation
Longitude
DECIMAL(10,7)
NOT NULL
—
Longitud reportada
GpsLocation
Speed
DECIMAL(8,2)
NOT NULL
—
Velocidad instantánea
GpsLocation
Course
DECIMAL(8,2)
NOT NULL
—
Rumbo del vehículo
GpsLocation
DateTime
DATETIME2
NOT NULL
—
Marca de tiempo de la posición
### 5.9. Esquema Notification — 5 tablas
** Tabla 27. Columnas del esquema Notification**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
AlertType
Id
TINYINT identity
NOT NULL
PK
Identificador del tipo de alerta
AlertType
Name
VARCHAR(30)
NOT NULL
—
Nombre del tipo
AlertType
Description
VARCHAR(100)
NOT NULL
—
Descripción del tipo
AlertType
UrgencyLevel
TINYINT
NOT NULL
—
Nivel de urgencia
Alert
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la alerta
Alert
RouteExecutionId
UNIQUEIDENTIFIER
NOT NULL
FK → RouteExecution
Ejecución que la origina
Alert
AlertTypeId
TINYINT
NOT NULL
FK → AlertType
Tipo de alerta
Alert
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Quien genera la alerta
Alert
DateTime
DATETIME2
NOT NULL
—
Momento de generación
Alert
Description
VARCHAR(100)
Sí
—
Detalle del evento
Alert
AcknowledgedAt
DATETIME2
Sí
—
Momento de acuse de recibo
AlertRecipient
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del destinatario
AlertRecipient
AlertId
UNIQUEIDENTIFIER
NOT NULL
FK → Alert
Alerta destinada
AlertRecipient
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Perfil destinatario
AlertRecipient
DateTimeRead
DATETIME2
NOT NULL
—
Momento de lectura
Boarding
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador de la bordada
Boarding
RouteExecutionId
UNIQUEIDENTIFIER
NOT NULL
FK → RouteExecution
Ejecución en curso
Boarding
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Estudiante que borda
Boarding
RouteStopId
UNIQUEIDENTIFIER
NOT NULL
FK → RouteStop
Parada del evento
Boarding
DateTime
DATETIME2
NOT NULL
—
Momento del escaneo
Boarding
BoardingType
VARCHAR(20)
NOT NULL
CHECK (ON_BOARD, OFF_BOARD)
Subida o bajada
DeviceToken
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del token
DeviceToken
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Perfil dueño del dispositivo
DeviceToken
ExpoToken
VARCHAR(300)
NOT NULL
UNIQUE
Token de notificaciones push
DeviceToken
UpdatedAt
DATETIME2
NOT NULL
—
Última actualización
### 5.10. Esquema Exceptional — 2 tablas
** Tabla 28. Columnas del esquema Exceptional**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
ExceptionalDriverUsage
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del registro
ExceptionalDriverUsage
BusId
UNIQUEIDENTIFIER
NOT NULL
FK → Bus
Bus utilizado
ExceptionalDriverUsage
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Persona que lo solicita
ExceptionalDriverUsage
DateTime
DATETIME2
NOT NULL
—
Momento del uso
ExceptionalDriverUsage
Reason
VARCHAR(200)
NOT NULL
—
Motivo del uso excepcional
ExceptionalRouteUsage
Id
UNIQUEIDENTIFIER
NOT NULL
PK
Identificador del registro
ExceptionalRouteUsage
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Persona que lo solicita
ExceptionalRouteUsage
RouteId
UNIQUEIDENTIFIER
NOT NULL
FK → Route
Ruta afectada
ExceptionalRouteUsage
StartDateTime
DATETIME
NOT NULL
—
Inicio de la ventana autorizada
ExceptionalRouteUsage
EndDateTime
DATETIME
NOT NULL
—
Fin de la ventana autorizada
ExceptionalRouteUsage
Reason
VARCHAR(200)
NOT NULL
—
Motivo del desvío
### 5.11. Esquema Settings — 2 tablas
** Tabla 29. Columnas del esquema Settings**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
PasswordPolicies
Id
TINYINT identity
NOT NULL
PK
Identificador de la política de contraseñas
PasswordPolicies
MinLongitude
TINYINT
NOT NULL
—
Longitud mínima de la política de contraseñas
PasswordPolicies
MaxLongitude
TINYINT
NOT NULL
—
Longitud máxima de la política de contraseñas
PasswordPolicies
RequiresCapitalLetters
TINYINT
NOT NULL
—
Exige mayúsculas en la política de contraseñas
PasswordPolicies
RequiresNumbers
TINYINT
NOT NULL
—
Exige números en la política de contraseñas
PasswordPolicies
RequiresSymbols
TINYINT
NOT NULL
—
Exige símbolos en la política de contraseñas
PasswordPolicies
ExpirationDays
TINYINT
NOT NULL
—
Días de expiración de la política de contraseñas
SecuritySettings
Id
TINYINT identity
NOT NULL
PK
Identificador del parámetro
SecuritySettings
NameSettings
VARCHAR(100)
NOT NULL
UNIQUE
Nombre del parámetro de seguridad
SecuritySettings
ValueSettings
VARCHAR(100)
NOT NULL
—
Valor del parámetro de seguridad
SecuritySettings
Description
VARCHAR(150)
NOT NULL
—
Descripción del parámetro de seguridad
### 5.12. Esquema Audit — 2 tablas
** Tabla 30. Columnas del esquema Audit**
Tabla
Columna
Tipo
Nulidad
PK/FK
Descripción
Audit
Id
BIGINT identity
NOT NULL
PK
Identificador del evento
Audit
ProfileId
UNIQUEIDENTIFIER
NOT NULL
FK → Profile
Actor que ejecuta la acción
Audit
Action
VARCHAR(255)
NOT NULL
—
Acción realizada
Audit
DateTime
DATETIME2
NOT NULL
—
Momento de la acción
Audit
Description
VARCHAR(500)
NOT NULL
—
Detalle de la acción
Audit
IPAddress
VARCHAR(50)
NOT NULL
—
Origen de la acción
Audit
Application
VARCHAR(255)
NOT NULL
—
Aplicación desde la que se ejecuta
LogErrors
Id
BIGINT identity
NOT NULL
PK
Identificador del error
LogErrors
TypeError
VARCHAR(100)
NOT NULL
—
Clasificación del error
LogErrors
Description
VARCHAR(150)
NOT NULL
—
Descripción del error
LogErrors
IPAddress
VARCHAR(50)
NOT NULL
—
Origen del error
LogErrors
OccurredAt
DATETIME2
NOT NULL
—
Momento en que ocurrió
### 5.13. Enums y datos semilla
Los conjuntos de valores cerrados se protegen con restricciones en línea:
** Tabla 31. Conjuntos de valores admitidos**
Conjunto
Valores admitidos
Tabla
Estado de ciclo de vida
ACTIVE, INACTIVE
Casi todas las entidades de negocio
Rol
SUPER_ADMIN, ADMIN, DRIVER, PARENT, STUDENT
Role
Tipo de identificación
TI, CC
Person
Relación familiar
Parent, Student
FamilyMember
Tipo de bordada
ON_BOARD, OFF_BOARD
Boarding
Día de la semana
Monday … Friday
RouteSchedule
Datos semilla con los que arranca el sistema:
** Tabla 32. Datos semilla y tablas afectadas**
Contenido
Reglas semilla
Tablas afectadas
Roles y permisos
Cinco roles canónicos; 20 permisos de dominio (recurso.acción) y sus asignaciones por rol
Role, Permissions, PermissionsRole
Identidades de prueba
Personas y perfiles de prueba para desarrollo y demostración
Person, Profile, SessionProfile
Ubicación
20 ciudades con país
City
Instituciones
3 colegios, 15 sedes y 90 cursos
School, SchoolCampus, Course
Flota
20 marcas, 20 modelos, 20 buses y 20 asignaciones de conductor
Brand, Model, Bus, DriverAssignemts
Rutas
20 rutas, 20 paradas, 60 pasos de ruta, 20 asignaciones de bus, 20 asignaciones de estudiantes, 40 programaciones y 20 ejecuciones
Route, Stop, RouteStop, RouteBusAssignments, RouteStudentAssignments, RouteSchedule, RouteExecution
Geolocalización
20 dispositivos y 20 posiciones de referencia
GpsDevice, GpsLocation
Notificaciones
20 tipos de alerta, 20 alertas, 20 destinatarios y 20 bordadas
AlertType, Alert, AlertRecipient, Boarding
Familias y licencias
20 familias, 40 miembros y 20 licencias de conducir
Family, FamilyMember, DriverLicense
Usos excepcionales
20 registros de uso de bus y 20 registros de desvío de ruta
ExceptionalDriverUsage, ExceptionalRouteUsage
### 5.14. Índices y restricciones
** Tabla 33. Índices y restricciones del modelo**
Mecanismo
Aplicación
Propósito
IDX_Role_Name
Búsqueda por nombre de rol
Resolver el rol por nombre en autorización
IDX_Profile_RoleId_PersonId
Perfil por rol y persona
Consultas de listado por rol
IDX_Person_Email
Persona por correo
Búsqueda y evolución de credenciales
IDX_Person_Name, IDX_Person_LastName
Persona por nombre y apellido
Búsqueda en listados administrativos
IDX_Person_IdentificationNumber
Persona por documento
Verificación de duplicados
IDX_Person_Phone
Persona por teléfono
Búsqueda y contacto
Unicidad declarada
Person.IdentificationNumber, Bus.Plate, GpsDevice.Imei, DriverLicense.LicenseNumber, DeviceToken.ExpoToken, Role.Name, SecuritySettings.NameSettings
Evita registros duplicados a nivel de base de datos
Unicidad de negocio
SchoolAdmin.ProfileId (un administrador administra un solo colegio)
Regla de negocio garantizada en el motor
Restricciones de dominio
Estados, tipos de documento, relación familiar, tipo de bordada y día de la semana
Impide valores fuera de catálogo
Integridad referencial
45 claves foráneas declaradas entre esquemas
Evita huérfanos entre contextos de dominio
### 5.15. Reglas de negocio y retención
** Tabla 34. Reglas de negocio y retención de datos**
Regla
Expresión en el modelo
Baja lógica
La inhabilitación de una entidad se realiza con Status = INACTIVE; los listados operativos filtran por ACTIVE.
Un perfil por sede
Profile.CampuseId enlaza al perfil con una sede concreta; el colegio se deriva de esa sede.
Un colegio por administrador
La restricción única sobre SchoolAdmin.ProfileId impide que un administrador atienda varias instituciones.
Vigencia de asignaciones
DriverAssignemts.AssignedTo admite NULL, lo que indica vigencia abierta; el conductor activo es el que contiene la fecha actual.
Orden de paradas
RouteStop.OrderSequence es el orden efectivo del recorrido; no puede haber dos paradas con el mismo orden en una ruta.
Ejecución contenedora
Las bordadas y las alertas siempre referencian una RouteExecution; no pueden existir sin un trayecto real.
Parada de bajada
RouteStudentAssignments.RouteStopId fija la parada de descargo del estudiante dentro de la ruta.
Ruta de la ejecución
RouteExecution.RouteId se permite nulo para no bloquear el registro de un trayecto cuando la ruta se reordena en caliente; se completa al cierre del trayecto.
Fin de trayecto
RouteExecution.EndDateTime admite NULL mientras el trayecto está en curso y se escribe al finalizar.
Dispositivo exclusivo
Un bus tiene un solo dispositivo de geolocalización y un dispositivo no se comparte entre buses.
Expiración de licencia
DriverLicense.LicenseExpirationDate habilita el bloqueo de asignación de conducción cuando la licencia está vencida.
Registro de uso excepcional
Todo uso de bus fuera del plan y todo desvío de ruta exigen motivo y ventana de tiempo.
Retención de auditoría
Los eventos de Audit y los registros de LogErrors no se eliminan por operación de negocio; su ciclo de vida se define por política de respaldo y archivamiento de la base.
Retención operativa
Las posiciones de GpsLocation y las sesiones de SessionProfile son datos de volumen creciente y se depuran por antigüedad conforme a la política de retención de la plataforma.
Copias de seguridad
La base se respalda de forma programada con restauración verificada; el esquema se reconstruye con migraciones si la restauración no es posible.

### 6. Contratos de interfaz

Los contratos se agrupan en tres familias: recursos REST versionados bajo el prefijo /api/v1 y expuestos a través del paso de API, eventos de dominio publicados en el broker, y llamadas RPC internas de baja latencia. La autorización se evalúa por rol y permiso tanto en el paso de API como en cada servicio, de modo que la columna de rol de la tabla siguiente representa la matriz de acceso efectiva.
### 6.1. Recursos REST
** Tabla 35. Recursos REST y roles autorizados**
Verbo
Ruta
Rol
Descripción
POST
/api/v1/iam/auth/login
Público
Autentica y emite token de acceso y de refresco
POST
/api/v1/iam/auth/refresh
Público
Rota el token de refresco y renueva la sesión
POST
/api/v1/iam/auth/logout
Autenticado
Revoca la sesión vigente
POST
/api/v1/iam/auth/forgot-password
Público
Inicia la recuperación conforme a la política de contraseñas
POST
/api/v1/iam/auth/reset-password
Público
Restablece la contraseña verificando el código de longitud seis
POST
/api/v1/iam/auth/change-password
Autenticado
Cambia la contraseña vigente aplicando la política de contraseñas
GET
/api/v1/iam/jwks
Público
Publica las llaves para verificar la firma de los tokens
GET, POST
/api/v1/iam/roles
SUPER_ADMIN
Consulta y creación de roles
GET
/api/v1/iam/permissions
SUPER_ADMIN
Consulta del catálogo de permisos
GET, PATCH
/api/v1/iam/profiles[/{id}/status]
ADMIN
Listado de perfiles de la institución y activación o inactivación de un perfil
POST, DELETE
/api/v1/users/persons[/{id}]
ADMIN
Alta de personas (estudiante, acudiente, conductor, administrador) y baja lógica
POST
/api/v1/users/families
ADMIN
Creación de un grupo familiar
POST
/api/v1/users/families/{id}/members
ADMIN
Vinculación de un miembro al grupo familiar
POST
/api/v1/users/driver-licenses
ADMIN
Registro de licencia de conducir con vigencia
POST, PUT
/api/v1/schools/schools[/{id}]
SUPER_ADMIN
Alta y actualización de la institución educativa
POST
/api/v1/schools/campuses
SUPER_ADMIN
Alta de sede
POST
/api/v1/schools/courses
ADMIN
Alta de curso en una sede
POST
/api/v1/schools/cities
SUPER_ADMIN
Alta de ciudad de referencia
GET
/api/v1/buses/brands, /models
ADMIN
Catálogos de marcas y modelos
POST, PUT
/api/v1/buses/buses[/{id}]
ADMIN
Alta y actualización de bus con placa, capacidad y dispositivo
DELETE
/api/v1/buses/buses/{id}/driver
ADMIN
Retiro del conductor asignado
POST, DELETE
/api/v1/routes/routes[/{id}]
ADMIN
Alta de ruta con sector y horario, y baja de la ruta
POST
/api/v1/routes/routes/{id}/stops
ADMIN
Incorporación de parada en orden dentro de la ruta
POST
/api/v1/routes/routes/{id}/students
ADMIN
Asignación de estudiante a una parada
POST
/api/v1/routes/routes/{id}/bus
ADMIN
Asignación de bus a la ruta
GET
/api/v1/routes/routes/{id}/schedules
ADMIN
Programación semanal de la ruta
POST
/api/v1/routes/executions
ADMIN, DRIVER
Apertura de la ejecución real de un trayecto
GET
/api/v1/notifications/alert-types
ADMIN
Catálogo de tipos de alerta y nivel de urgencia
POST, PATCH
/api/v1/notifications/alerts[/{id}/status]
ADMIN, DRIVER
Generación de una alerta sobre una ejecución y acuse de recibo
GET
/api/v1/notifications/boarding
ADMIN, DRIVER
Consulta de eventos de bordada
GET
/api/v1/notifications/recipients
ADMIN
Consulta de destinatarios y lectura de alertas
POST, GET, PATCH
/api/v1/gps/devices[/{id}[/status]]
ADMIN
Registro, consulta de estado y última conexión, y activación o suspensión del dispositivo
GET
/api/v1/gps/locations
ADMIN, DRIVER, PARENT
Consulta de posiciones registradas
POST
/api/v1/exceptional/driver-usages, /route-usages
ADMIN
Registro de uso excepcional de bus o de desvío de ruta con motivo y ventana
GET, PUT
/api/v1/config/password-policies, /settings
SUPER_ADMIN
Consulta y ajuste de la política de contraseñas y de los parámetros de seguridad
POST
/api/v1/audit/entries
Interno
Ingesta de eventos de auditoría desde los servicios
GET
/api/v1/audit/errors
SUPER_ADMIN
Consulta del registro de errores
GET
/health (paso de API)
Público
Verificación de salud del punto de entrada
Todas las rutas de negocio se protegen con token Bearer salvo las declaradas como públicas; las respuestas de error siguen una estructura común con marca de tiempo, mensaje y detalle del camino, y las listas se sirven con paginación por página y tamaño.
### 6.2. Eventos de dominio (Kafka)
El broker opera en modo KRaft sin componente de coordinación externo y publica seis tópicos verificados en el entorno de ejecución. La llave de partición es el identificador de la entidad, para conservar el orden por agregado.
** Tabla 36. Eventos de dominio y sus consumidores**
Evento
Productor
Consumidor
Payload resumen
student.created
Servicio de gestión de usuarios
Servicio de identificación y acceso
Identificador del evento, persona, correo, número de documento y sede
driver.created
Servicio de gestión de usuarios
Servicio de identificación y acceso
Mismo contenido que el alta de estudiante, con el rol de conductor
admin.created
Servicio de gestión de usuarios
Servicio de identificación y acceso
Mismo contenido con el colegio a administrar en lugar de sede
parent.created
Servicio de gestión de usuarios
Servicio de identificación y acceso
Identificador del evento, persona, correo, documento y sede
profile.created
Servicio de identificación y acceso
Sin consumidor registrado
Confirmación de la creación del perfil de acceso para una persona
student.scanned
Servicio de notificaciones
Servicio de notificaciones
Bordada, alerta, perfil del estudiante, ejecución de ruta, tipo de bordada y marca de tiempo
Contrato del payload de alta de personas (EventId, PersonId, Email, IdentificationNumber, CampusId opcional y SchoolId solo en el alta de administradores) y del escaneo (BoardingId, AlertId, StudentProfileId, RouteExecutionId, BoardingType, Timestamp): los campos son aditivos, los consumidores toleran campos desconocidos y el productor es dueño del esquema.
Reglas de consumo: un consumidor que falle vuelve a leer el mensaje según la política de reintentos y la contraparte no se bloquea; la creación del perfil de acceso y la generación de la notificación son operaciones idempotentes respecto al identificador del evento.
### 6.3. Flujo gRPC
Las llamadas internas de baja latencia usan RPC sobre canal persistente en el puerto 5001, en texto plano (h2c) dentro de la red privada de la plataforma; el transporte cifrado hacia el exterior lo termina el paso de API.
+--------------------------------+          +----------------------------------+
| Servicio de identificación     |          | Servicio de gestión escolar      |
| y acceso (cliente)             |   RPC    | (servidor, puerto 5001)          |
|  tras consumir admin.created:  +--------->|  SchoolManagementService         |
|  LinkAdminSchool, GetCampus,   |  h2c     |   GetSchool, GetCampus,          |
|  GetCampusByName               |          |   GetCampusByName, GetAdminSchool|
+--------------------------------+          |   LinkAdminSchool                |
| Servicio de gestión de usuarios|          +----------------------------------+
| (cliente) consulta sede y      |
| colegio al registrar personas  |
+--------------------------------+
+--------------------------------+          +----------------------------------+
| Servicio de rutas (servidor)   |          | Servicio de gestión de usuarios  |
|  RouteService                  |          |  FamilyService                   |
|   CreateRoute, GetRoute,       |          |   RegisterFamilyGroup            |
|   ListRoutes, UpdateRoute,     |          |    (nombre del grupo,            |
|   DeleteRoute, GetCurrentRoute,|          |     observaciones y miembros     |
|   GetStudentRoute, StartTrip,  |          |     con tipo de relación)        |
|   EndTrip, AssignStudentToRoute|          +----------------------------------+
|   AssignBusToRoute,            |
|   AttachStopToRoute            |
+--------------------------------+
** Figura 19. Flujo de llamadas gRPC entre servicios**
Comportamiento del cliente: canal compartido con ciclo de vida gestionado por el contenedor, tiempo límite por llamada, reintento acotado ante fallo transitorio y transformación del error remoto a un error de dominio que la capa de API traduce a una respuesta de problema. Los mensajes de solicitud y respuesta transportan identificadores como cadenas y estados como valores booleanos o de catálogo, evitando estructuras anidadas profundas.

### 7. Decisiones y trade-offs de diseño

** Tabla 37. Decisiones de diseño y alternativas descartadas**
Decisión
Alternativa descartada
Criterio y contrapartida
Una sola base relacional con esquema por servicio
Base de datos independiente por servicio
Se gana integridad referencial transversal y operación de respaldo única; se asume la disciplina de no leer datos ajenos y el acoplamiento en las migraciones.
Paso de API único con enrutamiento
Exponer cada servicio directamente
Se ganan política de seguridad, límite de tasa y CORS comunes; se acepta un punto único que debe escalar y estar disponible.
Orientación a eventos para el alta de usuarios y los escaneos
Encadenamiento síncrono de llamadas
Se desacoplan productor y consumidor en el tiempo y se toleran caídas puntuales; se asume consistencia eventual entre persona y perfil de acceso.
RPC interna para consultas y trayectos de baja latencia
Todo por REST
Se reducen tiempos de respuesta en el camino caliente; se acepta un segundo contrato que debe versionarse con la misma disciplina.
Token de acceso firmado con RS256 y llave privada solo en el servicio de identificación
Llave simétrica compartida
Cualquier servicio puede verificar sin custodiar la llave; se añade la gestión de renovación y publicación de llaves públicas.
Arquitectura hexagonal dentro de cada servicio
Modelo MVC de capas únicas
Se aísla el dominio de marcos y bases de datos a cambio de más tipos y más archivos por caso de uso.
Baja lógica con ACTIVE/INACTIVE
Borrado físico
Se conserva trazabilidad y reversibilidad; se asume que los listados deben filtrar y que el volumen crece.
Identificadores globales únicos como llave primaria
Identidad secuencial compartida
Permite crear registros sin consultar al centro de secuencia y fusionar datos entre servicios; se pierde la localidad de las claves en las búsquedas por rango.
Aplicación web para administración y aplicación móvil para operación de campo
Una sola aplicación web responsiva
Cada superficie se optimiza para su contexto (tablas y formularios frente a acciones grandes y registro sin conexión); se mantiene un sistema de diseño común para no duplicar lenguaje visual.
Contrato de datos por transferencia de datos separado del modelo interno
Exponer entidades de dominio directamente
Se evita que los cambios internos rompan a los clientes; se paga con mapeos adicionales en cada operación.
Design tokens con ocho temas cerrados
Personalización libre de colores
Se garantiza contraste y coherencia visual; se limita la expresividad de la personalización institucional.
Programación por catálogos parametrizados
Condiciones fijas en código
Nuevos tipos de alerta o parámetros de seguridad se habilitan como datos; se requiere cuidado para no convertir el catálogo en una fuente de reglas opacas.
Cierre de la sección: cada decisión queda asociada a su contrapartida, de modo que el costo de mantenerla sea explícito para quien evolucione la plataforma; los trade-offs relacionados con la comunicación y con la estructura de datos son los que mayor efecto tienen sobre el rendimiento y sobre el costo de cambio, por lo que se revisan primero cuando cambian los requisitos.
