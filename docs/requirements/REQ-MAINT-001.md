---
status: "passed"
date: 2026-04-02
---

# REQ-MAINT-001: Patrón CRUD reutilizable con servicio genérico

## Declaración (Statement)
El sistema shall implementar un servicio genérico (`ApiService`) que permita ejecutar operaciones CRUD sobre cualquier tabla de la base de datos sin necesidad de crear un servicio específico por entidad. Las páginas CRUD shall seguir un patrón replicable (lista/formulario, variables de estado, métodos estándar) que permita agregar nuevas entidades con mínimo esfuerzo.

## Justificación (Rationale)
Un servicio genérico elimina la duplicación de código HTTP por entidad. Un patrón de página estandarizado permite que un desarrollador cree una nueva página CRUD copiando una existente y modificando solo los campos específicos de la entidad, reduciendo el tiempo de desarrollo y los errores.

## Criterios de Aceptación (Acceptance Criteria)
- `ApiService` opera sobre cualquier tabla usando el parámetro `string tabla`:
  - `ListarAsync(tabla, limite?)` — GET genérico
  - `CrearAsync(tabla, datos, camposEncriptar?)` — POST genérico
  - `ActualizarAsync(tabla, nombreClave, valorClave, datos, camposEncriptar?)` — PUT genérico
  - `EliminarAsync(tabla, nombreClave, valorClave)` — DELETE genérico
- Los datos se manejan como `Dictionary<string, object?>` en lugar de clases modelo tipadas, permitiendo flexibilidad total
- Las páginas CRUD comparten el mismo patrón de variables de estado: `registros`, `cargando`, `mostrarFormulario`, `editando`, `mensaje`, `exito`, `limite`
- Las páginas CRUD comparten el mismo ciclo de métodos: `OnInitializedAsync()`, `CargarRegistros()`, `NuevoRegistro()`, `EditarRegistro()`, `GuardarRegistro()`, `EliminarRegistro()`, `Cancelar()`
- Para agregar una nueva entidad CRUD, basta con: copiar una página existente, cambiar el nombre de la tabla, y ajustar los campos del formulario
- `SpService` es igualmente genérico: ejecuta cualquier SP por nombre con parámetros dinámicos

## Método de Verificación (Verification Method)
Análisis — Comparar el código de 2+ páginas CRUD (ej: Empresa.razor vs Producto.razor) para verificar que siguen el mismo patrón estructural y que la única diferencia son los campos de la entidad.

## Más Información (More Information)
- Servicio genérico: `Services/ApiService.cs` — 5 métodos públicos, todos parametrizados por `tabla`
- Servicio SP: `Services/SpService.cs` — 1 método público parametrizado por `nombreSP`
- Tipo de datos: `List<Dictionary<string, object?>>` como estructura universal
- JSON: `PropertyNameCaseInsensitive = true` para compatibilidad con propiedades en minúsculas de la API
- Patrón de página documentado en: `Parte5_CrudProducto.md` y `Parte6_CrudDemasTablas.md`
