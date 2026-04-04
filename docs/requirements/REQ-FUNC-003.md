---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-003: CRUD Persona

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Persona, compuesta por los campos código (PK, VARCHAR 10), nombre (VARCHAR 100), email (VARCHAR 100) y teléfono (VARCHAR 20).

## Justificación (Rationale)
La entidad Persona es una tabla maestra referenciada como clave foránea por Cliente y Vendedor. Representa a las personas naturales del sistema comercial.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/persona`, se muestran todos los registros en una tabla con columnas código, nombre, email y teléfono. El límite de registros es configurable.
- **Crear:** Formulario con los 4 campos. Al guardar, se invoca `POST /api/persona`.
- **Editar:** Se cargan los datos actuales. El campo código está deshabilitado (PK). Al guardar, se invoca `PUT /api/persona/codigo/{valor}`.
- **Eliminar:** Diálogo de confirmación. Si se confirma, `DELETE /api/persona/codigo/{valor}`.
- Mensajes de éxito/error y recarga de lista después de cada operación.

## Método de Verificación (Verification Method)
Demostración — Ejecutar las 4 operaciones CRUD en `/persona`.

## Más Información (More Information)
- Página: `Components/Pages/Persona.razor` (ruta: `/persona`)
- Servicio: `ApiService` con tabla `"persona"`
- Tabla BD: `persona` (codigo VARCHAR 10 PK, nombre VARCHAR 100, email VARCHAR 100, telefono VARCHAR 20, todos NOT NULL)
- Patrón: CRUD simple
