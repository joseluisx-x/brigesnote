# 🏫 Bridges Note - MINEDU ESCALE Bridge API

Microservicio y API REST en **Laravel 13** (PHP 8.3+) diseñado para sincronizar, estructurar y servir el catálogo oficial de instituciones educativas (colegios) y la división política/geográfica (regiones, provincias y distritos) del Perú, consumiendo directamente la API pública oficial de **MINEDU / ESCALE** (*Estadística de la Calidad Educativa*).

---

## 📋 Tabla de Contenidos

1. [Propósito del Proyecto](#-propósito-del-proyecto)
2. [Arquitectura y Flujo de Datos](#-arquitectura-y-flujo-de-datos)
3. [Requisitos del Sistema](#-requisitos-del-sistema)
4. [Instalación y Puesta en Marcha](#-instalación-y-puesta-en-marcha)
5. [Configuración de Entorno (.env)](#-configuración-de-entorno-env)
6. [Sincronización de Datos (Consola Artisan)](#-sincronización-de-datos-consola-artisan)
7. [Documentación de la API REST](#-documentación-de-la-api-rest)
8. [Estructura del Proyecto](#-estructura-del-proyecto)
9. [Pruebas](#-pruebas)
10. [Solución de Problemas Frecuentes](#-solución-de-problemas-frecuentes)

---

## 🎯 Propósito del Proyecto

Servir como un **puente de datos optimizado** para aplicaciones cliente (sistemas de admisión universitaria, plataformas de matrícula escolar, intranets, etc.), permitiéndoles autocompletar ubigeos y consultar colegios mediante consultas locales ultrarrápidas, sin depender de la latencia o disponibilidad de los servidores estatales.

---

## 🔄 Arquitectura y Flujo de Datos

```
[ API MINEDU / ESCALE ] 
 (escale.minedu.gob.pe)
          │
          ▼  (Comando: php artisan sync:locations)
[ Base de Datos Local MySQL (escalebridge) ]
  ├── regions
  ├── provinces
  ├── districts
  └── schools (con ubigeo / idDistrito)
          │
          ▼  (LocationController)
[ Endpoints API REST Local / Producción ]
  ├── GET /api/regiones
  ├── GET /api/provincias/{idRegion}
  ├── GET /api/distritos/{idProvince}
  └── GET /api/colegios/{ubigeo}
          │
          ▼
[ Clientes / Aplicaciones Externas ] (ej. admisionweb)
```

Las rutas de la API se registran en `bootstrap/app.php` (Laravel 13) y se cargan automáticamente con el prefijo `/api`.

---

## 💻 Requisitos del Sistema

- **PHP:** `8.3` – `8.5`
- **Laravel Framework:** `13.x`
- **Gestor de Paquetes:** Composer 2.x
- **Base de Datos:** MySQL 5.7+ / 8.0+ o MariaDB 10.3+
- **Extensiones PHP requeridas:** `pdo_mysql`, `mbstring`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `fileinfo`, `curl`
- **Servidor Web Local:** Laragon, XAMPP o `php artisan serve`

> ⚠️ **Importante:** el servidor web debe apuntar su `DocumentRoot` a la carpeta `public/`. No expongas la raíz del proyecto.

---

## 🚀 Instalación y Puesta en Marcha

### 1. Clonar el repositorio
```bash
git clone https://github.com/hubelsolis/brigesnote.git
cd brigesnote
```

### 2. Instalar dependencias de Composer
```bash
composer install
```

### 3. Configurar variables de entorno
Copia el archivo de ejemplo y genera la clave de cifrado:
```bash
cp .env.example .env
php artisan key:generate
```

Ajusta en `.env` las credenciales de MySQL (`DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`).

### 4. Crear la Base de Datos
Crea una base de datos vacía en tu gestor MySQL (ej. HeidiSQL o phpMyAdmin):
- **Nombre:** `escalebridge`
- **Cotejamiento:** `utf8mb4_unicode_ci`

### 5. Ejecutar las Migraciones
Crea las tablas en la base de datos:
```bash
php artisan migrate
```

### 6. Sincronizar el catálogo escolar
Descarga la información oficial desde MINEDU:
```bash
php artisan sync:locations
```

### 7. Iniciar el servidor
```bash
php artisan serve
```
La aplicación quedará disponible en `http://127.0.0.1:8000`.

> Si usas Laragon o Apache, configura el `DocumentRoot` del sitio en la carpeta `public/` del proyecto. Si usas Apache en Windows, utiliza la versión **Thread Safe (TS)** de PHP para el módulo de Apache.

---

## ⚙️ Configuración de Entorno (.env)

Asegúrate de tener estos valores configurados en tu archivo `.env`:

```env
APP_NAME="Bridges Note"
APP_ENV=local
APP_KEY=base64:...
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000

# Conexión MySQL
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=escalebridge
DB_USERNAME=root
DB_PASSWORD=

# Sesiones y caché en archivo (no requieren tablas adicionales)
SESSION_DRIVER=file
CACHE_STORE=file
QUEUE_CONNECTION=sync
```

### Variables nuevas de Laravel 13

La plantilla `.env.example` incluye las variables incorporadas por Laravel 13. Las más relevantes son:

| Variable | Valor por defecto | Descripción |
|---|---|---|
| `APP_LOCALE` | `en` | Idioma principal de la aplicación |
| `APP_FALLBACK_LOCALE` | `en` | Idioma de respaldo |
| `APP_FAKER_LOCALE` | `en_US` | Locale usado por Faker en pruebas y seeders |
| `APP_MAINTENANCE_DRIVER` | `file` | Driver del modo mantenimiento |
| `BCRYPT_ROUNDS` | `12` | Coste del hash de contraseñas |
| `LOG_STACK` | `single` | Canales del stack de logs |
| `LOG_DEPRECATIONS_CHANNEL` | `null` | Canal para avisos de deprecación |
| `LOG_LEVEL` | `debug` | Nivel mínimo de logging |
| `SESSION_ENCRYPT` | `false` | Cifrado de la sesión |
| `SESSION_PATH` / `SESSION_DOMAIN` | `/` / `null` | Ámbito de la cookie de sesión |
| `BROADCAST_CONNECTION` | `log` | Conexión de broadcasting |
| `FILESYSTEM_DISK` | `local` | Disco de archivos por defecto |
| `CACHE_STORE` | `file` | Reemplaza al antiguo `CACHE_DRIVER` |
| `MEMCACHED_HOST` | `127.0.0.1` | Host de Memcached |
| `REDIS_CLIENT` | `phpredis` | Cliente Redis |
| `MAIL_SCHEME` | `null` | Esquema de conexión SMTP |
| `AWS_USE_PATH_STYLE_ENDPOINT` | `false` | Endpoint estilo ruta para S3 |
| `VITE_APP_NAME` | `${APP_NAME}` | Nombre visible en el frontend |

> Si vienes de una versión anterior, ten en cuenta que Laravel 13 usa `CACHE_STORE` en lugar de `CACHE_DRIVER`.

---

## 📥 Sincronización de Datos (Consola Artisan)

El comando central del sistema es:

```bash
php artisan sync:locations
```

### ¿Cómo opera la sincronización?
1. **Regiones (`/padron/rest/regiones`):** Descarga los 25 departamentos del país con sus nombres, coordenadas (`pointX`, `pointY`) y nivel de `zoom`.
2. **Provincias (`/padron/rest/provincias?idRegion={id}`):** Por cada región, descarga sus provincias.
3. **Distritos (`/padron/rest/distritos?idProvincia={id}`):** Por cada provincia, descarga los distritos correspondientes.
4. **Colegios (`/padron/rest/instituciones?estados=1&ubigeo={idDistrito}`):** Por cada distrito, descarga las instituciones educativas activas y las almacena en la tabla `schools` vinculando el código modular (`codMod`) como llave primaria y asociando el `idDistrito`.

> ⏱️ **Nota:** La sincronización total recorre más de 1,800 distritos y 80,000 colegios, por lo que puede tomar varios minutos según la velocidad de conexión y respuesta del servidor del MINEDU.

> 🌐 **Requisito de red:** el comando necesita salida a Internet hacia `escale.minedu.gob.pe`. Si hay proxy, firewall o el servidor del MINEDU no responde, la sincronización fallará con un error de conexión.

---

## 📡 Documentación de la API REST

Todos los endpoints retornan respuestas en formato JSON bajo el prefijo `/api/`.

### 1. Listar Regiones / Departamentos
Obtiene todas las regiones del país.

- **URL:** `/api/regiones`
- **Método:** `GET`
- **Respuesta Exitosa (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "idRegion": "15",
      "nombreRegion": "LIMA",
      "pointX": -12.046374,
      "pointY": -77.042793,
      "zoom": 10
    }
  ]
}
```

---

### 2. Listar Provincias por Región
Obtiene las provincias pertenecientes a una región (mediante los 2 primeros dígitos del código de departamento).

- **URL:** `/api/provincias/{idRegion}`
- **Método:** `GET`
- **Ejemplo:** `/api/provincias/15` (Provincias de Lima)
- **Respuesta Exitosa (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "idProvincia": "1501",
      "nombreProvincia": "LIMA",
      "pointX": -12.046374,
      "pointY": -77.042793
    }
  ]
}
```

- **Sin resultados (404 Not Found):**
```json
{
  "status": "error",
  "message": "No se encontraron provincias para esta región."
}
```

---

### 3. Listar Distritos por Provincia
Obtiene los distritos de una provincia (mediante los 4 dígitos de ubigeo provincial).

- **URL:** `/api/distritos/{idProvince}`
- **Método:** `GET`
- **Ejemplo:** `/api/distritos/1501` (Distritos de la provincia de Lima)
- **Respuesta Exitosa (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "idDistrito": "150101",
      "nombreDistrito": "LIMA"
    },
    {
      "idDistrito": "150102",
      "nombreDistrito": "ANCON"
    }
  ]
}
```

- **Sin resultados (404 Not Found):**
```json
{
  "status": "error",
  "message": "No se encontraron distritos para esta provincia."
}
```

---

### 4. Listar Colegios por Distrito (Ubigeo)
Obtiene todas las instituciones educativas registradas en un distrito según su ubigeo de 6 dígitos.

- **URL:** `/api/colegios/{ubigeo}`
- **Método:** `GET`
- **Ejemplo:** `/api/colegios/150101`
- **Respuesta Exitosa (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "codMod": "0273572",
      "idDistrito": "150101",
      "nombreCenEdu": "JUAN JIMENEZ PIMENTEL",
      "cenPob": "LIMA",
      "dirCen": "JR. LAMPA 240",
      "director": "PEDRO PEREZ",
      "dis_crecer": null,
      "dis_juntos": null
    }
  ]
}
```

- **Sin resultados (404 Not Found):**
```json
{
  "status": "error",
  "message": "No se encontraron escuelas para este distrito."
}
```

---

## 📂 Estructura del Proyecto

```
brigesnote/
├── app/
│   ├── Console/Commands/
│   │   └── SyncLocations.php       # Comando 'sync:locations' (consumo de API ESCALE)
│   ├── Http/Controllers/
│   │   └── LocationController.php  # Controlador de endpoints /api/
│   └── Models/
│       ├── District.php            # Modelo Distritos
│       ├── Province.php            # Modelo Provincias
│       ├── Region.php              # Modelo Regiones
│       └── School.php              # Modelo Colegios / Escuelas
├── bootstrap/
│   ├── app.php                     # Registro de rutas web, API, consola y middleware
│   └── providers.php
├── config/                         # Configuración del framework
├── database/migrations/
│   ├── ..._create_users_table.php
│   ├── ..._create_cache_table.php
│   ├── ..._create_jobs_table.php
│   ├── ..._create_regions_table.php
│   ├── ..._create_provinces_table.php
│   ├── ..._create_districts_table.php
│   └── ..._create_schools_table.php
├── public/
│   ├── index.php                   # Punto de entrada de la aplicación
│   └── .htaccess
├── routes/
│   ├── api.php                     # Definición de rutas REST (prefijo /api)
│   ├── console.php
│   └── web.php
├── storage/                        # Logs, caché y sesiones
├── tests/
├── artisan
├── composer.json
├── phpunit.xml
├── .env.example                    # Plantilla de variables de entorno
└── README.md
```

---

## 🧪 Pruebas

Ejecuta la suite con:

```bash
php artisan test
```

Los tests de ejemplo incluidos no requieren conexión a la base de datos.

---

## 🛠️ Solución de Problemas Frecuentes

### 1. Las rutas `/api/*` devuelven 404 o se muestra la página de bienvenida
- **Causa:** el servidor web apunta a la raíz del proyecto en lugar de `public/`.
- **Solución:** configura el `DocumentRoot` (Laragon, Apache o Nginx) en la carpeta `public/`. Con `php artisan serve` esto ya viene resuelto.

### 2. Error: `could not find driver`
- **Causa:** falta la extensión `pdo_mysql`.
- **Solución:** habilítala en `php.ini` o instala el paquete correspondiente (`php8.3-mysql` en Linux).

### 3. Error: `SQLSTATE[HY000] [2002] Connection refused`
- **Causa:** MySQL no está iniciado o `DB_HOST`/`DB_PORT` son incorrectos.
- **Solución:** inicia MySQL y revisa los valores `DB_*` en `.env`.

### 4. Error: `Base table or view not found: 1146 Table '...sessions' doesn't exist`
- **Causa:** `SESSION_DRIVER=database` sin la tabla de sesiones migrada.
- **Solución:** ejecuta `php artisan migrate` o cambia `SESSION_DRIVER=file` en `.env`.

### 5. `sync:locations` falla con `cURL error 6`, `cURL error 28` o HTTP 403
- **Causa:** no hay conectividad hacia `escale.minedu.gob.pe` (proxy, firewall, DNS o caída del servicio).
- **Solución:** verifica la salida a Internet, la configuración de proxy y reintenta más tarde. El resto de la API sigue funcionando con los datos ya sincronizados.

### 6. Error: `No application encryption key has been specified`
- **Solución:** ejecuta `php artisan key:generate`.

### 7. Error: `Class "..." not found` o cambios que no se reflejan
- **Solución:** ejecuta `composer install`, `composer dump-autoload` y `php artisan optimize:clear`.

### 8. Las migraciones chocan con tablas ya existentes
- **Causa:** la base de datos contiene tablas de una instalación anterior.
- **Solución:** usa una base de datos vacía o, si puedes descartar los datos, ejecuta `php artisan migrate:fresh` (destructivo). Haz un respaldo antes.

---

## 📄 Licencia

Este proyecto es de código abierto bajo la licencia [MIT](LICENSE).
