---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-014: Eliminar factura con cascada

## Declaración (Statement)
El sistema shall permitir eliminar una factura existente junto con todas sus líneas de producto asociadas. La eliminación se ejecuta mediante el stored procedure `sp_borrar_factura_y_productosporfactura`, que aprovecha ON DELETE CASCADE y los triggers para restaurar el stock de los productos.

## Justificación (Rationale)
Las facturas anuladas o erróneas deben poder eliminarse completamente. La cascada automática elimina las líneas de detalle, y los triggers `trg_prodfact_delete` restauran el stock de cada producto afectado, manteniendo la integridad del inventario.

## Criterios de Aceptación (Acceptance Criteria)
- Al presionar "Eliminar" en una factura de la lista, se muestra un diálogo de confirmación JavaScript
- Si se confirma, se invoca `POST /api/procedimientos/ejecutarsp` con `nombreSP: "sp_borrar_factura_y_productosporfactura"` y parámetro `p_numero`
- El SP elimina la factura; la constraint ON DELETE CASCADE elimina automáticamente las líneas de productosporfactura
- El trigger `trg_prodfact_delete` restaura el stock de cada producto eliminado
- El SP retorna JSON con: mensaje de éxito, número eliminado, total eliminado y cantidad de productos eliminados
- Si es exitoso, se muestra mensaje de éxito y se recarga la lista de facturas
- Si hay error, se muestra el mensaje del SP
- La factura eliminada ya no aparece en la lista

## Método de Verificación (Verification Method)
Demostración — Eliminar una factura con productos. Verificar que: la factura desaparece de la lista, las líneas de detalle ya no existen en la BD, y el stock de los productos involucrados se restauró a los valores previos.

## Más Información (More Information)
- Página: `Components/Pages/Factura.razor` (vista: `"listar"`, acción eliminar)
- Servicio: `SpService.EjecutarSpAsync("sp_borrar_factura_y_productosporfactura", { p_numero: N })`
- Método: `EliminarFactura()`
- El SP retorna: `{"mensaje":"Factura eliminada exitosamente", "numero_eliminado":N, "total_eliminado":M, "productos_eliminados":K}`
- Cadena de eliminación: SP DELETE factura → CASCADE DELETE productosporfactura → trigger restaura stock → SP recalcula respuesta
