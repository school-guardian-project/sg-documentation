# Manual de Despliegue - School Guardian

## Índice
1. [Objetivo](#objetivo)
2. [Arquitectura General](#arquitectura-general)
3. [Requisitos Previos](#requisitos-previos)
4. [Despliegue del Backend en AWS EC2](#despliegue-del-backend-en-aws-ec2)
5. [Base de Datos (RDS SQL Server)](#base-de-datos-rds-sql-server)
6. [API Gateway (Kong) + Reverse Proxy (Caddy)](#api-gateway-kong--reverse-proxy-caddy)
7. [Servicio GPS (ms-gps)](#servicio-gps-ms-gps)
8. [Despliegue Web (Frontend Angular) en Vercel](#despliegue-web-frontend-angular-en-vercel)
9. [Aplicación Móvil (React Native + Expo)](#aplicación-móvil-react-native--expo)
10. [Verificación Post-Despliegue](#verificación-post-despliegue)
11. [Troubleshooting](#troubleshooting)
12. [Anexos](#anexos)

## 1. Objetivo

Este manual describe el paso a paso ejecutado para desplegar completamente la plataforma School Guardian: backend con 7 microservicios + Kong + Caddy en AWS EC2, base de datos SQL Server en AWS RDS, aplicación web Angular en Vercel, y aplicación móvil React Native (Expo) compilada para Android/iOS.

## 2. Arquitectura General

| Componente | Entorno | Descripción |
|---|---|---|
| **Base de datos** | AWS RDS (SQL Server Express) | Esquemas: Iam, UserManagement, School, Fleet, Route, Notification, Gps |
| **Backend** | AWS EC2 (Debian 13) | 7 microservicios .NET/Java + Kafka + Zookeeper + Kong (API Gateway) + Caddy (Reverse Proxy) |
| **Web Frontend** | Vercel | Aplicación Angular SPA accesible públicamente |
| **Móvil** | Expo EAS Build | Aplicación React Native para Android (.APK) e iOS (.IPA) |
| **GPS Tracker** | Dispositivo VT03F | Conexión TCP directa al EC2 puerto 8842 |

URLs/Endpoints principales:
- Web/API pública: `https://3-15-21-147.sslip.io` (Caddy + Kong)
- GPS TCP (tracker): `3.15.21.147:8842`
- RDS: `school-guardian.choygiky8hou.us-east-2.rds.amazonaws.com:1433`

## 3. Requisitos Previos

- Acceso SSH a EC2: `ssh -i ~/Descargas/sg-ssh.pem admin@ec2-3-15-21-147.us-east-2.compute.amazonaws.com`
- Claves locales: `sg-ssh.pem`, proyecto backend/web/móvil
- Cuentas: Vercel, Expo/EAS (cuenta `school-guardian` / `juan-chala-123`)
- Variables de entorno configuradas en cada componente (.env)

## 4. Despliegue del Backend en AWS EC2

### 4.1 Preparación de la instancia

Se configuró swap para soportar builds/imágenes Docker:

```bash
sudo fallocate -l 4G /var/lib/docker/swapfile
sudo chmod 600 /var/lib/docker/swapfile
sudo mkswap /var/lib/docker/swapfile
sudo swapon /var/lib/docker/swapfile
```

Verificación: `free -h`, `swapon --show`

### 4.2 Estructura del código desplegado

El código backend fue copiado desde el árbol `develop` local hacia el servidor en:
`/home/admin/projects/school-guardian/backend/`

Microservicios desplegados: `ms-iam`, `ms-user-management`, `ms-school-management`, `ms-fleet`, `ms-route`, `ms-notification`, `ms-forgot-information`, `ms-api-gateway`, `sg-ms-gps`.

**Importante:** `sg-ms-gps` es el único directorio git con repositorio en el servidor (rama `develop`). Los demás microservicios fueron desplegados desde copia de `develop`.

### 4.3 Configuración de `.env` para servicios

Para cada microservicio se creó/ajustó su archivo `.env` apuntando a RDS SQL Server:

- `ms-iam/.env`: `MSSQL_SA_PASSWORD`, `ConnectionStrings__DefaultConnection` hacia RDS
- Resto de servicios `.env`: configuración de conexión a RDS, Kafka (`kafka:9092`), etc.
- `sg-ms-gps/.env`: configuración específica del servicio GPS (TCP 8842, REST HTTP interno)

### 4.4 Levantado de stack Docker

Se utilizó Docker Compose multi-archivo:

```bash
# Desde /home/admin/projects/school-guardian/backend
docker compose up -d kafka zookeeper
docker compose up -d ms-iam ms-user-management ms-school-management ms-fleet ms-route ms-notification ms-forgot-information ms-gps
docker compose -f docker-compose.gateway.yml up -d kong
```

Creación de red: `sg-services-network`.

Creación manual de topic Kafka: `student.scanned` (verificado operativo).

## 5. Base de Datos (RDS SQL Server)

### 5.1 Estado inicial

- RDS: `school-guardian.choygiky8hou.us-east-2.rds.amazonaws.com:1433`, DB `school-guardian`, usuario `sgadmin`
- Migraciones Liquibase aplicadas al 100%: 8 esquemas, 133 changesets, 32 tablas
- `Iam.TermsAcceptance` con 101 registros

### 5.2 Correcciones aplicadas vía ALTER TABLE (idempotentes)

Debido a que el código (tanto `main` como `develop`) esperaba columnas que no existían en las migraciones aplicadas, se agregaron las columnas faltantes de forma no destructiva usando `sqlcmd` vía Docker:

```sql
-- UserManagement.Person
ALTER TABLE [UserManagement].[Person] ADD [CityId] UNIQUEIDENTIFIER NULL;
ALTER TABLE [UserManagement].[Person] ADD [SchoolId] UNIQUEIDENTIFIER NULL;

-- School.School
ALTER TABLE [School].[School] ADD [Latitude] DECIMAL(18,6) NULL;
ALTER TABLE [School].[School] ADD [Longitude] DECIMAL(18,6) NULL;

-- School.SchoolCampus
ALTER TABLE [School].[SchoolCampus] ADD [Latitude] DECIMAL(18,6) NULL;
ALTER TABLE [School].[SchoolCampus] ADD [Longitude] DECIMAL(18,6) NULL;

-- School.School.Phone (ajuste tipo para compatibilidad con entidad)
ALTER TABLE [School].[School] ALTER COLUMN [Phone] BIGINT NOT NULL;
```

Estos cambios fueron aplicados idempotentemente (verificando `COL_LENGTH`/tipo antes de alterar).

## 6. API Gateway (Kong) + Reverse Proxy (Caddy)

### 6.1 Kong (DB-less declarativo)

Archivo de configuración: `ms-api-gateway/kong/kong.yml`

#### Correcciones aplicadas

**1) Rutas `/user/*`** — El SPA llamaba a `/user/api/drivers` (sin `v1`), pero `ms-user-management` expone rutas con `api/v1/*`. Se ajustó la ruta de Kong:

```yaml
- name: ms-user-management
  url: http://ms-user-management:8080/api/v1
  routes:
    - name: user-management-route
      paths:
        - /user/api
      methods: [GET, POST, PUT, PATCH, DELETE, OPTIONS]
      strip_path: true
```

Respaldado como `kong.yml.bak-pre-user-v1`.

**2) Ruta GPS REST** — Expuesta sin JWT para que el móvil pueda consumirla con `fetch()` directo:

```yaml
- name: ms-gps
  url: http://ms-gps:8080
  routes:
    - name: gps-route
      paths:
        - /api/gps
      methods: [GET, POST, OPTIONS]
      strip_path: false
```

**3) CORS** — Se ajustó `notification-route` para incluir `OPTIONS` en `methods` (prevención de preflight 404). Backup: `kong.yml.bak-pre-notif`.

Tras modificaciones: `docker restart kong` (o `kong reload`).

### 6.2 Caddy (única entrada pública)

Archivo: `deploy/Caddyfile.gateway`

Configurado como único punto público expuesto (443/80). Certificado Let's Encrypt automático para `3-15-21-147.sslip.io`.

Caddy escucha 443 y hace reverse proxy hacia Kong (interno). Puerto 8842 **NO** pasa por Caddy (TCP directo para VT03F).

## 7. Servicio GPS (ms-gps)

### 7.1 Características

- Tecnología: Spring Boot (Java)
- TCP Server: escucha en `0.0.0.0:8842` (protocolo VT03F)
- REST API: expuesto internamente en `8080` → reenviado vía Kong en `/api/gps`
- Persistencia: **in-memory** (no requiere BD)
- Endpoints REST válidos:
  - `GET /api/gps/devices/{imei}` — obtener dispositivo
  - `GET /api/gps/devices/{imei}/connected` — estado conexión (true/false)
  - `GET /api/gps/devices/{imei}/location` — última ubicación
  - `POST /api/gps/test/{imei}` — test interno

**Nota:** No existe `GET /api/gps/connected` global. Se debe consultar por IMEI.

### 7.2 Configuración del tracker VT03F

El dispositivo VT03F debe apuntar **directamente** a EC2 por TCP puro (sin túnel Localtonet):

```
Servidor (IP/Dominio): 3.15.21.147
Puerto:                8842
Protocolo:             TCP
```

Comando SMS de ejemplo utilizado (formato VT03F):

```text
SERVER,1,3.15.21.147,8842,0#
```

Verificación cuando conecte:

```bash
# Logs del servidor
ssh -i ~/Descargas/sg-ssh.pem admin@ec2-3-15-21-147.us-east-2.compute.amazonaws.com
docker logs -f ms-gps
# Esperar: "GPS conectado desde: /XX.XX.XX.XX:XXXX"

# Desde internet
curl https://3-15-21-147.sslip.io/api/gps/devices/355468590730586/connected
# Debe pasar de false → true cuando el tracker conecte
```

## 8. Despliegue Web (Frontend Angular) en Vercel

### 8.1 Build/Deploy

Proyecto: `dvlp-front/dvlp-web` (Angular)
Build command: `npx ng build --configuration spa`
Output: `dist/school-guardian/browser`
Framework: null (configurado en `vercel.json`)

`vercel.json`:
```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": null,
  "installCommand": "npm ci",
  "buildCommand": "npx ng build --configuration spa",
  "outputDirectory": "dist/school-guardian/browser",
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

### 8.2 Variables de entorno Vercel

Se configuraron las variables de entorno en Vercel (preview/production según caso):

- `API_URL`: `https://3-15-21-147.sslip.io`
- `API_FORGOT_INFORMATION_URL`: `https://3-15-21-147.sslip.io`

Tras configurar variables se realizó redeploy para que el bundle inyectara los valores correctos.

Verificación del bundle desplegado:
```bash
curl -s https://guardian-escolar.vercel.app/main.js | grep -oE 'https://3-15-21-147.sslip.io' | head -1
# Debe retornar la URL del EC2
```

### 8.3 Solución a "pantalla blanca / texto crudo"

Causas corregidas:
1. **Kong routing**: `/user/api/*` → ahora mapeado correctamente a `/api/v1/*` del `ms-user-management`
2. **Faltantes en BD**: se agregaron las 6 columnas + ajuste `Phone` BIGINT en `School.School`
3. **Vercel env vars**: apuntando al backend correcto

Con estas correcciones, todos los endpoints principales retornan 200 vía gateway.

## 9. Aplicación Móvil (React Native + Expo)

### 9.1 Configuración

Proyecto: `dvlp-front/dvlp-movil/guardian-escolar`

Archivos relevantes:
- `app.json` / `eas.json`
- `.env` (gitignored) con variables `EXPO_PUBLIC_*`
- `src/core/config/gps.js` → `baseUrl: 'https://3-15-21-147.sslip.io'`

### 9.2 Variables de entorno en EAS

Se cargaron las variables `EXPO_PUBLIC_*` en el entorno `preview` de EAS Build (Android/iOS), entre ellas:
- `EXPO_PUBLIC_API_URL=https://3-15-21-147.sslip.io`
- `EXPO_PUBLIC_FORGOT_INFORMATION_API_URL=https://3-15-21-147.sslip.io`
- `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY=...`
- Variables de Notification API

Verificación: `eas env:list --environment preview`

### 9.3 Compilación Android (.APK)

Perfil `preview` configurado con `android.buildType: "apk"`:

```bash
cd /home/ton1/Documentos/projects/scholl-guardian-project/dvlp-front/dvlp-movil/guardian-escolar
npx --yes eas-cli@latest build -p android --profile preview --non-interactive
```

Resultado: build exitoso, enlace de descarga/install `.APK`.

### 9.4 Compilación iOS (.IPA)

Para compilar iOS se requiere cuenta Apple Developer y credenciales. Configuración realizada:

- `app.json` → `ios.bundleIdentifier: "com.schoolguardian.app"`
- `app.json` → `ios.infoPlist.ITSAppUsesNonExemptEncryption: false`
- `eas.json` → perfil `preview` con `"ios": {}`

Comando (interactivo por primera vez):

```bash
cd /home/ton1/Documentos/projects/scholl-guardian-project/dvlp-front/dvlp-movil/guardian-escolar
npx --yes eas-cli@latest build -p ios --profile preview
```

Durante el primer build interactivo: login Apple ID → seleccionar Team → gestionar credenciales automáticamente → generar certificado/provisioning. Tras finalizar, EAS entrega enlace para instalar `.IPA` desde Safari en iPhone.

### 9.5 Desarrollo con Expo Go (túnel público)

Para desarrollo/test en tiempo real desde cualquier red:

```bash
cd /home/ton1/Documentos/projects/scholl-guardian-project/dvlp-front/dvlp-movil/guardian-escolar
npx expo start --go --tunnel
```

Genera URL tipo: `exp://<subdominio>-<usuario>-8081.exp.direct`
Abrir en Expo Go (Android/iOS) → Enter URL manually → pegar la URL.

**Nota:** El APK compilado con `preview` es una build standalone (no depende del túnel Expo). Para desarrollo con hot reload se usa el túnel.

## 10. Verificación Post-Despliegue

### 10.1 Endpoints vía Kong/Caddy (sweep completo)

Con token admin (`admin@school-guardian.com` / `99000000001`):

| Endpoint | Resultado |
|---|---|
| `POST /api/v1/auth/login` | 200 |
| `GET /api/v1/auth/profile` | 200 |
| `GET /user/api/drivers` | 200 |
| `GET /user/api/students` | 200 |
| `GET /user/api/families` | 200 |
| `GET /user/api/parents` | 200 |
| `GET /user/api/admins` | 200 |
| `GET /fleet/api/buses` | 200 |
| `GET /fleet/api/gps-devices` | 200 |
| `GET /fleet/api/vehicle-types/brands` | 200 |
| `GET /fleet/api/vehicle-types/models?brandId=1` | 200 |
| `GET /route/api/cities` | 200 |
| `GET /route/api/routes` | 200 |
| `GET /route/api/stops` | 200 |
| `GET /school-management/api/v1/schools` | 200 |
| `GET /school-management/api/v1/schools/{id}/campuses` | 200 |
| `GET /api/gps/devices/{imei}/connected` | 200 (false hasta conectar tracker) |
| `GET /api/gps/devices/{imei}/location` | 200 |

### 10.2 Estados de contenedores

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
# Todos: kafka, zookeeper, ms-*, kong, caddy → Up/healthy
```

### 10.3 CORS

Verificado preflight `OPTIONS` hacia `https://guardian-escolar.vercel.app` retorna 200 con headers `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`.

## 11. Troubleshooting

| Problema | Causa | Solución |
|---|---|---|
| Web carga login pero resto vacío / texto crudo | Kong `/user/*` mal mapeado + columnas BD faltantes | Aplicar correcciones Kong (route `/user/api` + service path `/api/v1`) y ALTER TABLEs |
| `schools` / `campuses` 500 | `School.Phone` tipo INT vs entidad long; columnas Latitude/Longitude faltantes | `ALTER TABLE School.School ALTER COLUMN Phone BIGINT NOT NULL` + agregar columnas faltantes |
| GPS `/connected` global 404 | Endpoint no existe en ms-gps | Usar `/devices/{imei}/connected` |
| Tracker no conecta | Apunta a Localtonet/túnel viejo | Reconfigurar VT03F a `3.15.21.147:8842` (TCP) vía SMS/comando `SERVER,1,3.15.21.147,8842,0#` |
| `/user/*` 404 tras cambios | Backup/restauración de kong.yml incorrecta | Verificar `url: http://ms-user-management:8080/api/v1` y `paths: [/user/api]` con `strip_path: true` |
| iOS build falla por credenciales | Primera vez requiere login Apple interactivo | Ejecutar sin `--non-interactive`, responder login Apple → Team → gestionar credenciales automáticamente |
| Expo tunnel no abre en APK | APK es build standalone, no apunta a túnel Expo | Usar APK compilado (preview) o Expo Go + túnel para desarrollo |

## 12. Anexos

- **EC2 IP pública:** `3.15.21.147`
- **Dominio público:** `https://3-15-21-147.sslip.io`
- **Vercel Web:** `https://guardian-escolar.vercel.app`
- **IMEI GPS:** `355468590730586`
- **RDS Host:** `school-guardian.choygiky8hou.us-east-2.rds.amazonaws.com:1433`
- **Backups relevantes:** `kong.yml.bak-pre-user-v1`, `kong.yml.bak-pre-notif`
