---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-004: CRUD Producto

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Producto, compuesta por los campos código (PK, VARCHAR 10), nombre (VARCHAR 100), stock (INT) y valor unitario (DECIMAL 18,2).

## Justificación (Rationale)
Los productos son los ítems comercializables del sistema. Su stock es gestionado automáticamente por triggers al crear, editar o eliminar líneas de factura, pero el registro base del producto se administra desde esta página.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/producto`, se muestran todos los registros con columnas código, nombre, stock y valor unitario. El límite de registros es configurable.
- **Crear:** Formulario con los 4 campos. Stock acepta enteros; valor unitario acepta decimales con step 0.01. Al guardar, `POST /api/producto`.
- **Editar:** Se cargan los datos actuales. Código deshabilitado (PK). Al guardar, `PUT /api/producto/codigo/{valor}`.
- **Eliminar:** Diálogo de confirmación. `DELETE /api/producto/codigo/{valor}`.
- Los campos numéricos (stock, valorunitario) se envían con los tipos correctos en el JSON.

## Método de Verificación (Verification Method)
Demostración — Ejecutar las 4 operaciones CRUD en `/producto`, verificando que los valores numéricos se persisten correctamente.

## Más Información (More Information)
- Página: `Components/Pages/Producto.razor` (ruta: `/producto`)
- Servicio: `ApiService` con tabla `"producto"`
- Tabla BD: `producto` (codigo VARCHAR 10 PK, nombre VARCHAR 100, stock INT, valorunitario DECIMAL 18,2, todos NOT NULL)
- Nota: El stock es modificado automáticamente por los triggers `trg_prodfact_insert`, `trg_prodfact_update` y `trg_prodfact_delete` al gestionar líneas de factura
