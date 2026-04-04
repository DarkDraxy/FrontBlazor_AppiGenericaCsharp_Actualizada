---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-011: Ver detalle de factura con productos

## Declaración (Statement)
El sistema shall mostrar el detalle completo de una factura seleccionada, incluyendo: número, nombre del cliente, nombre del vendedor, fecha formateada, total, y una tabla de productos con código, nombre, cantidad, valor unitario y subtotal. Los datos se obtienen mediante el stored procedure `sp_consultar_factura_y_productosporfactura`.

## Justificación (Rationale)
El operador necesita inspeccionar el contenido completo de una factura sin modificarla, incluyendo cada línea de producto con sus cantidades y subtotales calculados.

## Criterios de Aceptación (Acceptance Criteria)
- Al presionar "Ver" en una factura de la lista, se invoca `POST /api/procedimientos/ejecutarsp` con `nombreSP: "sp_consultar_factura_y_productosporfactura"` y parámetro `p_numero`
- Se muestra una tarjeta (card) con encabezado "Factura #{número}"
- Se muestran los campos: Cliente, Vendedor, Fecha (formato `yyyy-MM-dd HH:mm`), Total
- Se muestra una tabla de productos con columnas: Código, Nombre, Cantidad, Valor Unitario, Subtotal
- Botones disponibles: Volver (regresa a lista), Editar (navega al formulario de edición)
- Si la factura no existe o el SP retorna error, se muestra mensaje de error

## Método de Verificación (Verification Method)
Demostración — Seleccionar "Ver" en una factura existente y verificar que los datos de cabecera y las líneas de producto coinciden con los datos de la BD.

## Más Información (More Information)
- Página: `Components/Pages/Factura.razor` (vista: `"ver"`)
- Servicio: `SpService.EjecutarSpAsync("sp_consultar_factura_y_productosporfactura", { p_numero: N })`
- El SP retorna JSON: `{"factura":{numero, fecha, total, fkidcliente, nombre_cliente, fkidvendedor, nombre_vendedor}, "productos":[{codigo_producto, nombre_producto, cantidad, valorunitario, subtotal}]}`
- Métodos auxiliares: `ObtenerProductos()`, `FormatearFecha()`, `ObtenerValor()`
