---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-009: CRUD Vendedor con clave foránea

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Vendedor, compuesta por id (PK, INT IDENTITY), carnet (INT), dirección (VARCHAR 100) y fkcodpersona (FK → persona.codigo, obligatoria). En la lista, la clave foránea se resolverá mostrando el nombre de la persona asociada.

## Justificación (Rationale)
El vendedor es un rol comercial asignado a una persona del directorio. Tiene un carnet numérico de identificación interna y una dirección. Se referencia como FK en la entidad Factura.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/vendedor`, se muestran los registros con columnas: id, carnet, dirección, nombre de persona (resuelto desde FK). Límite configurable.
- **Crear:** Formulario con campos:
  - Carnet (numérico entero)
  - Dirección (texto)
  - Persona (dropdown `<select>` con las personas cargadas desde `GET /api/persona`, obligatorio)
  - Al guardar: `POST /api/vendedor`
- **Editar:** Se cargan los datos actuales. Id deshabilitado (PK auto). Dropdown preselecciona la persona actual. Al guardar: `PUT /api/vendedor/id/{valor}`.
- **Eliminar:** Diálogo de confirmación. `DELETE /api/vendedor/id/{valor}`.
- Al inicializar la página, se carga la lista de personas para el dropdown.

## Método de Verificación (Verification Method)
Demostración — Crear un vendedor seleccionando una persona. Verificar que la lista muestra el nombre resuelto. Editar y eliminar correctamente.

## Más Información (More Information)
- Página: `Components/Pages/Vendedor.razor` (ruta: `/vendedor`)
- Servicio: `ApiService` con tabla `"vendedor"`, más `ListarAsync("persona")` para el dropdown
- Tabla BD: `vendedor` (id INT IDENTITY PK, carnet INT NOT NULL, direccion VARCHAR 100 NOT NULL, fkcodpersona VARCHAR 10 FK → persona.codigo NOT NULL)
- Patrón: CRUD con FK — similar a Cliente pero con una sola FK obligatoria
