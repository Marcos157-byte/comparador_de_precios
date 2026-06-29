# Kache — Backend API

Comparador de precios de supermercados en Ecuador. Backend REST construido con Django y PostgreSQL que expone los precios de Supermaxi, Coral y otras cadenas para que el usuario pueda comparar dónde le conviene comprar cada producto.

> Proyecto académico universitario — los scrapers respetan `robots.txt` y usan un `User-Agent` identificado.

---

## Stack tecnológico

| Componente | Versión |
|---|---|
| Python | 3.11 |
| Django | 5.1 |
| Django REST Framework | 3.15 |
| Base de datos | PostgreSQL |
| Driver PostgreSQL | psycopg2-binary 2.9 |
| Auth | djangorestframework-simplejwt 5.3 |
| Scraping | Playwright 1.45 (Chromium headless) |
| CORS | django-cors-headers 4.4 |
| Filtros | django-filter 24.3 |
| Variables de entorno | python-decouple 3.8 |
| Gestor de paquetes | [uv](https://docs.astral.sh/uv/) |

---

## Estructura del proyecto

```
kache-backend/
├── config/
│   ├── settings.py          # Configuración central de Django
│   ├── urls.py              # Router raíz — une todos los prefijos /api/
│   └── wsgi.py
├── apps/
│   ├── users/               # Usuarios y autenticación
│   ├── emails/              # Envío de correos masivos/individuales
│   ├── catalogo/            # Productos, marcas, categorías, etiquetas
│   ├── comercios/           # Ciudades, comercios, sucursales
│   ├── precios/             # Precios actuales e historial
│   ├── comparador/          # Listas de comparación del usuario
│   ├── extras/              # Perfil, publicidad, favoritos, alertas, notificaciones, reportes, reseñas
│   └── scraper/             # Management commands para poblar la BD
│       └── management/commands/
│           ├── scrapear_supermaxi.py
│           ├── scrapear_coral.py
│           └── unificar_productos_demo.py
├── pyproject.toml
└── .env
```

### Qué hace cada app

| App | Responsabilidad |
|---|---|
| `users` | Modelo `User` personalizado, auth JWT, historial de búsquedas del usuario |
| `emails` | Endpoint para enviar correos vía Gmail SMTP (admin) |
| `catalogo` | Catálogo maestro de productos con marca, categoría y etiquetas; endpoint de comparación |
| `comercios` | Ciudades, cadenas comerciales y sucursales físicas |
| `precios` | Precio actual de cada producto en cada comercio + historial automático de cambios |
| `comparador` | Listas de compra del usuario que agrupa ítems por comercio |
| `extras` | Perfil extendido, banners de publicidad, favoritos, alertas de precio, notificaciones internas, reportes de error y reseñas de comercios |
| `scraper` | Scrapers de Supermaxi y Coral (Playwright), y comando de unificación de productos duplicados |

---

## Modelos (tablas en BD)

El esquema tiene 21 tablas personalizadas:

| # | Modelo | App |
|---|---|---|
| 1 | `User` | users |
| 2 | `BusquedaReciente` | users |
| 3 | `Marca` | catalogo |
| 4 | `Etiqueta` | catalogo |
| 5 | `Categoria` | catalogo |
| 6 | `Producto` | catalogo |
| 7 | `Producto_etiquetas` (M2M automática) | catalogo |
| 8 | `Ciudad` | comercios |
| 9 | `Comercio` | comercios |
| 10 | `Sucursal` | comercios |
| 11 | `Precio` | precios |
| 12 | `HistorialPrecio` | precios |
| 13 | `ListaComparacion` | comparador |
| 14 | `ItemComparacion` | comparador |
| 15 | `PerfilUsuario` | extras |
| 16 | `Publicidad` | extras |
| 17 | `Favorito` | extras |
| 18 | `AlertaPrecio` | extras |
| 19 | `Notificacion` | extras |
| 20 | `ReporteProducto` | extras |
| 21 | `Resena` | extras |

---

## Requisitos previos

- Python 3.11
- PostgreSQL 14+
- [uv](https://docs.astral.sh/uv/getting-started/installation/) instalado globalmente
- Chromium (para los scrapers — se instala con el comando de abajo)

---

## Instalación

```bash
# 1. Clonar el repositorio
git clone <repo-url>
cd kache-backend

# 2. Instalar dependencias (uv crea el venv automáticamente)
uv sync

# 3. Instalar Chromium para Playwright
uv run playwright install chromium

# 4. Crear la base de datos en PostgreSQL
# (conectarse a psql y ejecutar:)
# CREATE DATABASE kache_db;

# 5. Configurar variables de entorno (ver sección .env)
# Crear el archivo .env en la raíz del proyecto

# 6. Migrar la base de datos
uv run manage.py migrate

# 7. Crear superusuario
uv run manage.py createsuperuser

# 8. Iniciar el servidor de desarrollo
uv run manage.py runserver
```

---

## Variables de entorno (`.env`)

Crear un archivo `.env` en la raíz del proyecto con las siguientes variables:

```env
# ── Django ─────────────────────────────────────────────────────
SECRET_KEY=cambia-esto-por-una-clave-larga-y-aleatoria
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# ── PostgreSQL ─────────────────────────────────────────────────
DB_NAME=kache_db
DB_USER=postgres
DB_PASSWORD=tu_password
DB_HOST=localhost
DB_PORT=5432
# Solo en producción con DigitalOcean Managed DB:
# DB_SSLMODE=require

# ── Email (Gmail SMTP) ─────────────────────────────────────────
EMAIL_HOST_USER=tu_correo@gmail.com
EMAIL_HOST_PASSWORD=xxxx xxxx xxxx xxxx
DEFAULT_FROM_EMAIL=tu_correo@gmail.com
```

| Variable | Descripción |
|---|---|
| `SECRET_KEY` | Clave secreta de Django. En producción debe ser larga y aleatoria. |
| `DEBUG` | `True` en desarrollo, `False` en producción. |
| `ALLOWED_HOSTS` | Lista separada por comas de hosts permitidos. |
| `DB_NAME` | Nombre de la base de datos PostgreSQL. |
| `DB_USER` | Usuario de PostgreSQL. |
| `DB_PASSWORD` | Contraseña de PostgreSQL. |
| `DB_HOST` | Host de PostgreSQL (`localhost` por defecto). |
| `DB_PORT` | Puerto de PostgreSQL (`5432` por defecto). |
| `DB_SSLMODE` | Solo necesario en DigitalOcean Managed DB: poner `require`. |
| `EMAIL_HOST_USER` | Correo Gmail desde el que se envían notificaciones. |
| `EMAIL_HOST_PASSWORD` | Contraseña de aplicación de Gmail (no la contraseña normal). |
| `DEFAULT_FROM_EMAIL` | Dirección que aparece en el campo "De:" de los correos. |

---

## Arrancar el proyecto

```bash
uv run manage.py runserver
```

- API disponible en: `http://127.0.0.1:8000/`
- Admin de Django en: `http://127.0.0.1:8000/admin/`

---

## Autenticación (flujo JWT)

Todos los endpoints protegidos requieren el header:

```
Authorization: Bearer <access_token>
```

**Flujo completo:**

```
1. POST /api/auth/login/          → devuelve access + refresh
2. Incluir en header: Authorization: Bearer <access>
3. Cuando el access expira (30 min):
   POST /api/auth/token/refresh/  → devuelve nuevo access y refresh
4. Para cerrar sesión:
   POST /api/auth/logout/         → invalida el refresh token
```

Tiempos de vida configurados en `settings.py`:
- **Access token:** 30 minutos
- **Refresh token:** 7 días (se rota en cada uso)

---

## Endpoints de la API

La base de todas las rutas es `http://localhost:8000`.

**Permisos:**
- **Pública** — sin autenticación
- **Auth** — requiere `Authorization: Bearer <token>`
- **Admin** — requiere usuario con `is_staff=True`

---

### Autenticación — `/api/auth/`

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| POST | `/api/auth/login/` | Pública | Login con `username` y `password`. Devuelve `access`, `refresh`, `user_id`, `username`, `email`, `is_staff`. |
| POST | `/api/auth/register/` | Pública | Registro con `username`, `email`, `password`, `password2`. Devuelve los mismos campos que login. |
| POST | `/api/auth/logout/` | Auth | Invalida el refresh token. Body: `{ "refresh": "..." }`. Responde 204. |
| POST | `/api/auth/token/refresh/` | Pública | Renueva el access token. Body: `{ "refresh": "..." }`. Devuelve nuevo `access` y `refresh`. |
| POST | `/api/auth/password-reset/` | Pública | Envía email de reset. Body: `{ "email": "..." }`. Responde 200 aunque el email no exista. |
| POST | `/api/auth/password-reset/confirm/` | Pública | Confirma el reset. Body: `{ "uid", "token", "new_password", "new_password2" }`. |

---

### Usuarios — `/api/users/`

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/users/` | Admin | Lista usuarios. Filtros: `?search=`, `?is_staff=true`, `?is_active=false`. |
| POST | `/api/users/` | Admin | Crea usuario. Body: `username`, `email`, `password`, `first_name`, `last_name`, `is_staff`, `is_active`. |
| GET | `/api/users/profile/` | Auth | Datos del usuario autenticado. |
| GET | `/api/users/stats/` | Admin | Estadísticas: `{ total, active, inactive, staff }`. |
| GET | `/api/users/busquedas/` | Auth | Últimas 20 búsquedas del usuario. |
| POST | `/api/users/busquedas/` | Auth | Registra una búsqueda. Body: `{ "termino": "leche", "num_resultados": 5 }`. |
| DELETE | `/api/users/busquedas/limpiar/` | Auth | Borra todo el historial de búsquedas del usuario. Responde 204. |
| GET | `/api/users/<pk>/` | Admin | Detalle de un usuario. |
| PUT/PATCH | `/api/users/<pk>/` | Admin | Actualiza un usuario. |
| DELETE | `/api/users/<pk>/` | Admin | Elimina un usuario. |
| POST | `/api/users/<pk>/toggle-active/` | Admin | Activa o desactiva un usuario. Devuelve `{ message, is_active }`. |

---

### Emails — `/api/emails/`

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| POST | `/api/emails/send/` | Admin | Envía correo. Body: `{ "subject", "message", "user_id" }`. Sin `user_id` envía a todos los usuarios activos no-staff. Devuelve `{ detail, sent, failed }`. |

---

### Catálogo — `/api/kache/`

#### Marcas

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/marcas/` | Pública | Lista todas las marcas. |
| POST | `/api/kache/marcas/` | Admin | Body: `{ "nombre", "pais_origen", "sitio_web" }`. |
| GET | `/api/kache/marcas/<pk>/` | Pública | Detalle de marca. |
| PUT/PATCH | `/api/kache/marcas/<pk>/` | Admin | Actualiza una marca. |
| DELETE | `/api/kache/marcas/<pk>/` | Admin | Elimina una marca. |

#### Etiquetas

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/etiquetas/` | Pública | Lista etiquetas (ej. "sin gluten", "importado"). |
| POST | `/api/kache/etiquetas/` | Admin | Body: `{ "nombre", "descripcion" }`. |
| GET | `/api/kache/etiquetas/<pk>/` | Pública | Detalle de etiqueta. |
| PUT/PATCH | `/api/kache/etiquetas/<pk>/` | Admin | Actualiza etiqueta. |
| DELETE | `/api/kache/etiquetas/<pk>/` | Admin | Elimina etiqueta. |

#### Categorías

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/categorias/` | Pública | Lista categorías con soporte de jerarquía padre/hijo. |
| POST | `/api/kache/categorias/` | Admin | Body: `{ "nombre", "descripcion", "id_categoria_padre" }`. |
| GET | `/api/kache/categorias/<pk>/` | Pública | Detalle de categoría. |
| PUT/PATCH | `/api/kache/categorias/<pk>/` | Admin | Actualiza categoría. |
| DELETE | `/api/kache/categorias/<pk>/` | Admin | Elimina categoría. |

#### Productos

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/productos/` | Pública | Lista productos. Filtros: `?buscar=`, `?categoria=`, `?id_categoria=`, `?marca=`, `?id_marca=`, `?etiqueta=`, `?tipo=`. Paginado a 20 por página. |
| POST | `/api/kache/productos/` | Admin | Body: `{ "nombre", "unidad_medida", "codigo_barras", "descripcion", "id_categoria", "id_marca", "etiquetas": [1, 2] }`. |
| **GET** | **`/api/kache/productos/<pk>/comparar/`** | **Pública** | **Endpoint principal de Kache.** Devuelve el producto con sus precios en todos los comercios, ordenados de menor a mayor. Marca el más barato con `es_mas_barato: true`. Ver ejemplo de respuesta abajo. |
| GET | `/api/kache/productos/<pk>/` | Pública | Detalle de producto. |
| PUT/PATCH | `/api/kache/productos/<pk>/` | Admin | Actualiza producto. |
| DELETE | `/api/kache/productos/<pk>/` | Admin | Elimina producto. |

**Respuesta de ejemplo — `/api/kache/productos/12/comparar/`:**

```json
{
  "id_producto": 12,
  "nombre": "Aceite La Favorita 1L",
  "unidad_medida": "1L",
  "codigo_barras": "7861234567890",
  "mejor_precio": "3.45",
  "total_comercios": 2,
  "precios_por_comercio": [
    {
      "id_comercio": 1,
      "nombre_comercio": "Supermaxi",
      "logo_url": null,
      "precio_efectivo": "3.45",
      "precio_normal": "3.99",
      "en_oferta": true,
      "es_mas_barato": true
    },
    {
      "id_comercio": 2,
      "nombre_comercio": "Coral",
      "logo_url": null,
      "precio_efectivo": "3.89",
      "precio_normal": "3.89",
      "en_oferta": false,
      "es_mas_barato": false
    }
  ]
}
```

---

### Comercios — `/api/kache/`

#### Ciudades

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/ciudades/` | Pública | Lista ciudades ordenadas por provincia y nombre. |
| POST | `/api/kache/ciudades/` | Admin | Body: `{ "nombre", "provincia" }`. |
| GET | `/api/kache/ciudades/<pk>/` | Pública | Detalle de ciudad. |
| PUT/PATCH | `/api/kache/ciudades/<pk>/` | Admin | Actualiza ciudad. |
| DELETE | `/api/kache/ciudades/<pk>/` | Admin | Elimina ciudad. |

#### Comercios

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/comercios/` | Pública | Lista comercios. Filtros: `?tipo=supermercado\|farmacia\|ferreteria`, `?activo=true`. Los comercios destacados aparecen primero. |
| POST | `/api/kache/comercios/` | Admin | Body: `{ "nombre", "tipo", "logo_url", "sitio_web", "activo", "destacado", "fecha_fin_destacado" }`. |
| GET | `/api/kache/comercios/<pk>/` | Pública | Detalle de comercio. |
| PUT/PATCH | `/api/kache/comercios/<pk>/` | Admin | Actualiza comercio. |
| DELETE | `/api/kache/comercios/<pk>/` | Admin | Elimina comercio. |

#### Sucursales

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/sucursales/` | Pública | Lista sucursales. Filtros: `?comercio=`, `?id_comercio=`, `?ciudad=` (ID numérico o nombre parcial). |
| POST | `/api/kache/sucursales/` | Admin | Body: `{ "id_comercio", "nombre_sucursal", "id_ciudad", "direccion", "activo" }`. |
| GET | `/api/kache/sucursales/<pk>/` | Pública | Detalle de sucursal. |
| PUT/PATCH | `/api/kache/sucursales/<pk>/` | Admin | Actualiza sucursal. |
| DELETE | `/api/kache/sucursales/<pk>/` | Admin | Elimina sucursal. |

---

### Precios — `/api/kache/`

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/precios/` | Pública | Lista precios ordenados por `precio_actual` ascendente. Filtros: `?producto=`, `?id_producto=`, `?comercio=`, `?id_comercio=`, `?tipo=`, `?en_oferta=true`. |
| POST | `/api/kache/precios/` | Admin | Body: `{ "id_producto", "id_comercio", "precio_actual", "precio_oferta", "en_oferta" }`. |
| GET | `/api/kache/precios/<pk>/` | Pública | Detalle de precio. Incluye `precio_efectivo` (oferta si activa, si no el normal). |
| PUT/PATCH | `/api/kache/precios/<pk>/` | Admin | Actualiza precio. Si `precio_actual` cambia, se guarda automáticamente un snapshot en `HistorialPrecio` via señal Django. |
| DELETE | `/api/kache/precios/<pk>/` | Admin | Elimina precio. |
| GET | `/api/kache/historial-precios/` | Pública | Historial de cambios de precio. Filtros: `?producto=`, `?id_producto=`, `?comercio=`, `?id_comercio=`. Solo lectura — se genera automáticamente. |

---

### Comparador (Listas de compra) — `/api/kache/`

Todos los endpoints requieren usuario autenticado. Cada usuario solo ve y modifica sus propias listas.

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/listas-comparacion/` | Auth | Lista las listas de comparación del usuario. |
| POST | `/api/kache/listas-comparacion/` | Auth | Crea una lista. Body: `{ "nombre": "Mi lista del mes" }`. |
| GET | `/api/kache/listas-comparacion/<pk>/` | Auth | Detalle con todos sus ítems. |
| DELETE | `/api/kache/listas-comparacion/<pk>/` | Auth | Elimina la lista y todos sus ítems. |
| POST | `/api/kache/listas-comparacion/<lista_pk>/items/` | Auth | Agrega un producto a la lista. Body: `{ "id_producto": 5, "id_comercio": 1 }`. Captura el `precio_efectivo` actual como `precio_momento`. Si el producto ya estaba, actualiza el comercio. |
| DELETE | `/api/kache/items-comparacion/<pk>/` | Auth | Quita un ítem de la lista. |

---

### Extras — `/api/kache/`

#### Perfil extendido

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/perfil/` | Auth | Perfil extendido del usuario (se crea automáticamente si no existe). Campos: `telefono`, `ciudad`, `foto_url`. |
| PUT/PATCH | `/api/kache/perfil/` | Auth | Actualiza el perfil. |

#### Publicidad

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/publicidad/` | Pública | Lista los banners con `activo=True` y dentro del rango de fechas vigentes. |

#### Favoritos

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/favoritos/` | Auth | Lista los productos marcados como favoritos. |
| POST | `/api/kache/favoritos/` | Auth | Agrega un favorito. Body: `{ "producto": 5 }`. |
| DELETE | `/api/kache/favoritos/<pk>/` | Auth | Quita un favorito. |

#### Alertas de precio

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/alertas-precio/` | Auth | Lista alertas del usuario. |
| POST | `/api/kache/alertas-precio/` | Auth | Body: `{ "producto": 5, "comercio": 1, "precio_objetivo": "2.50" }`. Si `comercio` es null, la alerta aplica al precio más barato entre todos los comercios. |
| GET | `/api/kache/alertas-precio/<pk>/` | Auth | Detalle de alerta. |
| PUT/PATCH | `/api/kache/alertas-precio/<pk>/` | Auth | Actualiza alerta (ej. cambiar `precio_objetivo` o poner `activa: false`). |
| DELETE | `/api/kache/alertas-precio/<pk>/` | Auth | Elimina alerta. |

#### Notificaciones

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/notificaciones/` | Auth | Lista notificaciones del usuario (más recientes primero). |
| PATCH | `/api/kache/notificaciones/<pk>/` | Auth | Marcar como leída. Body: `{ "leida": true }`. |

#### Reportes de productos

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/reportes/` | Admin | Lista todos los reportes enviados por usuarios. |
| POST | `/api/kache/reportes/` | Auth | Body: `{ "producto": 5, "comercio": 1, "motivo": "precio_incorrecto\|no_disponible\|otro", "comentario": "..." }`. |

#### Reseñas de comercios

| Método | Ruta | Permiso | Descripción |
|---|---|---|---|
| GET | `/api/kache/resenas/` | Pública | Lista reseñas. Filtro: `?comercio=1`. |
| POST | `/api/kache/resenas/` | Auth | Body: `{ "comercio": 1, "calificacion": 4, "comentario": "..." }`. Calificación del 1 al 5. Un usuario solo puede reseñar cada comercio una vez. |

---

## Scrapers

Los scrapers usan Playwright (Chromium) para renderizar JavaScript. Se ejecutan **manualmente** — no hay tarea programada ni ejecución automática.

### Supermaxi

```bash
uv run manage.py scrapear_supermaxi
```

Recorre las categorías declaradas en `CATEGORIAS_OBJETIVO` dentro del archivo del scraper. Por defecto scrapea "Ofertas" y "Leches" en la sucursal Iñaquito (Quito). Crea o actualiza `Ciudad`, `Sucursal`, `Comercio`, `Categoria`, `Producto` y `Precio`.

Para agregar más sucursales o categorías, editar las constantes `SUCURSALES` y `CATEGORIAS_OBJETIVO` en [apps/scraper/management/commands/scrapear_supermaxi.py](apps/scraper/management/commands/scrapear_supermaxi.py).

### Coral

```bash
uv run manage.py scrapear_coral
```

Usa el sitemap de Coral para encontrar URLs permitidas (respeta `robots.txt`). Crea o actualiza las mismas entidades que el scraper de Supermaxi.

### Unificar productos duplicados

Coral no publica código de barras, por lo que el mismo producto puede quedar registrado dos veces: una entrada de Supermaxi (con barcode) y otra de Coral (sin barcode). El comando de unificación fusiona ambas entradas bajo un solo `Producto`.

```bash
# 1. Correr ambos scrapers
uv run manage.py scrapear_supermaxi
uv run manage.py scrapear_coral

# 2. Abrir el admin e identificar los IDs de productos equivalentes
#    http://localhost:8000/admin/catalogo/producto/

# 3. Editar PARES_DEMO en el archivo del comando:
#    apps/scraper/management/commands/unificar_productos_demo.py
#    PARES_DEMO = [
#        (46, 67),   # (id_supermaxi_canonico, id_coral_duplicado)
#    ]

# 4. Ejecutar (es idempotente, se puede repetir sin problema)
uv run manage.py unificar_productos_demo
```

Después de la unificación, `/api/kache/productos/<pk>/comparar/` devuelve el producto con precios de ambos comercios.

---

## Paginación

Todos los endpoints de lista están paginados a 20 ítems por página:

```
GET /api/kache/productos/?page=2
```

La respuesta incluye `count`, `next`, `previous` y `results`.

---

## Despliegue en producción

```bash
# En el servidor (DigitalOcean Droplet)
cd /opt/comparador_de_precios
git pull origin main
uv sync
uv run manage.py migrate
uv run manage.py collectstatic --noinput
sudo systemctl restart kache
```

Variables adicionales requeridas en el `.env` de producción:

```env
DEBUG=False
ALLOWED_HOSTS=tu-dominio.com
SECRET_KEY=clave-secreta-larga-y-aleatoria-generada
DB_SSLMODE=require
```

---

## Admin de Django

El panel `/admin/` permite gestionar todos los modelos directamente. Requiere `is_staff=True`.

```bash
uv run manage.py createsuperuser
```
