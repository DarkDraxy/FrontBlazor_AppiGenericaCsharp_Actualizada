---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-013: Editar factura con productos

## Declaración (Statement)
El sistema shall permitir editar una factura existente, modificando el cliente, vendedor y/o las líneas de producto (agregar, quitar o cambiar cantidades). La actualización se ejecuta mediante el stored procedure `sp_actualizar_factura_y_productosporfactura`, que reemplaza atómicamente las líneas de detalle antiguas por las nuevas.

## Justificación (Rationale)
Los errores de facturación (cliente incorrecto, producto equivocado, cantidad errada) requieren poder modificar la factura completa. El SP maneja la lógica de restaurar el stock de los productos anteriores y decrementar el stock de los nuevos mediante triggers.

## Criterios de Aceptación (Acceptance Criteria)
- Al presionar "Editar" en una factura (desde la lista o desde la vista de detalle), se carga el formulario con:
  - Dropdowns de cliente y vendedor preseleccionados con los valores actuales
  - Filas de productos precargadas con los productos y cantidades actuales de la factura
- El operador puede modificar cliente, vendedor, agregar/quitar filas de producto y cambiar cantidades
- Al guardar, se muestra un diálogo de confirmación JavaScript
- Si se confirma, se invoca el SP `sp_actualizar_factura_y_productosporfactura` con parámetros: `p_numero`, `p_fkidcliente`, `p_fkidvendedor`, `p_productos` (JSON)
- El SP elimina los productos antiguos (triggers restauran stock) e inserta los nuevos (triggers decrementan stock)
- Si es exitoso, se muestra mensaje de éxito y se regresa a la vista de lista
- Si hay error, se muestra el mensaje del SP

## Método de Verificación (Verification Method)
Demostración — Editar una factura existente: cambiar un producto, modificar una cantidad y agregar una línea nueva. Verificar que el total se recalcula y que el stock de los productos se ajusta correctamente.

## Más Información (More Information)
- Página: `Components/Pages/Factura.razor` (vista: `"formulario"`, `editando = true`)
- Servicio: `SpService.EjecutarSpAsync("sp_actualizar_factura_y_productosporfactura", { p_numero, p_fkidcliente, p_fkidvendedor, p_productos })`
- Método: `EditarFactura()` carga datos actuales y establece `editando = true`, `numeroEditar = N`
- El SP internamente: DELETE productos antiguos → INSERT productos nuevos (ambos activan triggers de stock)
