---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-005: CRUD Rol

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Rol, compuesta por los campos id (PK, INT IDENTITY auto-incremental) y nombre (VARCHAR 50).

## Justificación (Rationale)
Los roles definen los perfiles de acceso del sistema (Administrador, Vendedor, Cajero, Contador, Cliente). Se asocian a usuarios mediante la tabla pivote rol_usuario.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/rol`, se muestran todos los registros con columnas id y nombre. El límite de registros es configurable.
- **Crear:** Formulario con campo nombre únicamente. El id se genera automáticamente (IDENTITY). Al guardar, `POST /api/rol`.
- **Editar:** Se carga el nombre actual. El campo id está deshabilitado (PK auto-generada). Al guardar, `PUT /api/rol/id/{valor}`.
- **Eliminar:** Diálogo de confirmación. `DELETE /api/rol/id/{valor}`.
- El id no se incluye en el payload de creación (lo asigna la BD).

## Método de Verificación (Verification Method)
Demostración — Ejecutar las 4 operaciones CRUD en `/rol`, verificando que el id se auto-genera al crear.

## Más Información (More Information)
- Página: `Components/Pages/Rol.razor` (ruta: `/rol`)
- Servicio: `ApiService` con tabla `"rol"`
- Tabla BD: `rol` (id INT IDENTITY PK, nombre VARCHAR 50 NOT NULL)
- Diferencia con CRUD simple de texto: PK es INT auto-incremental, no ingresada por el usuario
