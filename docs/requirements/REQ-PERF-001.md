---
status: "passed"
date: 2026-04-02
---

# REQ-PERF-001: Operaciones asíncronas sin bloqueo de UI

## Declaración (Statement)
El sistema shall ejecutar todas las operaciones de comunicación con la API REST de forma asíncrona (async/await), evitando el bloqueo del hilo de renderizado de Blazor Server. Durante la ejecución de operaciones, la interfaz shall mostrar un indicador de carga.

## Justificación (Rationale)
Blazor Server mantiene una conexión SignalR con el navegador. Si el hilo de renderizado se bloquea con operaciones síncronas, la UI se congela para el usuario. Las operaciones asíncronas permiten que la interfaz permanezca responsiva mientras se espera la respuesta de la API.

## Criterios de Aceptación (Acceptance Criteria)
- Todos los métodos que invocan la API usan `async Task` y `await`
- `OnInitializedAsync()` carga datos iniciales sin bloquear el renderizado
- Mientras se ejecuta una operación HTTP, la variable `cargando = true` activa el indicador visual
- Al completarse la operación, `cargando = false` oculta el indicador y se muestra el resultado
- No existen llamadas `.Result` o `.Wait()` sobre Tasks en ninguna página ni servicio
- `HttpClient.GetAsync()`, `PostAsync()`, `PutAsync()`, `DeleteAsync()` se invocan siempre con `await`

## Método de Verificación (Verification Method)
Análisis — Inspeccionar el código de `ApiService.cs`, `SpService.cs` y todas las páginas `.razor` para verificar que toda operación HTTP usa async/await y que no hay bloqueos síncronos.

## Más Información (More Information)
- Servicios: `ApiService.cs` y `SpService.cs` — todos los métodos públicos son `async Task<T>`
- Páginas: Todas usan `protected override async Task OnInitializedAsync()` para carga inicial
- Patrón: `cargando = true` → `await operación` → `cargando = false` → `StateHasChanged()` implícito
- Referencia: Microsoft. (2024). *ASP.NET Core Blazor — Call a web API*. https://learn.microsoft.com/aspnet/core/blazor/call-web-api
