# Diccionario de Datos (Data Dictionary)
## FrontBlazor — Sistema CRUD Genérico con Facturación

Versión 1.0  
Preparado por Carlos Arturo Castro Castro  
Proyecto Educativo — Blazor Server + API REST Genérica  
2026-04-02

## Tabla de Contenido (Table of Contents)
<!-- TOC -->
* [1. Introducción (Introduction)](#1-introducción-introduction)
* [2. Tablas Independientes (Independent Tables)](#2-tablas-independientes-independent-tables)
    * [2.1 empresa](#21-empresa)
    * [2.2 persona](#22-persona)
    * [2.3 producto](#23-producto)
    * [2.4 rol](#24-rol)
    * [2.5 ruta](#25-ruta)
    * [2.6 usuario](#26-usuario)
* [3. Tablas Dependientes (Dependent Tables)](#3-tablas-dependientes-dependent-tables)
    * [3.1 cliente](#31-cliente)
    * [3.2 vendedor](#32-vendedor)
    * [3.3 factura](#33-factura)
    * [3.4 productosporfactura](#34-productosporfactura)
* [4. Tablas Pivote (Junction Tables)](#4-tablas-pivote-junction-tables)
    * [4.1 rol_usuario](#41-rol_usuario)
    * [4.2 rutarol](#42-rutarol)
* [5. Diagrama de Relaciones (Relationships Diagram)](#5-diagrama-de-relaciones-relationships-diagram)
* [6. Restricciones y Constraints (Constraints)](#6-restricciones-y-constraints-constraints)
* [7. Triggers](#7-triggers)
* [8. Stored Procedures](#8-stored-procedures)
    * [8.1 Facturación (Invoice)](#81-facturación-invoice)
    * [8.2 Usuarios y Roles (Users and Roles)](#82-usuarios-y-roles-users-and-roles)
    * [8.3 Rutas y Permisos (Routes and Permissions)](#83-rutas-y-permisos-routes-and-permissions)
* [9. Datos de Ejemplo (Sample Data)](#9-datos-de-ejemplo-sample-data)
<!-- TOC -->

## Historial de Revisiones (Revision History)

| Nombre | Fecha | Motivo del Cambio | Versión |
|--------|-------|--------------------|---------|
| Castro Castro, C. A. | 2026-04-02 | Documento inicial | 1.0 |

---

## 1. Introducción (Introduction)

Este documento describe la estructura completa de la base de datos `bdfacturas_sqlserver_local` utilizada por el sistema FrontBlazor. Incluye la definición detallada de cada tabla, columna, tipo de dato, restricciones, triggers, stored procedures y datos de ejemplo.

**Motor de base de datos:** SQL Server 2016+  
**Script fuente:** [`../script_bd/bdfacturas_sqlserver.sql`](../script_bd/bdfacturas_sqlserver.sql)  
**Script alternativo (PostgreSQL):** [`../script_bd/bdfacturas_postgres.sql`](../script_bd/bdfacturas_postgres.sql)  
**Script alternativo (MySQL/MariaDB):** [`../script_bd/bdfacturas_mysql_mariadb.sql`](../script_bd/bdfacturas_mysql_mariadb.sql)  
**API backend:** Castro Castro, C. A. (2026). *ApiGenericaCsharp*. https://github.com/ccastro2050/ApiGenericaCsharp

**Convenciones de nomenclatura:**
- Tablas y columnas: `snake_case` en minúsculas
- Claves foráneas: prefijo `fk` + nombre de la columna referenciada (ej: `fkcodpersona` → `persona.codigo`)
- Constraints: `pk_tabla` (PK), `fk_tabla_referencia` (FK), `uq_tabla` (UNIQUE)
- Triggers: `trg_tabla_operacion` (ej: `trg_prodfact_insert`)

**Tipos de datos utilizados:**

| Tipo SQL Server | Descripción | Uso |
|-----------------|-------------|-----|
| NVARCHAR(n) | Texto Unicode de longitud variable | Códigos, nombres, emails |
| INT | Entero de 32 bits | IDs auto-incrementales, cantidades |
| DECIMAL(18,2) | Numérico exacto con 2 decimales | Valores monetarios, créditos |
| DATETIME2 | Fecha y hora de alta precisión | Fecha de factura |
| INT IDENTITY(1,1) | Entero auto-incremental desde 1 | PKs generadas automáticamente |

---

## 2. Tablas Independientes (Independent Tables)

Tablas sin claves foráneas. Pueden existir sin depender de otras tablas.

### 2.1 empresa

Empresas u organizaciones comerciales. Referenciada como FK opcional por `cliente`.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **codigo** | NVARCHAR(10) | NO | — | PK (`pk_empresa`) | Código único de la empresa |
| nombre | NVARCHAR(100) | NO | — | — | Nombre o razón social |

**Página CRUD:** `/empresa` ([REQ-FUNC-002](requirements/REQ-FUNC-002.md))  
**Referenciada por:** `cliente.fkcodempresa`

### 2.2 persona

Personas naturales del sistema. Referenciada como FK por `cliente` y `vendedor`.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **codigo** | NVARCHAR(10) | NO | — | PK (`pk_persona`) | Código único de la persona |
| nombre | NVARCHAR(100) | NO | — | — | Nombre completo |
| email | NVARCHAR(100) | NO | — | — | Correo electrónico |
| telefono | NVARCHAR(20) | NO | — | — | Número telefónico |

**Página CRUD:** `/persona` ([REQ-FUNC-003](requirements/REQ-FUNC-003.md))  
**Referenciada por:** `cliente.fkcodpersona`, `vendedor.fkcodpersona`

### 2.3 producto

Productos comercializables con control de stock. Referenciada por `productosporfactura`.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **codigo** | NVARCHAR(10) | NO | — | PK (`pk_producto`) | Código único del producto |
| nombre | NVARCHAR(100) | NO | — | — | Nombre del producto |
| stock | INT | NO | — | — | Unidades disponibles en inventario |
| valorunitario | DECIMAL(18,2) | NO | — | — | Precio por unidad |

**Página CRUD:** `/producto` ([REQ-FUNC-004](requirements/REQ-FUNC-004.md))  
**Referenciada por:** `productosporfactura.fkcodproducto`  
**Nota:** El stock es modificado automáticamente por los triggers de `productosporfactura` al insertar, actualizar o eliminar líneas de factura.

### 2.4 rol

Roles de acceso al sistema (Administrador, Vendedor, Cajero, etc.).

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **id** | INT IDENTITY(1,1) | NO | Auto | PK (`pk_rol`) | ID auto-incremental |
| nombre | NVARCHAR(50) | NO | — | — | Nombre del rol |

**Página CRUD:** `/rol` ([REQ-FUNC-005](requirements/REQ-FUNC-005.md))  
**Referenciada por:** `rol_usuario.fkidrol`, `rutarol.fkidrol`

### 2.5 ruta

Rutas (paths) de la aplicación que pueden controlarse por permisos.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **id** | INT IDENTITY(1,1) | NO | Auto | PK (`pk_ruta`) | ID auto-incremental |
| ruta | NVARCHAR(100) | NO | — | UNIQUE (`uq_ruta`) | Path de la ruta (ej: `/factura`) |
| descripcion | NVARCHAR(200) | NO | — | — | Descripción de la sección |

**Página CRUD:** `/ruta` ([REQ-FUNC-006](requirements/REQ-FUNC-006.md))  
**Referenciada por:** `rutarol.fkidruta`

### 2.6 usuario

Cuentas de acceso al sistema con contraseña (opcionalmente encriptada).

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **email** | NVARCHAR(100) | NO | — | PK (`pk_usuario`) | Email como identificador único |
| contrasena | NVARCHAR(200) | NO | — | — | Contraseña (texto plano o hash) |

**Página CRUD:** `/usuario` ([REQ-FUNC-007](requirements/REQ-FUNC-007.md))  
**Referenciada por:** `rol_usuario.fkemail`  
**Nota:** El campo NVARCHAR(200) permite almacenar tanto texto plano como hashes de longitud variable. La encriptación se ejecuta server-side en la API.

---

## 3. Tablas Dependientes (Dependent Tables)

Tablas con claves foráneas hacia tablas independientes.

### 3.1 cliente

Clientes comerciales vinculados a una persona y opcionalmente a una empresa.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **id** | INT IDENTITY(1,1) | NO | Auto | PK (`pk_cliente`) | ID auto-incremental |
| credito | DECIMAL(18,2) | NO | 0 | — | Límite de crédito del cliente |
| fkcodpersona | NVARCHAR(10) | NO | — | FK (`fk_cliente_persona`) → `persona.codigo` | Persona asociada (obligatoria) |
| fkcodempresa | NVARCHAR(10) | SÍ | NULL | FK (`fk_cliente_empresa`) → `empresa.codigo` | Empresa asociada (opcional) |

**Página CRUD:** `/cliente` ([REQ-FUNC-008](requirements/REQ-FUNC-008.md))  
**Referenciada por:** `factura.fkidcliente`

### 3.2 vendedor

Vendedores con carnet de identificación interna, vinculados a una persona.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **id** | INT IDENTITY(1,1) | NO | Auto | PK (`pk_vendedor`) | ID auto-incremental |
| carnet | INT | NO | — | — | Número de carnet interno |
| direccion | NVARCHAR(100) | NO | — | — | Dirección del vendedor |
| fkcodpersona | NVARCHAR(10) | NO | — | FK (`fk_vendedor_persona`) → `persona.codigo` | Persona asociada (obligatoria) |

**Página CRUD:** `/vendedor` ([REQ-FUNC-009](requirements/REQ-FUNC-009.md))  
**Referenciada por:** `factura.fkidvendedor`

### 3.3 factura

Cabecera de factura comercial. Registro padre en la relación master-detail con `productosporfactura`.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **numero** | INT IDENTITY(1,1) | NO | Auto | PK (`pk_factura`) | Número de factura auto-incremental |
| fecha | DATETIME2 | NO | GETDATE() | — | Fecha y hora de creación |
| total | DECIMAL(18,2) | NO | 0 | — | Total calculado (Σ subtotales) |
| fkidcliente | INT | NO | — | FK (`fk_factura_cliente`) → `cliente.id` | Cliente que compra |
| fkidvendedor | INT | NO | — | FK (`fk_factura_vendedor`) → `vendedor.id` | Vendedor que realiza la venta |

**Página CRUD:** `/factura` ([REQ-FUNC-010](requirements/REQ-FUNC-010.md) a [REQ-FUNC-014](requirements/REQ-FUNC-014.md))  
**Nota:** El campo `total` se calcula automáticamente por los triggers de `productosporfactura`. No se ingresa manualmente.

### 3.4 productosporfactura

Detalle de productos por factura (tabla de detalle en relación master-detail). PK compuesta.

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **fknumfactura** | INT | NO | — | PK + FK (`fk_prodfact_factura`) → `factura.numero` ON DELETE CASCADE | Número de factura padre |
| **fkcodproducto** | NVARCHAR(10) | NO | — | PK + FK (`fk_prodfact_producto`) → `producto.codigo` | Código del producto |
| cantidad | INT | NO | — | — | Unidades vendidas |
| subtotal | DECIMAL(18,2) | NO | 0 | — | Calculado: cantidad × valorunitario |

**Gestionada vía:** Stored procedures de factura (no tiene página CRUD propia)  
**Nota:** El `subtotal` se calcula automáticamente por el trigger `trg_prodfact_insert`. La constraint `ON DELETE CASCADE` elimina automáticamente los productos cuando se elimina la factura padre.

---

## 4. Tablas Pivote (Junction Tables)

Tablas de relación muchos-a-muchos con PK compuesta.

### 4.1 rol_usuario

Asociación entre usuarios y roles (un usuario puede tener múltiples roles).

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **fkemail** | NVARCHAR(100) | NO | — | PK + FK (`fk_rolusuario_usuario`) → `usuario.email` | Email del usuario |
| **fkidrol** | INT | NO | — | PK + FK (`fk_rolusuario_rol`) → `rol.id` | ID del rol |

**Gestionada vía:** Stored procedures de usuarios y roles

### 4.2 rutarol

Asociación entre rutas y roles (control de acceso basado en roles a rutas de la aplicación).

| Columna | Tipo | Nulo | Default | Constraint | Descripción |
|---------|------|------|---------|------------|-------------|
| **fkidruta** | INT | NO | — | PK + FK (`fk_rutarol_ruta`) → `ruta.id` ON DELETE CASCADE | ID de la ruta |
| **fkidrol** | INT | NO | — | PK + FK (`fk_rutarol_rol`) → `rol.id` ON DELETE CASCADE | ID del rol |

**Gestionada vía:** Stored procedures de rutas y permisos  
**Nota:** Ambas FKs tienen ON DELETE CASCADE — al eliminar una ruta o un rol, las asociaciones se eliminan automáticamente.

---

## 5. Diagrama de Relaciones (Relationships Diagram)

```
                    TABLAS INDEPENDIENTES
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ empresa  │    │ persona  │    │ producto │
    │ codigo PK│    │ codigo PK│    │ codigo PK│
    └────┬─────┘    └──┬───┬──┘    └────┬─────┘
         │ 0..1        │   │            │
         │        1..1 │   │ 1..1       │ 1..*
    ┌────▼─────────────▼┐ ┌▼──────────┐ │
    │ cliente           │ │ vendedor  │ │
    │ id PK (IDENTITY)  │ │ id PK (ID)│ │
    │ fkcodpersona FK   │ │ fkcodpers │ │
    │ fkcodempresa FK ? │ └─────┬─────┘ │
    └────────┬──────────┘       │        │
             │ 1..1         1..1│        │
             │     ┌────────────┘        │
             │     │                     │
        ┌────▼─────▼────┐    ┌───────────▼──────────┐
        │ factura       │    │ productosporfactura  │
        │ numero PK (ID)│◀──→│ fknumfactura FK (CC) │
        │ fkidcliente FK│    │ fkcodproducto FK     │
        │ fkidvendedor  │    │ (PK compuesta)       │
        └───────────────┘    └──────────────────────┘
                                ON DELETE CASCADE

                    TABLAS DE ACCESO
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ usuario  │    │   rol    │    │   ruta   │
    │ email PK │    │ id PK(ID)│    │ id PK(ID)│
    └────┬─────┘    └──┬───┬──┘    └────┬─────┘
         │             │   │            │
         │ *..* (N:M)  │   │ *..* (N:M) │
    ┌────▼─────────────▼┐ ┌▼────────────▼───┐
    │ rol_usuario       │ │ rutarol         │
    │ fkemail FK        │ │ fkidrol FK (CC) │
    │ fkidrol FK        │ │ fkidruta FK (CC)│
    │ (PK compuesta)    │ │ (PK compuesta)  │
    └───────────────────┘ └─────────────────┘
                           ON DELETE CASCADE

Leyenda:
  PK = Primary Key     FK = Foreign Key
  ID = IDENTITY        CC = CASCADE
  ?  = Nullable        N:M = Muchos a muchos
```

---

## 6. Restricciones y Constraints (Constraints)

### 6.1 Primary Keys

| Tabla | Constraint | Columna(s) | Tipo |
|-------|-----------|------------|------|
| empresa | `pk_empresa` | codigo | Simple (string) |
| persona | `pk_persona` | codigo | Simple (string) |
| producto | `pk_producto` | codigo | Simple (string) |
| rol | `pk_rol` | id | Simple (IDENTITY) |
| ruta | `pk_ruta` | id | Simple (IDENTITY) |
| usuario | `pk_usuario` | email | Simple (string) |
| cliente | `pk_cliente` | id | Simple (IDENTITY) |
| vendedor | `pk_vendedor` | id | Simple (IDENTITY) |
| factura | `pk_factura` | numero | Simple (IDENTITY) |
| productosporfactura | `pk_productosporfactura` | fknumfactura + fkcodproducto | Compuesta |
| rol_usuario | `pk_rol_usuario` | fkemail + fkidrol | Compuesta |
| rutarol | `pk_rutarol` | fkidruta + fkidrol | Compuesta |

### 6.2 Foreign Keys

| Constraint | Tabla Origen | Columna | → Tabla Destino | → Columna | Nulo | On Delete |
|-----------|-------------|---------|----------------|-----------|------|-----------|
| `fk_cliente_persona` | cliente | fkcodpersona | persona | codigo | NO | NO ACTION |
| `fk_cliente_empresa` | cliente | fkcodempresa | empresa | codigo | SÍ | NO ACTION |
| `fk_vendedor_persona` | vendedor | fkcodpersona | persona | codigo | NO | NO ACTION |
| `fk_factura_cliente` | factura | fkidcliente | cliente | id | NO | NO ACTION |
| `fk_factura_vendedor` | factura | fkidvendedor | vendedor | id | NO | NO ACTION |
| `fk_prodfact_factura` | productosporfactura | fknumfactura | factura | numero | NO | **CASCADE** |
| `fk_prodfact_producto` | productosporfactura | fkcodproducto | producto | codigo | NO | NO ACTION |
| `fk_rolusuario_usuario` | rol_usuario | fkemail | usuario | email | NO | NO ACTION |
| `fk_rolusuario_rol` | rol_usuario | fkidrol | rol | id | NO | NO ACTION |
| `fk_rutarol_ruta` | rutarol | fkidruta | ruta | id | NO | **CASCADE** |
| `fk_rutarol_rol` | rutarol | fkidrol | rol | id | NO | **CASCADE** |

### 6.3 Unique Constraints

| Constraint | Tabla | Columna | Descripción |
|-----------|-------|---------|-------------|
| `uq_ruta` | ruta | ruta | El path de ruta debe ser único |

---

## 7. Triggers

Los tres triggers operan sobre la tabla `productosporfactura` y gestionan automáticamente el stock de productos y el total de la factura.

### 7.1 trg_prodfact_insert (AFTER INSERT)

**Disparo:** Al insertar una línea de producto en una factura.

| Paso | Acción | Detalle |
|------|--------|---------|
| 1 | Validar stock | Si `producto.stock < cantidad` → `THROW 50001` con mensaje descriptivo y ROLLBACK |
| 2 | Calcular subtotal | `UPDATE productosporfactura SET subtotal = cantidad × producto.valorunitario` |
| 3 | Decrementar stock | `UPDATE producto SET stock = stock - cantidad` |
| 4 | Recalcular total | `UPDATE factura SET total = (SELECT SUM(subtotal) FROM productosporfactura WHERE fknumfactura = N)` |

### 7.2 trg_prodfact_update (AFTER UPDATE)

**Disparo:** Al modificar una línea de producto existente.

| Paso | Acción | Detalle |
|------|--------|---------|
| 1 | Validar stock | Considera devolución de la cantidad anterior: si `stock + cantidad_anterior < cantidad_nueva` → error |
| 2 | Recalcular subtotal | `subtotal = nueva_cantidad × valorunitario` |
| 3 | Ajustar stock | `stock = stock + cantidad_anterior - cantidad_nueva` (devuelve anterior, resta nueva) |
| 4 | Recalcular total | `total = SUM(subtotales)` de la factura afectada |

### 7.3 trg_prodfact_delete (AFTER DELETE)

**Disparo:** Al eliminar una línea de producto (manual o por CASCADE).

| Paso | Acción | Detalle |
|------|--------|---------|
| 1 | Restaurar stock | `UPDATE producto SET stock = stock + cantidad_eliminada` |
| 2 | Recalcular total | `total = SUM(subtotales)` de la factura afectada (o 0 si no quedan líneas) |

---

## 8. Stored Procedures

### 8.1 Facturación (Invoice)

#### sp_insertar_factura_y_productosporfactura

**Propósito:** Crear una factura con sus productos en una sola transacción atómica.

| Parámetro | Tipo | Dirección | Descripción |
|-----------|------|-----------|-------------|
| @p_fkidcliente | INT | IN | ID del cliente |
| @p_fkidvendedor | INT | IN | ID del vendedor |
| @p_productos | NVARCHAR(MAX) | IN | JSON array: `[{"codigo":"PR001","cantidad":2}, ...]` |
| @p_minimo_detalle | INT | IN (default 1) | Mínimo de productos requeridos |
| @p_resultado | NVARCHAR(MAX) | OUT | JSON resultado con factura y productos creados |

**Retorna (JSON):**
```json
{
  "factura": { "numero": 1, "fecha": "...", "total": 5000000, "fkidcliente": 1, "fkidvendedor": 1 },
  "productos": [{ "codigo_producto": "PR001", "nombre_producto": "...", "cantidad": 2, "valorunitario": 2500000, "subtotal": 5000000 }]
}
```

#### sp_consultar_factura_y_productosporfactura

**Propósito:** Obtener una factura con sus productos y nombres resueltos.

| Parámetro | Tipo | Dirección | Descripción |
|-----------|------|-----------|-------------|
| @p_numero | INT | IN | Número de factura |
| @p_resultado | NVARCHAR(MAX) | OUT | JSON con factura + productos |

**Retorna:** JSON con factura (incluye `nombre_cliente`, `nombre_vendedor`) y array de productos.

#### sp_listar_facturas_y_productosporfactura

**Propósito:** Listar todas las facturas con sus productos y nombres resueltos.

| Parámetro | Tipo | Dirección | Descripción |
|-----------|------|-----------|-------------|
| @p_resultado | NVARCHAR(MAX) | OUT | JSON array con todas las facturas |

**Retorna:** JSON array donde cada elemento tiene estructura factura + productos.

#### sp_actualizar_factura_y_productosporfactura

**Propósito:** Actualizar cabecera y reemplazar productos de una factura existente.

| Parámetro | Tipo | Dirección | Descripción |
|-----------|------|-----------|-------------|
| @p_numero | INT | IN | Número de factura a actualizar |
| @p_fkidcliente | INT | IN | Nuevo ID de cliente |
| @p_fkidvendedor | INT | IN | Nuevo ID de vendedor |
| @p_productos | NVARCHAR(MAX) | IN | Nuevo JSON array de productos |
| @p_minimo_detalle | INT | IN (default 1) | Mínimo de productos requeridos |
| @p_resultado | NVARCHAR(MAX) | OUT | JSON resultado |

**Lógica interna:** DELETE productos antiguos (trigger restaura stock) → UPDATE cabecera → INSERT nuevos productos (trigger decrementa stock).

#### sp_borrar_factura_y_productosporfactura

**Propósito:** Eliminar una factura y sus productos con restauración de stock.

| Parámetro | Tipo | Dirección | Descripción |
|-----------|------|-----------|-------------|
| @p_numero | INT | IN | Número de factura a eliminar |
| @p_resultado | NVARCHAR(MAX) | OUT | JSON confirmación |

**Retorna:**
```json
{
  "mensaje": "Factura eliminada exitosamente",
  "numero_eliminado": 1,
  "total_eliminado": 5000000.00,
  "productos_eliminados": 2
}
```

**Lógica:** DELETE factura → CASCADE elimina productosporfactura → trigger `trg_prodfact_delete` restaura stock de cada producto.

### 8.2 Usuarios y Roles (Users and Roles)

| SP | Propósito | Parámetros Principales |
|----|-----------|------------------------|
| `crear_usuario_con_roles` | Crear usuario y asignar roles | email, contrasena, roles (JSON) |
| `actualizar_usuario_con_roles` | Actualizar usuario y sus roles | email, contrasena, roles (JSON) |
| `eliminar_usuario_con_roles` | Eliminar usuario y sus asociaciones | email |
| `actualizar_roles_usuario` | Reemplazar los roles de un usuario | email, roles (JSON) |
| `consultar_usuario_con_roles` | Obtener usuario con sus roles | email |
| `listar_usuarios_con_roles` | Listar todos los usuarios con roles | — |

### 8.3 Rutas y Permisos (Routes and Permissions)

| SP | Propósito | Parámetros Principales |
|----|-----------|------------------------|
| `verificar_acceso_ruta` | Verificar si un rol tiene acceso a una ruta | ruta, rol |
| `listar_rutarol` | Listar todas las asociaciones ruta-rol | — |
| `crear_rutarol` | Crear asociación ruta-rol | idruta, idrol |
| `eliminar_rutarol` | Eliminar asociación ruta-rol | idruta, idrol |

---

## 9. Datos de Ejemplo (Sample Data)

El script SQL incluye datos de ejemplo para facilitar las pruebas.

### 9.1 Empresas (2 + 1 test)

| codigo | nombre |
|--------|--------|
| E001 | Comercial Los Andes S.A. |
| E002 | Distribuciones El Centro S.A. |
| E999 | Empresa Test |

### 9.2 Personas (6)

| codigo | nombre | email | telefono |
|--------|--------|-------|----------|
| P001 | Ana Torres | ana.torres@correo.com | 3011111111 |
| P002 | Carlos Pérez | carlos.perez@correo.com | 3022222222 |
| P003 | María Gómez | maria.gomez@correo.com | 3033333333 |
| P004 | Juan Díaz | juan.diaz@correo.com | 3044444444 |
| P005 | Laura Rojas | laura.rojas@correo.com | 3055555555 |
| P006 | Pedro Castillo | pedro.castillo@correo.com | 3066666666 |

### 9.3 Productos (8)

| codigo | nombre | stock | valorunitario |
|--------|--------|-------|---------------|
| PR001 | Laptop Lenovo IdeaPad | 17 | 2.500.000 |
| PR002 | Monitor Samsung 24" | 27 | 800.000 |
| PR003 | Teclado Logitech K380 | 42 | 150.000 |
| PR004 | Mouse HP | 55 | 90.000 |
| PR005 | Impresora Epson EcoTank | 14 | 1.100.000 |
| PR006 | Auriculares Sony WH-CH510 | 23 | 240.000 |
| PR007 | Tablet Samsung Tab A9 | 15 | 950.000 |
| PR008 | Disco Duro Seagate 1TB | 32 | 280.000 |

### 9.4 Roles (5)

| id | nombre |
|----|--------|
| 1 | Administrador |
| 2 | Vendedor |
| 3 | Cajero |
| 4 | Contador |
| 5 | Cliente |

### 9.5 Clientes (4)

| id | credito | persona | empresa |
|----|---------|---------|---------|
| 1 | 520.000 | P001 — Ana Torres | E001 |
| 2 | 250.000 | P003 — María Gómez | E002 |
| 3 | 400.000 | P005 — Laura Rojas | E001 |
| 5 | 700.000 | P006 — Pedro Castillo | E001 |

### 9.6 Vendedores (3)

| id | carnet | direccion | persona |
|----|--------|-----------|---------|
| 1 | 1001 | Calle 10 #5-33 | P002 — Carlos Pérez |
| 2 | 1002 | Carrera 15 #7-20 | P004 — Juan Díaz |
| 3 | 1003 | Avenida 30 #18-09 | P006 — Pedro Castillo |

### 9.7 Facturas de Ejemplo (6)

| numero | cliente | vendedor | productos | total |
|--------|---------|----------|-----------|-------|
| 1 | Ana Torres | Carlos Pérez | PR001 ×2 | 5.000.000 |
| 2 | María Gómez | Juan Díaz | PR002 ×1, PR003 ×3 | 1.250.000 |
| 3 | Laura Rojas | Pedro Castillo | PR004 ×5, PR005 ×1, PR006 ×2 | 2.030.000 |
| 4 | Ana Torres | Carlos Pérez | PR007 ×1 | 950.000 |
| 5 | María Gómez | Juan Díaz | PR007 ×2, PR008 ×3 | 2.740.000 |
| 6 | Laura Rojas | Pedro Castillo | PR001 ×1, PR002 ×2, PR003 ×5 | 4.850.000 |
