---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-010: Listar facturas con resumen

## Declaración (Statement)
El sistema shall mostrar una lista de todas las facturas registradas, incluyendo para cada una: número, nombre del cliente, nombre del vendedor, fecha, total y cantidad de productos asociados. Los datos se obtienen mediante el stored procedure `sp_listar_facturas_y_productosporfactura`.

## Justificación (Rationale)
La vista de lista es el punto de entrada al módulo de facturación. El operador necesita un resumen rápido de todas las facturas con datos ya resueltos (nombres en lugar de IDs) para decidir qué factura ver, editar o eliminar.

## Criterios de Aceptación (Acceptance Criteria)
- Al acceder a `/factura`, se invoca `POST /api/procedimientos/ejecutarsp` con `nombreSP: "sp_listar_facturas_y_productosporfactura"`
- Se muestra una tabla con columnas: Número, Cliente, Vendedor, Fecha (formateada), Total, Productos (conteo), Acciones
- La columna "Productos" muestra la cantidad de líneas de detalle de cada factura
- Cada fila tiene botones: Ver, Editar, Eliminar
- Botón "Nueva Factura" disponible sobre la tabla
- Se muestra spinner de carga mientras se obtienen los datos
- Si el SP retorna error, se muestra el mensaje al usuario

## Método de Verificación (Verification Method)
Demostración — Con facturas de ejemplo cargadas en la BD, acceder a `/factura` y verificar que la tabla muestra datos correctos con nombres resueltos y conteos de productos.

## Más Información (More Information)
- Página: `Components/Pages/Factura.razor` (ruta: `/factura`, vista: `"listar"`)
- Servicio: `SpService.EjecutarSpAsync("sp_listar_facturas_y_productosporfactura")`
- El SP retorna JSON con estructura: `[{"factura":{...},"productos":[...]}, ...]`
- El frontend parsea el JSON y aplana la estructura con `AplanarFacturaJson()`
- Variable de estado: `string vista = "listar"` controla qué vista se muestra
