---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-012: Crear factura con productos

## Declaración (Statement)
El sistema shall permitir crear una nueva factura seleccionando un cliente y un vendedor desde dropdowns, y agregando una o más líneas de producto con cantidad. La creación se ejecuta mediante el stored procedure `sp_insertar_factura_y_productosporfactura`, que gestiona atómicamente la inserción de la cabecera y el detalle.

## Justificación (Rationale)
La factura es la entidad transaccional central del sistema. Su creación involucra múltiples tablas (factura + productosporfactura) y lógica de negocio (validación de stock, cálculo de subtotales y total) que se ejecuta atómicamente en el servidor mediante SP y triggers.

## Criterios de Aceptación (Acceptance Criteria)
- Al presionar "Nueva Factura", se carga el formulario con:
  - Dropdown de clientes (muestra: "{Nombre} (Crédito: ${crédito})") cargados desde `GET /api/cliente` + `GET /api/persona`
  - Dropdown de vendedores (muestra: "{Nombre} (Carnet: {carnet})") cargados desde `GET /api/vendedor` + `GET /api/persona`
  - Sección de productos con al menos 1 fila:
    - Dropdown de producto (muestra: "{nombre} (Stock: {stock} - ${valorunitario})") cargado desde `GET /api/producto`
    - Campo cantidad (numérico, mínimo 1)
    - Botón "Quitar" (visible si hay más de 1 fila)
  - Botón "Agregar Producto" para añadir filas dinámicamente
- Al guardar, se serializa el arreglo de productos como JSON: `[{"codigo":"PR001","cantidad":2}, ...]`
- Se invoca `POST /api/procedimientos/ejecutarsp` con `nombreSP: "sp_insertar_factura_y_productosporfactura"` y parámetros: `p_fkidcliente`, `p_fkidvendedor`, `p_productos` (JSON)
- Si el SP es exitoso, se muestra mensaje de éxito y se regresa a la vista de lista
- Si hay error (ej: stock insuficiente), se muestra el mensaje retornado por el SP
- Se requiere al menos 1 producto con código seleccionado y cantidad > 0

## Método de Verificación (Verification Method)
Demostración — Crear una factura con 2+ productos. Verificar que aparece en la lista con el total calculado correctamente. Verificar que el stock de los productos disminuyó.

## Más Información (More Information)
- Página: `Components/Pages/Factura.razor` (vista: `"formulario"`, `editando = false`)
- Servicio: `SpService.EjecutarSpAsync("sp_insertar_factura_y_productosporfactura", { p_fkidcliente, p_fkidvendedor, p_productos })`
- Clases auxiliares: `ProductoFila` (Codigo, Cantidad), `ClienteInfo` (Id, Nombre, Credito), `VendedorInfo` (Id, Nombre, Carnet)
- Métodos: `MostrarFormularioNueva()`, `CargarDatosFormulario()`, `GuardarFactura()`, `AgregarFila()`, `QuitarFila()`
- El SP valida stock, calcula subtotales vía triggers, y retorna la factura creada como JSON
