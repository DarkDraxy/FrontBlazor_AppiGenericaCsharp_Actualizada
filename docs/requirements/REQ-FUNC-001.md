---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-001: Mostrar diagnóstico de conexión a base de datos

## Declaración (Statement)
El sistema shall mostrar en la página de inicio (Home) la información de diagnóstico de la conexión a la base de datos, incluyendo: proveedor, nombre de la base de datos, versión del servidor, dirección IP, puerto y usuario conectado.

## Justificación (Rationale)
El operador y el desarrollador necesitan verificar rápidamente que el frontend está comunicándose correctamente con la API y que esta, a su vez, está conectada a la base de datos esperada. Sin este diagnóstico, los errores de configuración se manifiestan de forma confusa en las páginas CRUD.

## Criterios de Aceptación (Acceptance Criteria)
- Al cargar la página `/`, se invoca `GET /api/diagnostico/conexion`
- Se muestran los 6 campos: proveedor, baseDatos, version, direccionIP, puerto, usuarioConectado
- Si la API no responde, se muestra un mensaje de error descriptivo en lugar de una excepción no controlada
- Los datos se presentan en un componente visual tipo tarjeta (card)

## Método de Verificación (Verification Method)
Demostración — Iniciar el sistema completo (BD + API + Frontend) y verificar que la página Home muestra los datos correctos del servidor SQL Server.

## Más Información (More Information)
- Página: `Components/Pages/Home.razor` (ruta: `/`)
- Servicio: `ApiService.ObtenerDiagnosticoAsync()`
- Endpoint API: `GET /api/diagnostico/conexion`
- Retorna: `Dictionary<string, string>` con las claves mencionadas
