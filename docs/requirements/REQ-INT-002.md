---
status: "passed"
date: 2026-04-02
---

# REQ-INT-002: Formularios CRUD con alternancia lista/formulario

## Declaración (Statement)
El sistema shall implementar en cada página CRUD un patrón de alternancia entre dos vistas: una vista de lista (tabla de registros) y una vista de formulario (campos editables). Solo una vista es visible a la vez, controlada por una variable booleana. En el caso de Factura, el patrón se extiende a tres vistas: lista, detalle y formulario.

## Justificación (Rationale)
La alternancia lista/formulario en una sola página evita la navegación entre rutas para operaciones CRUD, manteniendo al operador en contexto. Es un patrón común en aplicaciones de gestión que simplifica el flujo de trabajo.

## Criterios de Aceptación (Acceptance Criteria)
- **Vista lista (estado inicial):** Se muestra una tabla con los registros cargados. Botones "Nuevo", "Editar" y "Eliminar" visibles.
- **Vista formulario:** Al presionar "Nuevo" o "Editar", se oculta la lista y se muestra el formulario. Botones "Guardar" y "Cancelar" visibles.
- **Cancelar:** Regresa a la vista lista sin realizar cambios.
- **Guardar exitoso:** Regresa a la vista lista y recarga los datos.
- En páginas CRUD simple y con FK: alternancia controlada por `bool mostrarFormulario`
- En Factura: tres estados controlados por `string vista` ∈ {"listar", "ver", "formulario"}
- El formulario distingue entre modo creación (`editando = false`) y edición (`editando = true`):
  - En creación: todos los campos editables
  - En edición: la PK está deshabilitada
- Campo de límite de registros disponible en la vista lista

## Método de Verificación (Verification Method)
Demostración — En cualquier página CRUD, verificar la transición lista → formulario → lista al crear, editar y cancelar. En Factura, verificar las 3 vistas.

## Más Información (More Information)
- Patrón común en: Empresa, Persona, Producto, Rol, Ruta, Usuario, Cliente, Vendedor
- Variables de estado: `bool mostrarFormulario`, `bool editando`, `bool cargando`, `string mensaje`, `bool exito`
- Factura usa: `string vista` con 3 valores posibles
- Los campos del formulario usan `@bind` para enlace bidireccional de datos
