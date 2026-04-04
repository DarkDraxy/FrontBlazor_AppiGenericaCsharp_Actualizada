---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-002: CRUD Empresa

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Empresa, compuesta por los campos código (PK, VARCHAR 10) y nombre (VARCHAR 100).

## Justificación (Rationale)
La entidad Empresa es una tabla maestra independiente referenciada como clave foránea por Cliente. Gestionar empresas es un requisito previo para poder asociar clientes a organizaciones.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/empresa`, se muestran todos los registros en una tabla con columnas código y nombre. El límite de registros es configurable.
- **Crear:** Al presionar "Nuevo", se muestra un formulario con campos código y nombre. Al guardar, se invoca `POST /api/empresa` y el registro aparece en la lista.
- **Editar:** Al presionar "Editar" en un registro, se carga el formulario con los datos actuales. El campo código está deshabilitado (es PK). Al guardar, se invoca `PUT /api/empresa/codigo/{valor}`.
- **Eliminar:** Al presionar "Eliminar", se muestra un diálogo de confirmación JavaScript. Si se confirma, se invoca `DELETE /api/empresa/codigo/{valor}` y el registro desaparece de la lista.
- Después de cada operación exitosa, se muestra un mensaje de éxito y se recarga la lista.
- Si la API retorna error, se muestra el mensaje de error al usuario.

## Método de Verificación (Verification Method)
Demostración — Ejecutar las 4 operaciones CRUD en la página `/empresa` y verificar que los datos persisten correctamente.

## Más Información (More Information)
- Página: `Components/Pages/Empresa.razor` (ruta: `/empresa`)
- Servicio: `ApiService` — métodos `ListarAsync("empresa")`, `CrearAsync("empresa", datos)`, `ActualizarAsync("empresa", "codigo", valor, datos)`, `EliminarAsync("empresa", "codigo", valor)`
- Tabla BD: `empresa` (codigo VARCHAR 10 PK, nombre VARCHAR 100 NOT NULL)
- Patrón: CRUD simple — alternancia lista/formulario con `mostrarFormulario` bool
