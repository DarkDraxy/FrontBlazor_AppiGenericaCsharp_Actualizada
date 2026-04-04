---
status: "passed"
date: 2026-04-02
---

# REQ-INT-003: Mensajes de retroalimentación y diálogos de confirmación

## Declaración (Statement)
El sistema shall mostrar mensajes de retroalimentación visual después de cada operación CRUD (éxito o error) y shall solicitar confirmación mediante un diálogo JavaScript antes de ejecutar operaciones destructivas (eliminar).

## Justificación (Rationale)
El operador necesita saber inmediatamente si una operación fue exitosa o falló, y las eliminaciones deben requerir confirmación explícita para prevenir pérdida accidental de datos.

## Criterios de Aceptación (Acceptance Criteria)
- **Mensajes de éxito:** Después de crear, editar o eliminar exitosamente, se muestra un mensaje con fondo verde (clase Bootstrap `alert-success`) indicando la operación realizada.
- **Mensajes de error:** Si la API retorna error o no responde, se muestra un mensaje con fondo rojo (clase Bootstrap `alert-danger`) con el texto del error.
- **Indicador de carga:** Mientras se ejecuta una operación asíncrona, se muestra un spinner o texto "Cargando..." controlado por `bool cargando`.
- **Diálogo de confirmación:** Antes de eliminar cualquier registro, se invoca `JSRuntime.InvokeAsync<bool>("confirm", "¿Está seguro...?")`. Solo si retorna `true` se procede con la eliminación.
- Los mensajes se muestran en la parte superior del área de contenido.
- Los mensajes persisten hasta la siguiente operación o hasta que el usuario navega a otra vista.

## Método de Verificación (Verification Method)
Demostración — Ejecutar operaciones exitosas y provocar errores (ej: eliminar un registro referenciado por FK). Verificar que los mensajes aparecen con el color correcto. Intentar eliminar y cancelar el diálogo para verificar que no se ejecuta.

## Más Información (More Information)
- Variables de estado en cada página: `string mensaje`, `bool exito`
- Renderizado condicional: `@if (!string.IsNullOrEmpty(mensaje))` con clase CSS dinámica según `exito`
- JS Interop: `@inject IJSRuntime JSRuntime` → `await JSRuntime.InvokeAsync<bool>("confirm", texto)`
- El spinner se muestra con: `@if (cargando) { <p>Cargando...</p> }`
