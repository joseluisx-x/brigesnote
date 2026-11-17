# Plan de trabajo: migración de Bridges Note a PHP 8.3

**Fecha:** 16 de noviembre de 2026
**Repositorio:** https://github.com/hubelsolis/brigesnote.git
**Rama de trabajo:** `migracion-php-8.3`

---

## 1. Estado actual

| Elemento | Valor actual |
|---|---|
| PHP | 7.4 (el proyecto no funciona en PHP 8.1 o superior) |
| Framework | Laravel 7.30.7 (fijado en `composer.lock`) |
| Código propio | 4 modelos, 1 controlador (4 endpoints), 1 comando (`sync:locations`), 4 migraciones |
| Resto del proyecto | Andamiaje por defecto de Laravel 7 |
| Estructura | Sin carpeta `public/`: `index.php`, `.htaccess` y `server.php` están en la raíz |
| Entorno | `.env` ya configurado localmente (no se versiona) |

## 2. Objetivo

Que el proyecto instale y funcione en **PHP 8.3** sobre **Laravel 13**, con la API devolviendo las mismas respuestas JSON que antes.

**Por qué Laravel 13:** Laravel 7 solo soporta hasta PHP 8.0, así que migrar de PHP implica migrar de framework. Laravel 13 soporta PHP 8.3 – 8.5 y es la versión vigente. Laravel 12 ya no recibe correcciones de errores (solo de seguridad hasta febrero de 2027). Si el docente exige otra versión, solo cambia el paso 3.

## 3. Alcance

**Incluye:** `composer.json`, `composer.lock`, `app/`, `routes/`, `database/`, `config/`, `tests/`, estructura `public/`, `README.md`, `.env.example`.

**No incluye:** cambios de lógica, de endpoints ni del formato de las respuestas JSON.

## 4. Reglas para el agente

1. **No leer, modificar ni mostrar el contenido de `.env`.** Si hace falta una variable nueva, agregarla solo a `.env.example` y avisar.
2. No cambiar la lógica de negocio ni los contratos de la API.
3. Hacer **un commit por paso**, con mensajes del tipo `chore:`, `refactor:` o `feat:`.
4. **No hacer `git push`.** El push lo hace el responsable después de revisar los cambios.
5. Si un paso falla, detenerse y reportar el error en vez de improvisar.

## 5. Pasos

### Paso 0. Línea base (manual, antes del agente)
Con PHP 7.4 y el `.env` ya configurado, comprobar que el proyecto funciona hoy:

```bash
composer install
php artisan migrate
php artisan route:list
php artisan serve
```

Probar en el navegador o con `curl` y guardar las respuestas en un archivo **fuera del repositorio**:

- `/api/regiones`
- `/api/provincias/15`
- `/api/distritos/1501`
- `/api/colegios/150101`

Si la base está vacía, ejecutar `php artisan sync:locations` una vez (tarda varios minutos).

### Paso 1. Respaldo y base de datos de prueba (manual)
1. Verificar que `.env` aparece ignorado: `git status` **no** debe listarlo.
2. Crear una base de datos nueva y vacía (por ejemplo `escalebridge_test`, cotejamiento `utf8mb4_unicode_ci`) y apuntar `DB_DATABASE` a ella en `.env`. Así las migraciones nuevas no chocan con las tablas existentes ni con el historial de la tabla `migrations`.

### Paso 2. Commit y rama inicial (manual)
```bash
git add .
git commit -m "chore: estado inicial antes de migración a PHP 8.3 + plan de trabajo"
git checkout -b migracion-php-8.3
git push -u origin migracion-php-8.3
```

### Paso 3. Entorno PHP 8.3
Cambiar PHP a 8.3 en el entorno local (Laragon u otro) y verificar con `php -v`.

### Paso 4. Nuevo esqueleto Laravel 13
Crear un proyecto Laravel 13 en una carpeta temporal y usarlo como base:

```bash
composer create-project laravel/laravel temp-l13
```

Reemplazar la estructura del proyecto por la del esqueleto nuevo (incluida la carpeta `public/`, con `index.php` y `.htaccess` dentro), **conservando** el historial de Git, el `.env`, el `.gitignore` y el `README.md`. Eliminar archivos que ya no existen en versiones nuevas (`server.php`, `web.config` de la raíz, `app/Http/Kernel.php`, etc.).

### Paso 5. Portar el código propio
Copiar al esqueleto nuevo, sin cambiar su lógica:

| Origen | Observación |
|---|---|
| `app/Models/{Region,Province,District,School}.php` | Quitar el `use HasFactory` que no se usa |
| `app/Http/Controllers/LocationController.php` | Sin cambios |
| `app/Console/Commands/SyncLocations.php` | Se registra automáticamente al estar en `app/Console/Commands` |
| `routes/api.php` | Quitar la ruta `/user` con `auth:api` (viene del andamiaje y no existe ese guard). Registrar las rutas API en `bootstrap/app.php` |
| `database/migrations/2024_12_22_*` (regions, provinces, districts, schools) | Conservar; las migraciones por defecto (users, password_resets, failed_jobs) se reemplazan por las del esqueleto nuevo |

Eliminar `app/Region.php` (modelo vacío y duplicado de `app/Models/Region.php`).

### Paso 6. Dependencias y composer.json
- Confirmar `"php": "^8.3"`.
- Verificar que `fideloper/proxy`, `fruitcake/laravel-cors` y `facade/ignition` ya no estén (el framework nuevo los incluye o los reemplaza).
- Ejecutar `composer install` y subir el nuevo `composer.lock`.

### Paso 7. Verificación
```bash
php -v
php artisan migrate
php artisan route:list
php artisan serve
php artisan test
```

- Comparar las 4 respuestas con las guardadas en el paso 0 (mismo formato JSON).
- Ejecutar `php artisan sync:locations` una vez y confirmar que termina sin errores.
- Configurar la raíz del servidor web en la carpeta `public/` (en Laragon, el `DocumentRoot`).

### Paso 8. Documentación
Actualizar `README.md`: requisitos (PHP 8.3, Laravel 13), estructura del proyecto y solución de problemas. Eliminar la sección del error de `ArrayAccess`, que ya no aplica. Documentar en `.env.example` cualquier variable nueva.

### Paso 9. Revisión y entrega (manual)
```bash
git status        # .env NO debe aparecer
git diff main
git push
```

## 6. Criterios de éxito

- [ ] `composer install` termina sin errores en PHP 8.3
- [ ] `php artisan route:list` muestra las 4 rutas `/api/*`
- [ ] Los 4 endpoints devuelven el mismo JSON que en la línea base
- [ ] `php artisan sync:locations` se ejecuta sin errores
- [ ] `php artisan test` pasa
- [ ] `.env` no está en el repositorio
- [ ] `README.md` actualizado

## 7. Riesgos

| Riesgo | Mitigación |
|---|---|
| Estructura sin `public/` | Se corrige en el paso 4 y se verifica en el paso 7 |
| Choque con tablas ya existentes en la base | Usar una base de datos de prueba (paso 1) |
| `whereRaw('LEFT(...)')` es específico de MySQL | No usar SQLite para las pruebas; usar MySQL |
| Cambios del agente difíciles de revisar | Un commit por paso y rama separada de `main` |
| Pérdida de configuración o secretos | El agente no toca `.env` (regla 1) |
| Sincronización larga o con fallas del servidor del MINEDU | Ejecutarla una sola vez y revisar errores |
