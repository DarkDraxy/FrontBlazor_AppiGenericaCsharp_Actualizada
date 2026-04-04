---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-006: CRUD Ruta

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Ruta, compuesta por los campos ruta (PK, VARCHAR 100) y descripción (VARCHAR 200).

## Justificación (Rationale)
Las rutas representan los paths de la aplicación que pueden ser controlados por permisos basados en roles (tabla pivote rutarol). Su gestión permite definir qué secciones del sistema existen.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/ruta`, se muestran todos los registros con columnas ruta y descripción. El límite de registros es configurable.
- **Crear:** Formulario con campos ruta y descripción. Al guardar, `POST /api/ruta`.
- **Editar:** Se cargan los datos actuales. El campo ruta está deshabilitado (es la PK). Al guardar, `PUT /api/ruta/ruta/{valor}`.
- **Eliminar:** Diálogo de confirmación. `DELETE /api/ruta/ruta/{valor}`.

## Método de Verificación (Verification Method)
Demostración — Ejecutar las 4 operaciones CRUD en la página `/ruta`.

## Más Información (More Information)
- Página: `Components/Pages/Ruta.razor` (ruta de navegación: `/ruta`)
- Servicio: `ApiService` con tabla `"ruta"`
- Tabla BD: `ruta` (ruta VARCHAR 100 PK, descripcion VARCHAR 200 NOT NULL)
- Particularidad: La PK es el propio campo "ruta" (string), lo que hace que el nombre de la clave y de la tabla coincidan
