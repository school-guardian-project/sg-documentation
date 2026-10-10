
**Software Implementation Procedure**

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

# 1 Purpose and Scope

## 1.1 Purpose

Este procedimiento establece la secuencia reproducible mediante la cual la plataforma **Guardian Escolar** se implanta en un entorno de destino, se verifica su funcionamiento y se entrega a la institución educativa para su operación. La plataforma comprende una aplicación web de administración, aplicaciones móviles para conductor y acudiente, una puerta de enlace única, nueve microservicios de dominio, un broker de eventos, un canal de comunicación de baja latencia y una base de datos relacional única organizada en esquemas.

Los resultados verificables que persigue el procedimiento son cuatro:

- Infraestructura configurada de forma idéntica en cada entorno, sin diferencias no documentadas.
- Base de datos completa —**11 esquemas** y **38 tablas**— mediante migraciones versionadas y verificables.
- Cada componente expuesto únicamente en los puertos autorizados y respondiendo a sus comprobaciones de salud.
- Evidencia objetiva de aceptación, con veredicto **PASS/FAIL** por punto, antes de declarar la implantación concluida.

## 1.2 Alcance

Incluye: preparación de prerrequisitos; infraestructura base; migraciones; despliegue de los nueve servicios de dominio; publicación de las aplicaciones cliente; configuración inicial de políticas, catálogos y usuarios semilla; verificación posdespliegue; versionado; reversión; canal de integración y publicación continua; monitoreo y soporte; matriz de responsabilidades y cronograma; gestión de cambios y aceptación.

Excluye: el desarrollo de funcionalidad y las pruebas de nivel de código, ejecutados en las fases previas del ciclo de vida; la adquisición de licencias de terceros; y el aprovisionamiento de terminales móviles y dispositivos de geolocalización, responsabilidad de la institución receptora.

## 1.3 Audience and Responsibilities

**Tabla 2**. Perfiles de usuario del procedimiento

| Perfil                          | Uso del procedimiento                                       |
| ------------------------------- | ----------------------------------------------------------- |
| Líder técnico                   | Aprueba etapas y publicaciones; decide sobre reversión.     |
| Ingeniero de plataforma         | Ejecuta infraestructura, servicios, borde y clientes.       |
| Responsable de base de datos    | Ejecuta y verifica migraciones, respaldos y recuperaciones. |
| Responsable de calidad          | Ejecuta la verificación posdespliegue y emite el veredicto. |
| Responsable de operaciones      | Recibe el sistema; opera monitoreo, alertas e incidentes.   |
| Cliente (institución educativa) | Formula observaciones y firma la aceptación.                |

## 1.4 Usage Conventions

- Los valores entre marcadores angulares, por ejemplo &lt;SA_PASSWORD&gt; o &lt;ADMIN_EMAIL&gt;, son **sustitutos de secretos**: nunca se escriben valores reales en este documento ni en los registros de implantación.
- Toda orden técnica se reproduce en un bloque bash; los resultados esperados se describen en prosa o en tabla.
- Un paso cierre con una comprobación; si esta no se cumple, el paso falla y no se avanza.
- Los plazos del cronograma son **estimaciones de planificación**; los compromisos contractuales se fijan en el acta de aceptación.
- «Entorno» designa al conjunto de componentes desplegados con la misma finalidad; «componente», a cualquier unidad desplegable.

# 2 Prerequisites

## 2.1 Resources de hardware

El dimensionamiento se acuerda con la institución según el volumen esperado de estudiantes, rutas y dispositivos. El host de implantación debe cumplir:

**Tabla 3**. Requisitos de hardware del host de implantación

| Recurso              | Requisito funcional                                                                                                              | Criterio de comprobación                                           |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Procesador y memoria | Ejecutar simultáneamente motor de datos, broker, puerta de enlace y servicios sin saturación sostenida ni intercambio de memoria | Uso estable bajo el umbral de alerta durante la ventana de pruebas |
| Almacenamiento       | Volumen persistente para base de datos y broker, con espacio para respaldos completos y de registro                              | Espacio libre por encima del umbral de alerta                      |
| Sistema operativo    | Plataforma con ejecución de contenedores y cliente de orquestación operativo                                                     | Versión y estado del servicio verificados                          |
| Terminales móviles   | Cámara para escaneo QR y notificaciones push, uno por conductor en servicio                                                      | Verification funcional en el paso 4.6                              |

## 2.2 Base Software

**Tabla 4**. Base Software y versiones mínimas

| Componente                     | Versión mínima  | Finalidad                                | Comprobación                            |
| ------------------------------ | --------------- | ---------------------------------------- | --------------------------------------- |
| Motor de contenedores          | 24.0            | Ejecución de componentes                 | docker --version                        |
| Orquestador de contenedores    | 2.20            | Definición e inversión de conjuntos      | docker compose version                  |
| Control de versiones           | 2.40            | Obtención de la línea base del código    | git --version                           |
| SQL Server                     | 2022            | Persistencia relacional única            | Contenedor saludable                    |
| Apache Kafka                   | 4.1.0 (KRaft)   | Eventos de dominio                       | Consumo de tópicos operativo            |
| Kong                           | 3.9             | Enrutamiento, límite de tasa y CORS      | Respuesta en el puerto público          |
| Liquibase                      | 4.29            | Cambios de esquema versionados           | Cero cambiosets pendientes              |
| Java con Maven                 | 21              | Identificación y acceso; geolocalización | Compilación sin errores                 |
| .NET                           | 10              | Servicios de dominio restantes           | Compilación sin errores                 |
| Node.js con gestor de paquetes | LTS             | Construcción de aplicaciones cliente     | Compilación sin errores                 |
| Angular con Angular Material   | 21 y Material 3 | Aplicación web                           | Compilación de producción satisfactoria |
| React con Expo                 | Versión activa  | Aplicaciones móviles                     | Binario generado satisfactoriamente     |

## 2.3 Network and Ports

**Tabla 5**. Puertos autorizados por componente

| Puerto      | Componente                              | Alcance permitido                               |
| ----------- | --------------------------------------- | ----------------------------------------------- |
| 8000        | API Gateway (proxy)                | Público: único punto de entrada                 |
| 8443        | API Gateway (proxy TLS)            | Público: terminación de TLS                     |
| 8001 y 8002 | API e interfaz administrativa del borde | Solo red interna                                |
| 8080        | Servicios de dominio                    | Solo red de contenedores, resueltos por nombre  |
| 5001        | Canal gRPC de geolocalización           | Solo red interna                                |
| 1433        | SQL Server                              | Solo red interna; no expuesto a Internet        |
| 9092        | Apache Kafka                            | Solo red interna; no expuesto a Internet        |
| 6379        | Redis                                   | Solo red interna: caché y lista negra de tokens |
| 8090        | Herramienta de consulta de datos        | Solo desarrollo; prohibida en producción        |
| 4200        | Servidor de desarrollo web              | Solo desarrollo: origen de referencia de CORS   |

Antes de iniciar se confirma que ningún puerto (en particular 8000, 1433 y 9092) esté ocupado por otro proceso del host, con lsof -i :&lt;puerto&gt;.

## 2.4 Permissions, Secrets, and Access

**Tabla 6**. Permissions, Secrets, and Access requeridos

| Elemento                                                          | Requisito                                                   | Manejo                                            |
| ----------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------- |
| Acceso al host                                                    | Privilegios de administración del sistema de contenedores   | Cuenta personal con trazabilidad                  |
| Clave del motor (&lt;SA_PASSWORD&gt;) y usuario (&lt;SA_USER&gt;) | Generados antes del primer arranque y distintos por entorno | Almacén de secretos; nunca en el código           |
| Llave privada de firma (&lt;JWT_PRIVATE_KEY&gt;)                  | Exclusiva del servicio de identificación y acceso           | RS256: acceso de 15 minutos y refresco de 30 días |
| Llave pública de verificación                                     | Distribuida a los demás servicios y al borde                | Configuración declarada                           |
| Credenciales de mensajería móvil (&lt;PUSH_CREDENTIALS&gt;)       | Provisionadas por el proveedor                              | Almacén de secretos del entorno                   |
| Origen web permitido (&lt;WEB_ORIGIN&gt;)                         | Dominio público de la aplicación web                        | Configuración declarada del borde                 |
| Registro de imágenes                                              | Permiso de publicación y descarga                           | Credenciales rotadas por entorno                  |

## 2.5 Pre-Implementation Validation

docker info

docker compose version

df -h

docker network ls

docker images

La implantación no comienza si alguna comprobación falla o si la línea base de código no corresponde a la versión aprobada para publicación.

# 3 Environment Architecture

## 3.1 Supported Environments

**Tabla 7**. Supported Environments y su finalidad

| Entorno                 | Finalidad                                   | Actualización                                   | Acceso                                     | Datos                         |
| ----------------------- | ------------------------------------------- | ----------------------------------------------- | ------------------------------------------ | ----------------------------- |
| Desarrollo              | Construcción individual                     | Manual                                          | Equipo técnico                             | Sintéticos de demostración    |
| Integración             | Validación continua de cada cambio aprobado | Automática al integrar en la rama de desarrollo | Equipo técnico                             | Sintéticos                    |
| Pruebas (preproducción) | Verification, demostración y aceptación     | Automática al integrar en la rama principal     | Equipo técnico y responsable de producto   | Anonimizados desde producción |
| Producción              | Operación con usuarios reales               | Manual con aprobación formal                    | Responsable técnico y equipo de plataforma | Reales                        |

## 3.2 Deployment Topology

Acudiente (móvil) Conductor (móvil) Administrador (web)

\\ | /

+-------+-------+--------+--------+

| TLS/HTTP :8000 |

v |

+------------------+ |

| API Gateway |--- CORS, límite de tasa, JWT

| (Kong OSS) |

+--------+---------+

| red de contenedores, cada servicio :8080

+---------+-------+------+----------+----------+-----------+

v v v v v v

+-----+ +--------+ +---------+ +---------+ +----------+ +----------+

| IAM | |Usuarios| | Escolar | | Flota | | Rutas | |Notificac.|

+--+--+ +---+----+ +----+-----+ +----+----+ +----+-----+ +----+-----+

| | | | | |

+-------+------------+-----+------+-----------+-------------+

| canal gRPC :5001

v

+----------------+ +---------------+

| Excepcionales | | Auditoría y |

+----------------+ | Configuración |

+---------------+

|

v

+------------------+ +------------------+ +------------------+

| SQL Server :1433 | | Kafka :9092 | | Redis :6379 |

| 11 esquemas | | modo KRaft | | caché efímera |

| 38 tablas | | 6 topics | +------------------+

+------------------+ +------------------+

**Figura 1**. Deployment Topology de la plataforma

## 3.3 Component Inventory and Ports

**Tabla 8**. Component Inventory and Ports

| Componente                    | Naturaleza      | Puerto             | Persistencia                           |
| ----------------------------- | --------------- | ------------------ | -------------------------------------- |
| API Gateway (Kong OSS)   | Infraestructura | 8000 / 8443        | Configuración declarada versionada     |
| Identificación y acceso (IAM) | Dominio         | 8080               | Esquema Iam                            |
| Gestión de usuarios           | Dominio         | 8080               | Esquema UserManagement                 |
| Gestión escolar               | Dominio         | 8080 / 5001 (gRPC) | Esquemas School y Geographic           |
| Flota                         | Dominio         | 8080               | Esquema Fleet                          |
| Rutas                         | Dominio         | 8080               | Esquema Route                          |
| Notificaciones                | Dominio         | 8080               | Esquema Notification                   |
| Usos excepcionales            | Dominio         | 8080               | Esquema Exceptional                    |
| Auditoría                     | Dominio         | 8080               | Esquema Audit                          |
| Configuración                 | Dominio         | 8080               | Esquema Settings                       |
| SQL Server 2022               | Infraestructura | 1433               | Volumen persistente                    |
| Apache Kafka 4.1.0 (KRaft)    | Infraestructura | 9092               | Volumen persistente                    |
| Redis                         | Infraestructura | 6379               | Efímero, sin requisito de restauración |

## 3.4 Configuration Policies per Environment

**Tabla 9**. Configuration Policies per Environment

| Parámetro            | Desarrollo               | Integración                        | Pruebas                        | Producción                        |
| -------------------- | ------------------------ | ---------------------------------- | ------------------------------ | --------------------------------- |
| Nivel de bitácora    | debug                    | info                               | info                           | warn                              |
| Origen web permitido | <http://localhost:4200>  | https://&lt;integracion-domain&gt; | https://&lt;pruebas-domain&gt; | https://&lt;produccion-domain&gt; |
| Fuente de secretos   | Archivo de entorno local | Almacén de secretos                | Almacén de secretos            | Almacén de secretos               |
| Datos                | Sintéticos               | Sintéticos                         | Anonimizados                   | Reales                            |
| Publicación          | Manual                   | Automática                         | Automática                     | Manual con doble aprobación       |

La convención de nombres de variables es APP_\[SERVICIO\]\_\[VARIABLE\]; cada servicio conserva además sus claves de cadena de conexión y de dirección del broker.

# 4 Step-by-Step Implementation Procedure

## 4.1 Step 1: Base Infrastructure

### 4.1.1 Container Network and Secrets

1. Se crea la red compartida que interconecta todos los conjuntos de servicios.
2. Se prepara el archivo de entorno de cada conjunto a partir de su plantilla de ejemplo.
3. Se sustituyen todos los marcadores angulares por valores del almacén de secretos del entorno.
4. Se confirma que ningún archivo de entorno con valores reales quede incorporado a la línea base de código.

docker network create &lt;nombre-de-la-red&gt;

cp .env.example .env

openssl rand -base64 32

docker network ls

Cierre: la red figura en el inventario y el archivo de entorno contiene únicamente valores admitidos por la política de secretos.

### 4.1.2 Database Engine

1. Se levanta SQL Server 2022 con el puerto 1433 asignado solo a la red interna y con volumen persistente.
2. Se espera a que la comprobación de salud integrada —SELECT 1 cada 5 segundos, hasta 20 reintentos y 20 segundos de espera inicial— reporte estado saludable.
3. Se ejecuta el contenedor de inicialización, que crea la base school-guardian si no existe y termina con código 0.

docker compose up -d

docker compose ps

docker inspect --format '{{json .State.Health.Status}}' &lt;nombre-del-contenedor-sql&gt;

### 4.1.3 Event Broker

Se levanta Apache Kafka 4.1.0 en modo KRaft, con un único nodo que cumple las funciones de bróker y de controlador y quórum de controladores sobre el puerto 9093. El contenedor de inicialización espera a que el bróker responda la enumeración de tópicos y crea los pendientes con una partición y factor de replicación 1.

**Tabla 10**. Tópicos declarados en el broker de eventos

| Tópico          | Particiones | Factor de replicación | Uso                                   |
| --------------- | ----------- | --------------------- | ------------------------------------- |
| student.created | 1           | 1                     | Alta de estudiante hacia identidad    |
| driver.created  | 1           | 1                     | Alta de conductor hacia identidad     |
| admin.created   | 1           | 1                     | Alta de administrador hacia identidad |
| parent.created  | 1           | 1                     | Alta de acudiente hacia identidad     |
| profile.created | 1           | 1                     | Perfil creado en identidad            |
| student.scanned | 1           | 1                     | Escaneo QR y registro de bordada      |

docker exec &lt;nombre-del-contenedor-kafka&gt; /opt/kafka/bin/kafka-topics.sh \\

\--bootstrap-server kafka:9092 --list

Cierre: los seis tópicos figuran en el listado y el contenedor de inicialización terminó con código 0.

### 4.1.4 Ephemeral Store

Se levanta Redis en el puerto 6379 para caché de datos de referencia, control de tasa y lista negra de tokens. Por ser efímero no se define objetivo de recuperación: ante su pérdida la instancia se recrea y los clientes reconectan.

### 4.1.5 API Gateway

Se levanta Kong con configuración declarada y sin base de datos propia, montando el manifiesto en modo solo lectura; se verifican los puertos de proxy (8000 y 8443) y los de administración (8001 y 8002), estos últimos restringidos a la red interna.

curl -fsS <http://localhost:8000/health>

Cierre: respuesta con estado ok y marca de tiempo. En este punto la puerta de enlace aún no enruta tráfico de negocio: las rutas se declaran en el paso 4.4.

## 4.2 Step 2: Database and Migrations

### 4.2.1 Change Structure

Cada esquema posee su propio árbol de cambios gestionado por Liquibase, con las etapas en este orden: creación del esquema, tipos personalizados, tablas, restricciones y claves foráneas entre esquemas, vistas, funciones, procedimientos, disparadores e índices; después carga de datos inicial, permisos del motor, bloques transaccionales y etiquetas de publicación; y, por último, un árbol reservado de instrucciones de reversión.

El conjunto completo asciende a **11 esquemas**, **38 tablas** y **127 cambiosets**, cada uno de los cuales corresponde a un guion de definición de datos.

### 4.2.2 Application Order

El orden se deriva de las claves foráneas que cruzan esquemas; la orquestación encadena los contenedores de migración mediante dependencias de finalización exitosa:

**Tabla 11**. Application Order de las migraciones

| Orden | Esquema        | Cambiosets | Depende de                     |
| ----- | -------------- | ---------- | ------------------------------ |
| 1     | Geographic     | 3          | Inicialización del motor       |
| 2     | Gps            | 6          | Inicialización del motor       |
| 3     | Settings       | 3          | Inicialización del motor       |
| 4     | School         | 12         | Geographic                     |
| 5     | Iam            | 24         | School                         |
| 6     | Fleet          | 12         | Gps, School, Iam               |
| 7     | Route          | 25         | Geographic, School, Fleet, Iam |
| 8     | UserManagement | 18         | Iam                            |
| 9     | Notification   | 13         | Iam, Route                     |
| 10    | Audit          | 4          | Iam                            |
| 11    | Exceptional    | 7          | Iam, Fleet, Route              |

Existen relaciones recíprocas entre los esquemas de identidad, escolar y gestión de usuarios. Cuando un cambioset exige un objeto aún inexistente, el motor lo reporta como no aplicado y lo deja pendiente; una segunda pasada sobre el mismo conjunto lo aplica una vez creados los esquemas implicados. Por eso el procedimiento contempla expresamente la reejecución antes de declarar la etapa terminada.

### 4.2.3 Ejecución

docker compose up -d sqlserver sqlserver-init

docker compose up -d

docker compose up -d

docker compose ps --filter "status=exited"

Cada migración se ejecuta con la ruta de búsqueda de los changelogs, el changelog maestro del esquema y la orden update. La ejecución es idempotente: solo se aplican los cambiosets pendientes.

### 4.2.4 Verification

docker exec -it &lt;nombre-del-contenedor-sql&gt; /opt/mssql-tools18/bin/sqlcmd \\

\-S localhost,1433 -U "&lt;SA_USER&gt;" -P "&lt;SA_PASSWORD&gt;" -C -d school-guardian \\

\-Q "SELECT ID, AUTHOR, DATEEXECUTED FROM DATABASECHANGELOG ORDER BY DATEEXECUTED;"

docker exec -it &lt;nombre-del-contenedor-sql&gt; /opt/mssql-tools18/bin/sqlcmd \\

\-S localhost,1433 -U "&lt;SA_USER&gt;" -P "&lt;SA_PASSWORD&gt;" -C -d school-guardian \\

\-Q "SELECT SCHEMA_NAME(SCHEMA_ID) AS esquema, COUNT(\*) AS tablas FROM sys.tables GROUP BY SCHEMA_NAME(SCHEMA_ID) ORDER BY 1;"

docker exec -it &lt;nombre-del-contenedor-sql&gt; /opt/mssql-tools18/bin/sqlcmd \\

\-S localhost,1433 -U "&lt;SA_USER&gt;" -P "&lt;SA_PASSWORD&gt;" -C -d school-guardian \\

\-Q "SELECT OBJECT_NAME(parent_object_id) AS tabla, name AS clave FROM sys.foreign_keys ORDER BY 1;"

La etapa es satisfactoria cuando no hay cambiosets pendientes, se contabilizan 11 esquemas y 38 tablas, no existe bloqueo de cambio activo y las claves foráneas entre esquemas están presentes.

### 4.2.5 Rollback

1. Se obtiene el SQL que se aplicaría sin ejecutarlo, para revisar el impacto.
2. Se revierte el último cambio aplicado cuando su reversión está declarada en el árbol de reversión.
3. Si no tiene reversión declarada, se aplica una corrección hacia adelante aprobada por el responsable de base de datos.
4. Toda reversión se registra con motivo y responsable.

docker run --rm liquibase/liquibase:4.29 \\

\--searchPath=/workspace \\

\--changeLogFile=&lt;changelog-maestro-del-esquema&gt; \\

\--url="jdbc:sqlserver://&lt;host-sql&gt;:1433;databaseName=school-guardian;encrypt=false;trustServerCertificate=true" \\

\--username="&lt;SA_USER&gt;" --password="&lt;SA_PASSWORD&gt;" \\

update-sql

La orden rollback-count 1 en lugar de update-sql revierte el último cambio. Antes de cualquier reversión en un entorno con datos se restaura un respaldo verificado; la reversión de esquema no sustituye a la recuperación de datos.

## 4.3 Step 3: Domain Services

### 4.3.1 Startup Order

Las dependencias síncronas se resuelven en el momento de la petición y no bloquean el arranque; el orden determinista lo fija la base de datos, ya disponible tras la etapa anterior. El orden recomendado garantiza que la identidad esté disponible antes que los servicios que validan personas y perfiles:

**Tabla 12**. Startup Order de los servicios de dominio

| Orden | Servicio                      | Esquema            | Entorno de ejecución  | Dependencia lógica principal         |
| ----- | ----------------------------- | ------------------ | --------------------- | ------------------------------------ |
| 1     | Identificación y acceso (IAM) | Iam                | Java 21 + Spring Boot | Emisión y verificación de tokens     |
| 2     | Gestión de usuarios           | UserManagement     | .NET 10               | Validación de identidad contra IAM   |
| 3     | Gestión escolar               | School, Geographic | .NET 10               | Personas y campus                    |
| 4     | Configuración                 | Settings           | .NET 10               | Ninguna: parámetros globales         |
| 5     | Flota                         | Fleet              | .NET 10               | Institución, dispositivos y perfiles |
| 6     | Rutas                         | Route              | .NET 10               | Flota, institución y perfiles        |
| 7     | Notificaciones                | Notification       | .NET 10               | Rutas, flota y perfiles              |
| 8     | Usos excepcionales            | Exceptional        | .NET 10               | Rutas, flota e institución           |
| 9     | Auditoría                     | Audit              | .NET 10               | Eventos de todos los dominios        |

### 4.3.2 Build and Launch

dotnet restore

dotnet build

dotnet test

./mvnw clean package

docker compose up -d --build

Cada conjunto publica en el puerto 8080 dentro de la red de contenedores, por lo que no se producen colisiones en el host. Los secretos y las direcciones de los demás servicios se inyectan por variables de entorno, nunca se fijan en el código.

### 4.3.3 Health Check and Ports

**Tabla 13**. Comprobaciones de salud de los servicios de dominio

| Servicio                | Comprobación                                              | Criterio de aceptación             |
| ----------------------- | --------------------------------------------------------- | ---------------------------------- |
| Identificación y acceso | Punto de salud del marco Spring (/actuator/health)        | HTTP 200 y estado UP               |
| Servicios .NET          | Punto de salud expuesto por el anfitrión web              | HTTP 200                           |
| Todos                   | Consulta por nombre de servicio en la red de contenedores | Respuesta en menos de dos segundos |
| Canal gRPC (5001)       | Establecimiento de canal desde el cliente                 | Canal disponible sin errores       |
| Broker                  | Producción y consumo de un evento de prueba               | Evento entregado al consumidor     |

curl -fsS http://&lt;nombre-del-servicio&gt;:8080/health

curl -fsS http://&lt;nombre-del-servicio-iam&gt;:8080/actuator/health

docker compose logs --tail=100 &lt;nombre-del-servicio&gt;

Ante una respuesta inesperada se verifica, en este orden: estado del contenedor, registros del servicio, variables de entorno, disponibilidad de la base de datos y del broker. No se continúa hasta que los nueve servicios respondan.

## 4.4 Step 4: Gateway and Routing

### 4.4.1 Service and Route Registration

La configuración del borde es declarada y se versiona como código; toda modificación entra por una solicitud de cambio revisada. Cada servicio de dominio registra su destino en la red de contenedores, sus rutas bajo el prefijo versionado /api/v1, los métodos admitidos y sus políticas.

**Tabla 14**. Servicios declarados en la puerta de enlace y sus rutas

| Servicio declarado      | Destino                               | Rutas públicas         | Métodos                         |
| ----------------------- | ------------------------------------- | ---------------------- | ------------------------------- |
| Identificación y acceso | http://&lt;iam&gt;:8080               | /api/v1/auth           | GET, POST, PUT, DELETE, OPTIONS |
| Gestión de usuarios     | http://&lt;user-management&gt;:8080   | Personas y familias    | GET, POST, PUT, DELETE, OPTIONS |
| Gestión escolar         | http://&lt;school-management&gt;:8080 | Instituciones y sedes  | GET, POST, PUT, DELETE, OPTIONS |
| Flota                   | http://&lt;fleet&gt;:8080             | Buses y asignaciones   | GET, POST, PUT, DELETE, OPTIONS |
| Rutas                   | http://&lt;route&gt;:8080             | Rutas y paradas        | GET, POST, PUT, DELETE, OPTIONS |
| Notificaciones          | http://&lt;notification&gt;:8080      | Alertas y bordadas     | GET, POST                       |
| Usos excepcionales      | http://&lt;exceptional&gt;:8080       | Usos excepcionales     | GET, POST, PUT, DELETE, OPTIONS |
| Auditoría               | http://&lt;audit&gt;:8080             | Auditoría y errores    | GET                             |
| Configuración           | http://&lt;configuration&gt;:8080     | Parámetros y políticas | GET, PUT                        |

Antes de publicar se comprueba que los prefijos declarados se ajusten al contrato versionado y que ningún servicio de dominio quede sin ruta publicada.

### 4.4.2 Cross-Cutting Policies

**Tabla 15**. Cross-Cutting Policies del borde

| Política           | Configuración                                                                                                                                                                                                  | Alcance                                       |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| CORS               | Origen web del entorno y orígenes de desarrollo autorizados; métodos GET, POST, PUT, DELETE, OPTIONS; encabezamientos Accept, Authorization, Content-Type; credenciales activas; anticipación de 3600 segundos | Todas las rutas                               |
| Límite de tasa     | 300 peticiones por minuto, política local, clasificación por encabezamiento Authorization                                                                                                                      | Ruta de autenticación, extensible a las demás |
| Autenticación      | Verification de token JWT RS256 en el borde, con denegación por defecto                                                                                                                                        | Todos los recursos salvo login y refresco     |
| Terminación de TLS | Certificado del entorno en el proxy 8443                                                                                                                                                                       | Tráfico público                               |
| Registro           | Salida estándar con identificador de correlación                                                                                                                                                               | Todas las rutas                               |

### 4.4.3 Verification del ruteo

curl -fsS <http://localhost:8000/health>

curl -i -X GET <http://localhost:8000/api/v1/&lt;recurso-protegido>&gt;

curl -i -X GET <http://localhost:8000/api/v1/&lt;recurso-protegido>&gt; \\

\-H "Authorization: Bearer &lt;ACCESS_TOKEN&gt;"

Cierre: el borde responde; el recurso protegido sin credenciales es denegado; con credenciales válidas responde el servicio de destino; y el origen de la aplicación web figura en la política CORS.

## 4.5 Step 5: Web Application

1. Se ejecuta la construcción de producción con el origen del entorno como valor de configuración.
2. Se revisa el resultado: sin errores de compilación ni de análisis estático.
3. Se publica el artefacto estático en el servicio de contenido del entorno, servido por HTTPS.
4. Se registra el origen publicado en la política CORS del borde.
5. Se invalida la caché de contenido estático de la versión anterior.

npm ci

npm run lint

npm run test

npm run build -- --configuration production

Cierre: la aplicación carga desde el origen publicado, el navegador no reporta violaciones de CORS, el inicio de sesión muestra el formulario conforme a la política de longitud vigente y el menú lateral refleja el rol autenticado.

## 4.6 Step 6: Mobile Application

1. Se fija la versión del binario conforme al esquema de la sección 6.
2. Se compila para las dos plataformas con la configuración del entorno de destino.
3. Se registran las credenciales de notificaciones push y la dirección base del gateway.
4. Se verifican los permisos de cámara (escaneo QR), notificaciones y ubicación.
5. Se distribuye: paquete de distribución interna para pruebas y paquete de tienda para producción.
6. Se instala en un terminal de prueba por rol y se ejecuta el recorrido mínimo.

npm ci

npm run test

npx eas build --platform android --profile production

npx eas build --platform ios --profile production

Cierre: el conductor inicia sesión, consulta la ruta del día, abre el escáner y registra un escaneo de prueba; el acudiente consulta el seguimiento y recibe una notificación; el token de notificaciones queda registrado en la plataforma.

## 4.7 Paso 7: configuración inicial

### 4.7.1 Variables de entorno

**Tabla 16**. Variables de entorno de la configuración inicial

| Variable                              | Descripción                             | Valor de ejemplo                                                                                                           |
| ------------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| MSSQL_DATABASE                        | Nombre de la base de datos              | school-guardian                                                                                                            |
| MSSQL_USER / MSSQL_SA_PASSWORD        | Usuario y clave del motor               | &lt;SA_USER&gt; / &lt;SA_PASSWORD&gt;                                                                                      |
| ConnectionStrings_\_DefaultConnection | Cadena de conexión de cada servicio     | Server=sqlserver;Database=school-guardian;User Id=&lt;SA_USER&gt;;Password=&lt;SA_PASSWORD&gt;;TrustServerCertificate=True |
| Kafka_\_BootstrapServers              | Dirección del broker (.NET)             | kafka:9092                                                                                                                 |
| KAFKA_BOOTSTRAP_SERVERS               | Dirección del broker (Java)             | &lt;nombre-del-broker&gt;:9092                                                                                             |
| SCHOOL_GUARDIAN_DB_DIR                | Ubicación base de los changelogs        | valor por defecto                                                                                                          |
| KONG_DECLARATIVE_CONFIG               | Manifiesto del borde                    | /opt/kong/kong.yml                                                                                                         |
| Services_\_UserManagement_\_BaseUrl   | Dirección base del servicio de usuarios | http://&lt;user-management&gt;:8080                                                                                        |
| Services_\_Route_\_BaseUrl            | Dirección base del servicio de rutas    | http://&lt;route&gt;:8080                                                                                                  |
| CORS_ORIGIN                           | Origen web permitido                    | https://&lt;produccion-domain&gt;                                                                                          |
| LOG_LEVEL / REDIS_URL                 | Bitácora y almacén efímero              | info / redis://&lt;redis&gt;:6379                                                                                          |
| JWT_PRIVATE_KEY                       | Llave privada RS256, solo en IAM        | &lt;JWT_PRIVATE_KEY&gt;                                                                                                    |
| PUSH_CREDENTIALS                      | Credenciales de notificaciones push     | &lt;PUSH_CREDENTIALS&gt;                                                                                                   |
| DEMO_PASSWORD                         | Clave de los perfiles de demostración   | &lt;DEMO_PASSWORD&gt;                                                                                                      |

### 4.7.2 Políticas de seguridad

Las tablas de políticas se crean vacías durante la migración, de modo que su carga es obligatoria:

**Tabla 17**. Parámetros de las políticas de seguridad

| Parámetro                      | Contenido                                           | Verification                                           |
| ------------------------------ | --------------------------------------------------- | ------------------------------------------------------ |
| Longitud de la contraseña      | longitud mínima y máxima acordadas con el cliente   | Registro fuera de rango rechazado                      |
| Mayúsculas, números y símbolos | Requisito activo o inactivo por regla               | Registro sin el requisito rechazado cuando está activo |
| Expiración en días             | Vigencia declarada en la política de contraseñas    | Cambio obligatorio al vencer el plazo                  |
| Ajustes de seguridad           | Pares parámetro–valor de sesión, intentos y bloqueo | Los valores se reflejan en la aplicación               |
| Orígenes y límite de tasa      | Configuración declarada en el borde                 | Verificada en el paso 4.4                              |

### 4.7.3 Catálogos y datos de referencia

La migración carga datos de demostración que deben revisarse antes de la operación real:

**Tabla 18**. Catálogos y datos de referencia cargados

| Catálogo                 | Contenido cargado                                        | Acción requerida                              |
| ------------------------ | -------------------------------------------------------- | --------------------------------------------- |
| Roles y permisos         | Los cinco roles del modelo y permisos por rol            | Depurar los roles adicionales de demostración |
| Personas y perfiles      | Población de demostración por rol con hash de clave      | Restablecer las claves antes de la operación  |
| Ciudades                 | Ciudades de referencia con país y estado                 | Confirmar o reemplazar por el catálogo real   |
| Instituciones y sedes    | Instituciones, sedes y cursos de ejemplo                 | Reemplazar por la institución real            |
| Flota y dispositivos     | Marcas, modelos, buses y dispositivos de geolocalización | Reemplazar por la flota real                  |
| Rutas y paradas          | Rutas, paradas, programaciones y ejecuciones de ejemplo  | Reemplazar por las rutas reales               |
| Tipos de alerta          | Veinte tipos de alerta con nivel de urgencia             | Ajustar la taxonomía si se solicita           |
| Sesiones de demostración | Registros de sesión con dirección IP de ejemplo          | Eliminar en producción                        |

### 4.7.4 Usuarios semilla por rol

**Tabla 19**. Usuarios semilla por rol

| Rol         | Estado tras la migración                                 | Acción de configuración inicial                                                                                                  |
| ----------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| SUPER_ADMIN | El rol existe en el catálogo; no hay perfil asociado     | Crear el perfil administrador de plataforma con &lt;ADMIN_EMAIL&gt; y &lt;ADMIN_PASSWORD&gt; y verificar su permiso de auditoría |
| ADMIN       | Perfiles de administrador escolar con persona asociada   | Verificar la sede asignada y restablecer la clave inicial                                                                        |
| DRIVER      | Perfiles de demostración con licencia asociada           | Restablecer claves y confirmar la asignación conductor–bus                                                                       |
| PARENT      | Perfiles de demostración con vínculo familiar            | Restablecer claves y verificar la relación acudiente–estudiante                                                                  |
| STUDENT     | Perfiles de demostración con identificación QR vinculada | Verificar la vinculación con sede y ruta                                                                                         |

curl -i -X POST <http://localhost:8000/api/v1/auth/login> \\

\-H "Content-Type: application/json" \\

\-d '{"email":"&lt;admin_inicial&gt;@ejemplo.com","password":"&lt;ADMIN_PASSWORD&gt;"}'

Cierre: cada uno de los cinco roles inicia sesión, los permisos corresponden al rol y la clave inicial obliga a cambio en el primer acceso.

### 4.7.5 Cierre de la configuración inicial

Se registra la configuración aplicada con fecha, responsable y entorno; se rotan las credenciales provisionales de desarrollo si el entorno es de pruebas o de producción; se confirma que ningún secreto quede presente en los artefactos publicados ni en los registros; y, por último, se ejecuta la verificación completa de la sección 5.

# 5 Verification posdespliegue

## 5.1 Criterio de veredicto

- **PASS**: la comprobación se ejecuta y cumple el criterio indicado.
- **FAIL**: no se ejecuta, no cumple el criterio o su resultado no es reproducible.
- La implantación solo se declara concluida con **PASS** en todos los puntos; un **FAIL** activa la sección 7 o la reapertura de la etapa correspondiente.
- Cada punto registra evidencia: salida de comando, respuesta o referencia de registro.

## 5.2 Checklist de verificación

**Infraestructura y datos**

**Tabla 20**. Checklist de verificación de infraestructura y datos

| #   | Comprobación           | Criterio                                            | Veredicto |
| --- | ---------------------- | --------------------------------------------------- | --------- |
| I-1 | Motor de datos         | Estado saludable y puerto 1433 solo en red interna  | PASS/FAIL |
| I-2 | Inicialización         | Base school-guardian creada                         | PASS/FAIL |
| I-3 | Broker                 | Seis tópicos listados e inicialización con código 0 | PASS/FAIL |
| I-4 | Borde                  | Respuesta de salud en el puerto 8000                | PASS/FAIL |
| I-5 | Puertos                | Ningún servicio de dominio expuesto al exterior     | PASS/FAIL |
| B-1 | Esquemas y tablas      | 11 esquemas y 38 tablas presentes                   | PASS/FAIL |
| B-2 | Migraciones            | Cero cambiosets pendientes y sin bloqueo activo     | PASS/FAIL |
| B-3 | Integridad referencial | Claves foráneas entre esquemas creadas              | PASS/FAIL |
| B-4 | Datos de referencia    | Catálogos cargados y revisados                      | PASS/FAIL |
| B-5 | Respaldo inicial       | Respaldo completo y de registro verificado          | PASS/FAIL |

**Servicios de dominio**

**Tabla 21**. Checklist de verificación de servicios de dominio

| #   | Comprobación   | Criterio                                                 | Veredicto |
| --- | -------------- | -------------------------------------------------------- | --------- |
| S-1 | Disponibilidad | Los nueve servicios responden su punto de salud          | PASS/FAIL |
| S-2 | Puertos        | Cada servicio publica únicamente en 8080                 | PASS/FAIL |
| S-3 | Canal gRPC     | Canal disponible en el puerto 5001                       | PASS/FAIL |
| S-4 | Eventos        | Producción y consumo de un evento de prueba              | PASS/FAIL |
| S-5 | Aislamiento    | Cada servicio opera sobre su propio esquema              | PASS/FAIL |
| S-6 | Bitácoras      | Registros estructurados con identificador de correlación | PASS/FAIL |

**API Gateway y clientes**

**Tabla 22**. Checklist de verificación de puerta de enlace y clientes

| #   | Comprobación          | Criterio                                                        | Veredicto |
| --- | --------------------- | --------------------------------------------------------------- | --------- |
| G-1 | Rutas                 | Un servicio declarado por cada dominio bajo /api/v1             | PASS/FAIL |
| G-2 | Autorización          | Recurso protegido denegado sin token válido                     | PASS/FAIL |
| G-3 | CORS y límite de tasa | Origen publicado y límite demostrado con peticiones de prueba   | PASS/FAIL |
| G-4 | Aplicación web        | Carga por HTTPS, sin violaciones de CORS y con menú por rol     | PASS/FAIL |
| G-5 | Aplicación móvil      | Escaneo QR, seguimiento y notificación de prueba satisfactorios | PASS/FAIL |

**Seguridad, configuración y operación**

**Tabla 23**. Checklist de verificación de seguridad, configuración y operación

| #   | Comprobación            | Criterio                                                          | Veredicto |
| --- | ----------------------- | ----------------------------------------------------------------- | --------- |
| C-1 | Secretos                | Ningún valor real en artefactos ni en registros                   | PASS/FAIL |
| C-2 | Políticas de contraseña | Longitud, mayúsculas, números, símbolos y expiración operativos   | PASS/FAIL |
| C-3 | Usuarios semilla        | Los cinco roles autentican con los permisos correctos             | PASS/FAIL |
| C-4 | Auditoría               | Acción crítica registrada con quién, qué, cuándo, IP y aplicación | PASS/FAIL |
| O-1 | Paneles                 | Disponibilidad, latencia P95, tasa de error, carga e instancias   | PASS/FAIL |
| O-2 | Alertas                 | Reglas activas y notificación de prueba recibida                  | PASS/FAIL |
| O-3 | Objetivos de servicio   | 99,9 % de disponibilidad y P95 menor a 300 ms instrumentados      | PASS/FAIL |
| O-4 | Soporte                 | Procedimientos de atención y escalamiento definidos               | PASS/FAIL |

## 5.3 Registro y anexo de evidencias

El responsable de calidad completa una fila por punto con identificador, fecha, entorno, responsable, resultado de la comprobación, veredicto y referencia de evidencia. El registro se conserva con el acta de aceptación y alimenta la revisión de dirección.

# 6 Versionado y publicación de versiones

## 6.1 Esquema de versionado

**Tabla 24**. Esquema de versionado de artefactos

| Artefacto                        | Esquema                                | Regla                                                                                                                |
| -------------------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Plataforma (versión de conjunto) | MAJOR.MINOR.PATCH                      | MAJOR ante cambios incompatibles de contrato o esquema; MINOR ante funcionalidad compatible; PATCH ante correcciones |
| Imagen de contenedor             | Etiqueta inmutable ligada a la versión | Nunca se reutiliza una etiqueta publicada                                                                            |
| Contrato de API                  | Prefijo /api/v1                        | Un cambio incompatible exige nueva versión de contrato                                                               |
| Eventos del broker               | Versionado aditivo                     | Los consumidores toleran campos desconocidos                                                                         |
| Migraciones                      | Cambiosets con identificador único     | Nunca se edita un cambioset ya aplicado                                                                              |
| Aplicación móvil                 | Número de versión visible al usuario   | Se incrementa en cada distribución                                                                                   |

## 6.2 Flujo de ramas y aprobaciones

main ← producción; solo recibe integraciones de publicación; siempre estable

└── dev ← integración continua; recibe las ramas de funcionalidad

├── feature/\[descripcion\] ← una historia de usuario

├── fix/\[descripcion\] ← una corrección

├── chore/\[descripcion\] ← infraestructura, dependencias, documentación

└── hotfix/\[descripcion\] ← corrección urgente directa a producción

**Figura 2**. Flujo de ramas y aprobaciones

Reglas: ninguna integración directa en las ramas protegidas; una tarea por rama; revisión con al menos una aprobación; tamaño máximo recomendado de 400 líneas de código excluyendo pruebas; mensajes de integración con formato convencional (feat, fix, docs, refactor, test, chore, perf).

## 6.3 Contenido de una publicación

1. Versión asignada y etiqueta de imagen correspondiente.
2. Relación de historias de usuario y correcciones incluidas.
3. Cambios de esquema aplicados y su estado de reversión.
4. Cambios de configuración del borde.
5. Resultado de la verificación posdespliegue.
6. Instrucciones de reversión vigentes para esa versión.
7. Registro en el historial de despliegues: fecha, servicio, versión, entorno, responsable e incidencias.

## 6.4 Ventanas de publicación

**Tabla 25**. Ventanas de publicación por entorno

| Entorno     | Ventana                                                                                    | Aprobación                                                  |
| ----------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| Integración | Continua tras cada integración aprobada                                                    | Automática si todas las puertas pasan                       |
| Pruebas     | Continua tras integrar en la rama principal                                                | Automática, con aviso al responsable de producto            |
| Producción  | Ventana acordada con la institución, evitando el horario de servicio de transporte escolar | Responsable técnico y, en cambios mayores, doble aprobación |

# 7 Procedimiento de reversión (rollback)

## 7.1 Disparadores

**Tabla 26**. Disparadores de reversión y decisión asociada

| Disparador                         | Condición de activación                                       | Decisión                                         |
| ---------------------------------- | ------------------------------------------------------------- | ------------------------------------------------ |
| Verification posdespliegue fallida | Al menos un punto en FAIL tras dos intentos de corrección     | Rollback total a la versión anterior            |
| Despliegue canáreo degradado       | Métricas fuera de umbral durante la observación de 15 minutos | Rollback automática                             |
| Tasa de error elevada              | Superior al 5 % durante 5 minutos                             | Rollback inmediata                              |
| Latencia degradada                 | P95 superior a 500 ms durante 5 minutos                       | Rollback o mitigación según alcance             |
| Servicio caído                     | Comprobación de salud fallida durante más de 1 minuto         | Reinicio y, si persiste, reversión               |
| Migración incompatible             | Error de esquema que impide el funcionamiento                 | Detención y reversión de datos previa aprobación |
| Incidente P0 o P1                  | Clasificación de severidad crítica                            | Decisión del comandante de incidente             |

## 7.2 Pasos de reversión

1. Se declara el incidente y se clasifica su severidad.
2. Se comunica la reversión en curso a las partes interesadas.
3. Se detiene la publicación y se congelan los cambios relacionados.
4. Se restaura la versión anterior de las imágenes de los servicios afectados.
5. Si cambió la configuración del borde, se restaura el manifiesto declarado anterior.
6. Si se publicó la aplicación web, se restaura el artefacto anterior y se limpia la caché.
7. Si se distribuyó la aplicación móvil, se retira el canal y se comunica la versión a usar.
8. Se ejecuta la verificación de la sección 5 sobre los puntos afectados.
9. Se cierra el incidente con causa raíz y acciones de mejora.

kubectl rollout undo deployment/&lt;servicio&gt; -n &lt;espacio-de-nombres&gt;

kubectl rollout status deployment/&lt;servicio&gt; -n &lt;espacio-de-nombres&gt;

En despliegues basados en contenedores sin orquestador, la reversión consiste en volver a involucrar la etiqueta anterior de la imagen y reiniciar el conjunto de servicios.

## 7.3 Rollback de la base de datos

1. Se suspenden las escrituras de los servicios afectados.
2. Se confirma que el cambio dispone de reversión declarada.
3. Se ejecuta la reversión de un cambio conforme a la sección 4.2.5.
4. Si no existe reversión declarada, se restaura el respaldo previo más reciente en una instancia de espera, se apunta la configuración de conexión en una ventana controlada y se verifica esquema por esquema: inicio de sesión de identidad, lectura escolar, últimas posiciones de geolocalización y anexo de auditoría.
5. Se documenta la decisión sobre la ventana de eventos del broker: reproducir desde el punto de control o aceptar la brecha.
6. Se registra la operación en el informe de incidente.

**Tabla 27**. Métodos de respaldo y puntos de recuperación

| Activo                              | Método de respaldo                           | Punto de recuperación | Tiempo de recuperación | Responsable                       |
| ----------------------------------- | -------------------------------------------- | --------------------- | ---------------------- | --------------------------------- |
| SQL Server: completos y de registro | Respaldo nativo o gestionado                 | ≤ 15 min              | ≤ 2 h                  | Plataforma y responsable de datos |
| Changelogs de migración             | Historial de versiones del código            | 0                     | Minutos                | Todos los servicios               |
| Retención del broker                | Retención por tópico y espejo si se requiere | Por tópico            | Por tópico             | Plataforma                        |
| Configuración del borde             | Exportación declarada versionada             | 0                     | Minutos                | Borde                             |
| Ephemeral Store                     | Sin requisito de restauración                | —                     | Recreación             | Borde                             |

## 7.4 Verification posterior a la reversión

**Tabla 28**. Verification posterior a la reversión

| #   | Comprobación  | Criterio                                                   | Veredicto |
| --- | ------------- | ---------------------------------------------------------- | --------- |
| R-1 | Servicios     | Los servicios afectados responden su punto de salud        | PASS/FAIL |
| R-2 | Base de datos | Migraciones consistentes y sin cambiosets pendientes       | PASS/FAIL |
| R-3 | Borde         | Rutas y políticas de la versión anterior activas           | PASS/FAIL |
| R-4 | Clientes      | Aplicación web y móvil operan contra la versión restaurada | PASS/FAIL |
| R-5 | Métricas      | Tasa de error y latencia dentro de umbral                  | PASS/FAIL |
| R-6 | Registro      | Incidente documentado con causa raíz y acciones            | PASS/FAIL |

# 8 Canal de integración y publicación continua

## 8.1 Etapas del canal

**Tabla 29**. Etapas del canal de integración y publicación continua

| Orden | Etapa                          | Disparador                                     | Duración objetivo | Contenido                                                                                                                                                      |
| ----- | ------------------------------ | ---------------------------------------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Solicitud de cambio            | Cualquier actualización de rama                | < 2 min           | Análisis de estilo, validación de contratos y detección de cambios incompatibles; si falla, se bloquea la fusión                                               |
| 2     | Calidad de código              | Cambios en el código de un servicio            | < 5 min           | Pruebas unitarias, compilación y análisis estático de seguridad                                                                                                |
| 3     | Validación de cambios de datos | Cambios en changelogs                          | < 3 min           | Validación de changelogs y vista previa del SQL; si falla, se bloquea la fusión                                                                                |
| 4     | Despliegue a integración       | Fusión aprobada en la rama de desarrollo       | < 15 min          | Pruebas de integración, construcción de imágenes, despliegue y comprobación de humo en el puerto 8000                                                          |
| 5     | Despliegue a pruebas           | Fusión en la rama principal                    | < 45 min          | Batería completa, contratos, etiquetado de imagen, despliegue, comprobación de humo, rendimiento y notificación al responsable de producto                     |
| 6     | Producción                     | Aprobación manual tras pasar todas las puertas | Variable          | Migración previa al código, despliegue canáreo al 10 % y observación de 15 minutos: satisfactorio → 100 % del tráfico; no satisfactorio → reversión automática |

## 8.2 Puertas de calidad

**Tabla 30**. Puertas de calidad y efecto de su fallo

| Puerta                     | Condición de paso                                                               | Efecto si falla            |
| -------------------------- | ------------------------------------------------------------------------------- | -------------------------- |
| Estilo y formato           | Sin hallazgos                                                                   | Bloqueo de la fusión       |
| Contratos                  | Validación satisfactoria y sin cambios incompatibles no declarados              | Bloqueo de la fusión       |
| Pruebas unitarias          | 100 % satisfactorias; cobertura de líneas ≥ 80 % y ≥ 90 % en la capa de dominio | Bloqueo de la fusión       |
| Pruebas de integración     | Todas satisfactorias                                                            | Detención del despliegue   |
| Cambios de datos           | Validación de changelogs satisfactoria                                          | Bloqueo de la fusión       |
| Humo                       | Respuesta de salud del borde y recorrido mínimo                                 | Detención del despliegue   |
| Rendimiento                | P95 menor a 300 ms y tasa de error inferior al 1 %                              | Bloqueo de la publicación  |
| Verification posdespliegue | Todos los puntos en PASS                                                        | Activación de la reversión |

Los umbrales de cobertura y rendimiento proceden de la estrategia de calidad del proyecto; cuando un umbral no esté formalizado se propone como criterio de aceptación y requiere aprobación antes de usarse como puerta.

## 8.3 Publicación canárea en producción

**Tabla 31**. Pasos de la publicación canárea en producción

| Paso | Acción                                                          | Criterio de continuación                 |
| ---- | --------------------------------------------------------------- | ---------------------------------------- |
| 1    | Aprobación formal del responsable técnico                       | Publicación aprobada                     |
| 2    | Verification de pruebas de la versión, incluido rendimiento     | Todas las puertas en PASS                |
| 3    | Aprobación del alcance funcional por el responsable de producto | Historias incluidas aceptadas            |
| 4    | Migración de base de datos antes del código                     | Verification de migración satisfactoria  |
| 5    | Despliegue al 10 % del tráfico                                  | Métricas en umbral durante 15 minutos    |
| 6    | Ampliación al 100 % del tráfico                                 | Sin alertas activas de severidad P0 o P1 |
| 7    | Observación intensiva los primeros 30 minutos                   | Responsable de guardia disponible        |
| 8    | Registro de la publicación                                      | Historial de despliegues actualizado     |

# 9 Monitoreo y soporte post-implementación

## 9.1 Objetivos de nivel de servicio

**Tabla 32**. Objetivos de nivel de servicio

| Indicador                 | Objetivo                          | Medición                             | Presupuesto de error                          |
| ------------------------- | --------------------------------- | ------------------------------------ | --------------------------------------------- |
| Disponibilidad            | 99,9 %                            | Peticiones exitosas en 30 días       | 43,8 minutos de indisponibilidad al mes       |
| Latencia de API           | P95 menor a 300 ms                | Histograma de duración de peticiones | Alerta por encima de 500 ms durante 5 minutos |
| Tasa de error             | Inferior al 1 % en carga          | Fallidas sobre total                 | Alerta por encima del 5 % durante 5 minutos   |
| Entrega de notificaciones | Tasa de notificaciones entregadas | Entregadas sobre emitidas            | Definido con la institución                   |
| Registro de bordadas      | Registro sin incidencias          | Bordadas registradas sin error       | Definido con la institución                   |

## 9.2 Métricas, bitácoras y trazas

**Tabla 33**. Pilares de observabilidad y herramienta asociada

| Pilar     | Contenido                                                                                                              | Herramienta                                      |
| --------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Bitácoras | JSON estructurado con timestamp, level, service, version, environment, correlationId, userId, message y contexto       | Recolección centralizada con consulta por índice |
| Métricas  | http_requests_total, http_request_duration_seconds, http_requests_in_flight, métricas de negocio y de conexión a datos | Prometheus y paneles de Grafana                  |
| Trazas    | Propagación del identificador de trazado entre borde y servicios, con instrumentación automática del acceso a datos    | OpenTelemetry con consola de trazas              |

El punto de exposición de métricas (GET /metrics) se mantiene solo en la red interna y no se publica en el borde. No se registran claves, tokens completos ni datos personales sin enmascarar. Los paneles requeridos son: ejecutivo (tasa de error, latencia P95 por servicio, carga, disponibilidad frente al objetivo e instancias) y por servicio (métricas de tasa, error y duración, entorno de ejecución, conexión a datos y métricas del broker).

## 9.3 Reglas de alerta

**Tabla 34**. Reglas de alerta y severidad asociada

| Alerta                | Condición                                     | Severidad | Destino                       |
| --------------------- | --------------------------------------------- | --------- | ----------------------------- |
| Tasa de error alta    | Superior al 5 % durante 5 minutos             | P1        | Guardia y canal de incidentes |
| Latencia alta         | P95 superior a 500 ms durante 5 minutos       | P2        | Canal de alertas              |
| Servicio caído        | Comprobación de salud fallida más de 1 minuto | P1        | Guardia                       |
| Mensajes sin consumir | Antigüedad fuera de umbral                    | P2        | Canal de alertas              |
| Espacio en disco bajo | Uso superior al 80 %                          | P2        | Canal de alertas              |
| Reinicios frecuentes  | Más de tres reinicios en 10 minutos           | P1        | Guardia                       |

## 9.4 Gestión de incidentes

**Tabla 35**. Niveles de gestión de incidentes

| Nivel | Definición                                                    | Respuesta           | Resolución |
| ----- | ------------------------------------------------------------- | ------------------- | ---------- |
| P0    | Sistema totalmente caído con impacto total                    | Inmediato (< 5 min) | 1 hora     |
| P1    | Funcionalidad crítica degradada con impacto mayoritario       | 15 minutos          | 4 horas    |
| P2    | Funcionalidad secundaria degradada con alternativa disponible | 1 hora              | 24 horas   |
| P3    | Inconveniente menor sin impacto operativo                     | 4 horas             | 1 semana   |

Secuencia de atención: detección y acuse de la alerta → clasificación de severidad → comunicación inicial con servicio afectado, impacto y próxima actualización → diagnóstico con paneles, bitácoras, trazas, estado de contenedores y comprobación manual de salud → mitigación empezando por la opción más segura (reversión, desactivación de la funcionalidad, escalamiento, aumento de tiempos de espera) → resolución cuando las métricas recuperan sus niveles, las comprobaciones de salud responden y el responsable de producto confirma la operación → retrospectiva dentro de los dos días hábiles siguientes, con causa raíz y acciones.

## 9.5 Respaldos y recuperación

**Tabla 36**. Frecuencia de verificación de respaldos y restauraciones

| Verification                                        | Frecuencia         |
| --------------------------------------------------- | ------------------ |
| Éxito del trabajo de respaldo                       | Diaria, con alerta |
| Ejercicio de restauración a un punto en el tiempo   | Trimestral         |
| Integridad de los changelogs en la copia restaurada | Cada ejercicio     |

Principios: se respalda la instancia completa y se documentan los responsables de recuperación por esquema; la configuración y los secretos del borde forman parte del conjunto de respaldo; un respaldo que nunca se ha restaurado no es un respaldo. La restauración contempla declarar la severidad, identificar el respaldo completo más reciente y los registros posteriores, restaurar en una instancia de espera, cambiar la configuración de conexión en ventana controlada, verificar por esquema, decidir la ventana de reproducción de eventos y cerrar con retrospectiva.

## 9.6 Mantenimiento y escalamiento

- **Ventanas de mantenimiento**: programadas con la institución fuera del horario de servicio de transporte; se comunican con anticipación y se limitan a cambios aprobados.
- **Actualizaciones**: se sigue el orden de la sección 6.3 —migración, servicios, configuración del borde, humo— con verificación al cierre de cada etapa.
- **Escalamiento horizontal**: los servicios son sin estado y se escalan detrás del borde; rutas y notificaciones escalan por volumen de eventos, y los de lectura por número de peticiones.
- **Presupuesto de error**: agotado el objetivo de disponibilidad, se priorizan correcciones por encima de nueva funcionalidad.
- **Soporte post-implementación**: periodo de acompañamiento con guardia definida, revisión semanal de alertas y reporte de estado al cierre de cada semana.

# 10 Matriz RACI y cronograma de implantación

## 10.1 Roles del equipo de implantación

**Tabla 37**. Roles del equipo de implantación

| Rol                          | Responsabilidad principal                                 |
| ---------------------------- | --------------------------------------------------------- |
| Líder técnico                | Dirección técnica, aprobaciones y decisiones de reversión |
| Ingeniero de plataforma      | Infraestructura, servicios, borde y clientes              |
| Responsable de base de datos | Migraciones, verificación, respaldos y recuperación       |
| Responsable de calidad       | Verification posdespliegue y emisión de veredictos        |
| Responsable de operaciones   | Monitoreo, alertas, incidentes y mantenimiento            |
| Cliente (institución)        | Definición de datos, observaciones y aceptación           |
| Asesor académico             | Acompañamiento y revisión del procedimiento               |

## 10.2 Matriz RACI

R: responsable de ejecución · A: responsable último · C: consultado · I: informado

**Tabla 38**. Matriz RACI de las actividades de implantación

| Actividad                                 | Líder | Plataforma | Datos | Calidad | Operaciones | Cliente | Asesor |
| ----------------------------------------- | ----- | ---------- | ----- | ------- | ----------- | ------- | ------ |
| Validación de prerrequisitos              | A     | R          | C     | C       | I           | I       | I      |
| Infraestructura base                      | A     | R          | C     | I       | I           | I       | I      |
| Migraciones y verificación de esquemas    | A     | C          | R     | C       | I           | I       | I      |
| Despliegue de servicios de dominio        | A     | R          | C     | I       | I           | I       | I      |
| API Gateway y ruteo                  | A     | R          | I     | C       | C           | I       | I      |
| Publicación de la aplicación web          | A     | R          | I     | C       | I           | I       | I      |
| Generación y distribución de la app móvil | A     | R          | I     | C       | I           | C       | I      |
| Configuración inicial y usuarios semilla  | A     | C          | C     | I       | R           | C       | I      |
| Verification posdespliegue                | A     | C          | C     | R       | C           | I       | I      |
| Rollback ante incidencias                | A     | R          | R     | C       | C           | I       | I      |
| Monitoreo y atención de incidentes        | A     | C          | I     | I       | R           | I       | I      |
| Aceptación y cierre                       | A     | C          | I     | C       | C           | R       | C      |

## 10.3 Cronograma

Duraciones estimadas de planificación; los compromisos contractuales se fijan con el cliente.

**Tabla 39**. Cronograma estimado de implantación

| Fase                      | Actividad central                                     | Duración estimada | Dependencia         | Resultado                 |
| ------------------------- | ----------------------------------------------------- | ----------------- | ------------------- | ------------------------- |
| 1 Preparación           | Prerequisites, secretos y accesos                    | 2 jornadas        | Línea base aprobada | Entorno habilitado        |
| 2 Infraestructura base  | Motor, broker, almacén efímero y borde                | 1 jornada         | Fase 1              | Infraestructura saludable |
| 3 Base de datos         | Migraciones de los 11 esquemas y verificación         | 2 jornadas        | Fase 2              | 38 tablas disponibles     |
| 4 Servicios de dominio  | Construcción, arranque y salud de los nueve servicios | 2 jornadas        | Fase 3              | Servicios operativos      |
| 5 API Gateway      | Rutas, políticas y verificación de ruteo              | 1 jornada         | Fase 4              | Tráfico enrutado          |
| 6 Clientes              | Publicación web y generación y distribución móvil     | 2 jornadas        | Fase 5              | Aplicaciones disponibles  |
| 7 Configuración inicial | Variables, políticas, catálogos y usuarios semilla    | 1 jornada         | Fase 6              | Sistema parametrizado     |
| 8 Verification          | Checklist completo con veredicto                      | 2 jornadas        | Fase 7              | Veredicto PASS            |
| 9 Aceptación            | Demostración, observaciones y firma                   | 1 jornada         | Fase 8              | Acta firmada              |
| 10 Soporte              | Acompañamiento post-implementación                    | 30 días           | Fase 9              | Cierre del soporte        |

Total estimado: 14 jornadas de implantación más 30 días de soporte posterior.

## 10.4 Hitos y puertas de aceptación

**Tabla 40**. Hitos y puertas de aceptación

| Hito                       | Puerta de paso                                     | Evidencia                        |
| -------------------------- | -------------------------------------------------- | -------------------------------- |
| M1 — Infraestructura lista | Salud de motor, broker y borde                     | Salidas de comprobación          |
| M2 — Datos disponibles     | Sin cambiosets pendientes; 11 esquemas y 38 tablas | Consulta de historial de cambios |
| M3 — Servicios operativos  | Salud de los nueve servicios y canal gRPC          | Salidas de comprobación          |
| M4 — Tráfico enrutado      | Denegación sin token y enrutamiento con token      | Salidas de comprobación          |
| M5 — Clientes publicados   | Recorrido web y móvil satisfactorios               | Registro de recorrido            |
| M6 — Sistema parametrizado | Políticas y usuarios semilla operativos            | Registro de configuración        |
| M7 — Aceptación            | Verification en PASS y firma del acta              | Acta de aceptación               |

# 11 Gestión de cambios y aceptación

## 11.1 Clasificación de los cambios

**Tabla 41**. Clasificación de los cambios

| Tipo       | Alcance                                            | Ejemplo                                          | Aprobación                                          |
| ---------- | -------------------------------------------------- | ------------------------------------------------ | --------------------------------------------------- |
| Estándar   | Preaprobado y reversible                           | Ajuste de nivel de bitácora, rotación de secreto | Responsable de plataforma                           |
| Mayor      | Afecta esquema, contrato o configuración del borde | Nueva columna, nueva ruta pública                | Responsable técnico y responsable de base de datos  |
| Emergencia | Corrige una incidencia en curso                    | Rollback de versión defectuosa                  | Comandante de incidente, con ratificación posterior |

## 11.2 Flujo de aprobación

1. **Solicitud**: identificación del cambio, solicitante, motivo, entorno afectado y urgencia.
2. **Evaluación de impacto**: componentes implicados, riesgo, necesidad de migración y plan de reversión.
3. **Aprobación**: según la clasificación de la sección 11.1.
4. **Preparación**: ejecución previa en el entorno de pruebas con su verificación.
5. **Ejecución**: en la ventana autorizada, siguiendo la sección 4 y el orden migración → servicios → configuración del borde → humo.
6. **Verification**: checklist de la sección 5 acotado a lo modificado.
7. **Cierre**: registro actualizado, comunicación a las partes y, si aplica, actualización de la instrucción de reversión.

## 11.3 Registro de cambios

**Tabla 42**. Campos del registro de cambios

| Campo                     | Contenido                                            |
| ------------------------- | ---------------------------------------------------- |
| Identificador y fecha     | Consecutivo único y fecha de solicitud               |
| Descripción y solicitante | Qué cambia, por qué y quién lo solicita              |
| Clasificación             | Estándar, mayor o emergencia                         |
| Componentes afectados     | Servicio, esquema, borde o cliente                   |
| Migración asociada        | Sí/no y su estado de reversión                       |
| Aprobador y ventana       | Responsable que autoriza y fecha autorizada          |
| Verification              | Resultado del checklist aplicable                    |
| Rollback disponible      | Sí/no y su procedimiento                             |
| Estado                    | Solicitado, aprobado, ejecutado, verificado, cerrado |

## 11.4 Acceptance Criteria

**Tabla 43**. Acceptance Criteria y su verificación

| #    | Criterio                                                                    | Verification                                      |
| ---- | --------------------------------------------------------------------------- | ------------------------------------------------- |
| A-1  | Todos los puntos del checklist en estado PASS                               | Registro de verificación firmado                  |
| A-2  | 11 esquemas y 38 tablas sin cambiosets pendientes                           | Consulta de historial de cambios                  |
| A-3  | Los nueve servicios operan y responden su salud                             | Salidas de comprobación                           |
| A-4  | Único punto de entrada público: puerta de enlace en el puerto 8000          | Inventario de puertos                             |
| A-5  | Aplicación web publicada por HTTPS con menú por rol                         | Recorrido de aceptación                           |
| A-6  | Aplicaciones móviles instaladas con escaneo QR y notificaciones funcionales | Recorrido por rol                                 |
| A-7  | Los cinco roles autentican con los permisos correspondientes                | Registro de sesión por rol                        |
| A-8  | Políticas de contraseña y ajustes de seguridad operativos                   | Pruebas de registro y expiración                  |
| A-9  | 99,9 % de disponibilidad y P95 menor a 300 ms instrumentados                | Paneles y reglas de alerta                        |
| A-10 | Respaldos con recuperación ≤ 15 min y ≤ 2 h                                 | Registro de respaldos y ejercicio de restauración |
| A-11 | Instrucciones de reversión disponibles y probadas                           | Registro de la prueba de reversión                |
| A-12 | Personal de operaciones capacitado en atención de incidentes                | Registro de capacitación                          |

## 11.5 Acta de aceptación y cierre

El acta contiene: identificación de la plataforma y de la versión implantada, entorno, fecha, alcance verificado, resultado del checklist con su veredicto global, observaciones con su plan de atención, responsables de la implantación y de la operación, y el compromiso de soporte posterior.

**Tabla 44**. Campos del acta de aceptación

| Campo                     | Contenido                                         |
| ------------------------- | ------------------------------------------------- |
| Versión implantada        | Versión de conjunto e imágenes correspondientes   |
| Fecha de implantación     | Fecha de cierre de la verificación                |
| Veredicto global          | PASS / FAIL                                       |
| Observaciones del cliente | Lista con responsable y fecha compromiso          |
| Firmas                    | Responsable técnico y cliente                     |
| Soporte comprometido      | Periodo, canal de atención y tiempos de respuesta |

Si el veredicto global es FAIL, la implantación no se cierra: se activa la sección 7 o se reabre la etapa correspondiente, y la aceptación se agenda de nuevo cuando todos los puntos estén en PASS. Una vez firmada, el sistema pasa a operación conforme a la sección 9, y los cambios posteriores se gestionan conforme a la sección 11.