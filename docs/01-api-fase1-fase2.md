# API Reference — Fase 1 y Fase 2

> **Fecha:** 2026-03-08
> **Estado del backend:** Fases 0, 1 y 2 completadas (23 de 50 endpoints)
> **Base URL:** `http://<host>:6001/api/v1`

---

## Convenciones generales

### Autenticacion

Todos los endpoints (excepto `POST /auth/login`) requieren el header:

```
Authorization: Bearer <access_token>
```

El token se obtiene al hacer login. Expira segun la configuracion del servidor (`JWT_EXPIRES_IN`).

### Formato de respuesta

**Exito — objeto simple:**
```json
{ "data": { ... } }
```

**Exito — lista paginada:**
```json
{
  "data": [ ... ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

**Error de dominio (409, 422, 404):**
```json
{
  "error": {
    "code": "NOMBRE_ERROR",
    "message": "Descripcion legible del error",
    "details": null
  }
}
```

**Error de validacion (400):**
```json
{
  "message": ["nombre must be a string", "email must be an email"],
  "error": "Bad Request",
  "statusCode": 400
}
```

**No autorizado (401):**
```json
{ "message": "Unauthorized", "statusCode": 401 }
```

**Sin permisos (403):**
```json
{ "message": "Forbidden resource", "statusCode": 403 }
```

### Jerarquia de roles

| Rol | Nivel | Acceso |
|-----|-------|--------|
| `bodeguero` | 1 | Catalogo (lectura/escritura), entradas, salidas, stock |
| `supervisor` | 2 | Todo lo de bodeguero + ajustes, alertas, historial |
| `admin` | 3 | Todo, incluyendo gestion de usuarios y activar/desactivar registros |

Un endpoint marcado con rol `bodeguero` acepta tambien `supervisor` y `admin`.

---

## Modulo IAM — Auth

### POST `/auth/login`

Autentica al usuario y devuelve un JWT.

**Auth requerida:** No

**Body:**
```json
{
  "email": "admin@omnitag.cl",
  "password": "admin123"
}
```

**Respuesta 200:**
```json
{
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 401 | Credenciales invalidas o usuario inactivo |

---

### POST `/auth/refresh`

Genera un nuevo token a partir del token actual.

**Auth requerida:** Si (cualquier rol)

**Body:** ninguno

**Respuesta 200:**
```json
{
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

### GET `/auth/me`

Devuelve los datos del usuario autenticado extraidos del JWT.

**Auth requerida:** Si (cualquier rol)

**Respuesta 200:**
```json
{
  "data": {
    "id": 1,
    "rol": "admin",
    "nombre": "Administrador"
  }
}
```

---

## Modulo IAM — Usuarios

> Todos los endpoints de este modulo requieren rol `admin`.

### GET `/usuarios`

Lista todos los usuarios con filtros opcionales.

**Query params:**

| Param | Tipo | Descripcion |
|-------|------|-------------|
| `rol` | `admin` \| `supervisor` \| `bodeguero` | Filtrar por rol |
| `activo` | `true` \| `false` | Filtrar por estado |
| `page` | number | Pagina (default: 1) |
| `limit` | number | Registros por pagina (default: 20) |

**Respuesta 200:**
```json
{
  "data": [
    {
      "id_usuario": 1,
      "nombre": "Administrador",
      "email": "admin@omnitag.cl",
      "rol": "admin",
      "activo": true
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 1,
    "totalPages": 1
  }
}
```

---

### POST `/usuarios`

Crea un nuevo usuario.

**Body:**
```json
{
  "nombre": "Juan Perez",
  "email": "juan@omnitag.cl",
  "password": "MiPassword123!",
  "rol": "bodeguero"
}
```

| Campo | Tipo | Requerido | Validacion |
|-------|------|-----------|------------|
| `nombre` | string | Si | max 80 chars |
| `email` | string | Si | formato email, max 100 chars |
| `password` | string | Si | — |
| `rol` | string | Si | `admin` \| `supervisor` \| `bodeguero` |

**Respuesta 201:**
```json
{
  "data": {
    "id_usuario": 2,
    "nombre": "Juan Perez",
    "email": "juan@omnitag.cl",
    "rol": "bodeguero",
    "activo": true
  }
}
```

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 409 | El email ya esta registrado |

---

### GET `/usuarios/:id`

Obtiene un usuario por ID.

**Respuesta 200:**
```json
{
  "data": {
    "id_usuario": 1,
    "nombre": "Administrador",
    "email": "admin@omnitag.cl",
    "rol": "admin",
    "activo": true
  }
}
```

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 404 | Usuario no encontrado |

---

### PUT `/usuarios/:id`

Actualiza los datos de un usuario.

**Body:** (todos los campos son opcionales)
```json
{
  "nombre": "Juan Perez Actualizado",
  "email": "nuevo@omnitag.cl",
  "rol": "supervisor"
}
```

**Respuesta 200:** mismo formato que GET `/usuarios/:id`

---

### PATCH `/usuarios/:id/activar`

Activa o desactiva un usuario.

**Body:**
```json
{ "activo": false }
```

**Respuesta 200:** mismo formato que GET `/usuarios/:id`

---

## Modulo Catalogo — Categorias

> Todos los endpoints requieren rol minimo `bodeguero`.

### GET `/categorias`

Lista todas las categorias.

**Respuesta 200:**
```json
{
  "data": [
    {
      "id_categoria": 1,
      "nombre": "Herramientas",
      "descripcion": "Herramientas manuales y electricas"
    }
  ]
}
```

---

### GET `/categorias/:id`

Obtiene una categoria por ID.

**Respuesta 200:**
```json
{
  "data": {
    "id_categoria": 1,
    "nombre": "Herramientas",
    "descripcion": "Herramientas manuales y electricas"
  }
}
```

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 404 | Categoria no encontrada |

---

### POST `/categorias`

Crea una nueva categoria.

**Body:**
```json
{
  "nombre": "Herramientas",
  "descripcion": "Herramientas manuales y electricas"
}
```

| Campo | Tipo | Requerido | Validacion |
|-------|------|-----------|------------|
| `nombre` | string | Si | max 80 chars |
| `descripcion` | string | No | — |

**Respuesta 201:** mismo formato que GET `/categorias/:id`

---

### PUT `/categorias/:id`

Actualiza una categoria.

**Body:** mismo que POST

**Respuesta 200:** mismo formato que GET `/categorias/:id`

---

## Modulo Catalogo — Productos

### GET `/productos`

Lista productos con filtros y paginacion.

**Rol requerido:** `bodeguero`

**Query params:**

| Param | Tipo | Descripcion |
|-------|------|-------------|
| `activo` | `true` \| `false` | Filtrar por estado |
| `categoria` | number | Filtrar por id_categoria |
| `q` | string | Busqueda por nombre o codigo |
| `page` | number | Pagina (default: 1) |
| `limit` | number | Registros por pagina (default: 20) |

**Respuesta 200:**
```json
{
  "data": [
    {
      "id_producto": 1,
      "codigo": "PROD-001",
      "nombre": "Martillo 500g",
      "descripcion": "Martillo de acero",
      "unidad_medida": "unidad",
      "stock_min": "5.00",
      "stock_max": "50.00",
      "activo": true,
      "id_categoria": 1
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 1,
    "totalPages": 1
  }
}
```

> Nota: `stock_min` y `stock_max` se devuelven como string decimal (comportamiento de Prisma con `Decimal`).

---

### GET `/productos/:id`

Obtiene un producto por ID.

**Rol requerido:** `bodeguero`

**Respuesta 200:**
```json
{
  "data": {
    "id_producto": 1,
    "codigo": "PROD-001",
    "nombre": "Martillo 500g",
    "descripcion": "Martillo de acero",
    "unidad_medida": "unidad",
    "stock_min": "5.00",
    "stock_max": "50.00",
    "activo": true,
    "id_categoria": 1
  }
}
```

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 404 | Producto no encontrado |

---

### POST `/productos`

Crea un nuevo producto.

**Rol requerido:** `bodeguero`

**Body:**
```json
{
  "codigo": "PROD-001",
  "nombre": "Martillo 500g",
  "descripcion": "Martillo de acero con mango de madera",
  "unidad_medida": "unidad",
  "stock_min": 5,
  "stock_max": 50,
  "id_categoria": 1
}
```

| Campo | Tipo | Requerido | Validacion |
|-------|------|-----------|------------|
| `codigo` | string | Si | max 30 chars, unico |
| `nombre` | string | Si | max 120 chars |
| `descripcion` | string | No | — |
| `unidad_medida` | string | Si | max 20 chars |
| `stock_min` | number | No | >= 0 |
| `stock_max` | number | No | >= 0 |
| `id_categoria` | number | No | — |

**Respuesta 201:** mismo formato que GET `/productos/:id`

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 409 | El codigo de producto ya existe |

---

### PUT `/productos/:id`

Actualiza un producto. Todos los campos son opcionales.

**Rol requerido:** `bodeguero`

**Body:** mismos campos que POST (todos opcionales)

**Respuesta 200:** mismo formato que GET `/productos/:id`

---

### PATCH `/productos/:id/activar`

Activa o desactiva un producto.

**Rol requerido:** `admin`

**Body:**
```json
{ "activo": false }
```

**Respuesta 200:** mismo formato que GET `/productos/:id`

---

## Modulo Catalogo — Proveedores

### GET `/proveedores`

Lista proveedores con filtros y paginacion.

**Rol requerido:** `bodeguero`

**Query params:**

| Param | Tipo | Descripcion |
|-------|------|-------------|
| `activo` | `true` \| `false` | Filtrar por estado |
| `q` | string | Busqueda por nombre o rut |
| `page` | number | Pagina (default: 1) |
| `limit` | number | Registros por pagina (default: 20) |

**Respuesta 200:**
```json
{
  "data": [
    {
      "id_proveedor": 1,
      "rut": "76.123.456-7",
      "nombre": "Distribuidora Industrial Ltda",
      "contacto": "Maria Gonzalez",
      "email": "contacto@distribuidora.cl",
      "telefono": "+56912345678",
      "activo": true
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 1,
    "totalPages": 1
  }
}
```

---

### GET `/proveedores/:id`

Obtiene un proveedor por ID.

**Rol requerido:** `bodeguero`

**Respuesta 200:** mismo formato que un item del listado

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 404 | Proveedor no encontrado |

---

### POST `/proveedores`

Crea un nuevo proveedor.

**Rol requerido:** `bodeguero`

**Body:**
```json
{
  "rut": "76.123.456-7",
  "nombre": "Distribuidora Industrial Ltda",
  "contacto": "Maria Gonzalez",
  "email": "contacto@distribuidora.cl",
  "telefono": "+56912345678"
}
```

| Campo | Tipo | Requerido | Validacion |
|-------|------|-----------|------------|
| `rut` | string | No | max 12 chars |
| `nombre` | string | Si | max 120 chars |
| `contacto` | string | No | max 80 chars |
| `email` | string | No | formato email, max 100 chars |
| `telefono` | string | No | max 20 chars |

**Respuesta 201:** mismo formato que GET `/proveedores/:id`

**Errores:**
| Codigo | Descripcion |
|--------|-------------|
| 409 | El RUT ya esta registrado |

---

### PUT `/proveedores/:id`

Actualiza un proveedor. Todos los campos son opcionales.

**Rol requerido:** `bodeguero`

**Body:** mismos campos que POST (todos opcionales)

**Respuesta 200:** mismo formato que GET `/proveedores/:id`

---

### PATCH `/proveedores/:id/activar`

Activa o desactiva un proveedor.

**Rol requerido:** `admin`

**Body:**
```json
{ "activo": false }
```

**Respuesta 200:** mismo formato que GET `/proveedores/:id`

---

## Endpoints proximos (Fase 3 en adelante)

Los siguientes modulos estan en desarrollo y sus endpoints no estan disponibles aun:

| Modulo | Endpoints | Descripcion |
|--------|-----------|-------------|
| Movimientos — Entradas | 6 | Crear, editar, confirmar y anular entradas de mercaderia |
| Movimientos — Salidas | 6 | Crear, editar, confirmar y anular salidas de mercaderia |
| Movimientos — Ajustes | 3 | Ajustes de inventario inmediatos (solo supervisor) |
| Inventario — Stock | 3 | Consulta de stock actual por producto/bodega |
| Inventario — Bodegas | 4 | CRUD de bodegas |
| Inventario — Alertas | 3 | Consulta y gestion de alertas de stock |
| Inventario — Historial | 2 | Historial de movimientos |

Se publicara un nuevo documento en esta carpeta al completar cada fase.
