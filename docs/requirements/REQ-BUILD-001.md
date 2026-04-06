---
status: "passed"
date: 2026-04-02
---

# REQ-BUILD-001: Estructura del proyecto .NET 9.0 Blazor Server

## Declaración (Statement)
El sistema shall estar organizado como un proyecto Blazor Server sobre .NET 9.0 con la siguiente estructura: un archivo de proyecto `.csproj` sin paquetes NuGet externos, configuración en `appsettings.json`, dos servicios inyectados por DI (`ApiService`, `SpService`), un layout maestro con navegación, y páginas Razor organizadas bajo `Components/Pages/`.

## Justificación (Rationale)
La estructura estándar de un proyecto Blazor Server facilita la comprensión por parte de desarrolladores familiarizados con ASP.NET Core. La ausencia de paquetes externos reduce la complejidad de dependencias y demuestra que el SDK .NET 9.0 es suficiente para construir una aplicación CRUD completa.

## Criterios de Aceptación (Acceptance Criteria)
- El archivo `.csproj` especifica `<TargetFramework>net9.0</TargetFramework>` sin `<PackageReference>` externos
- `Program.cs` registra:
  - `AddRazorComponents().AddInteractiveServerComponents()` para Blazor Server
  - `HttpClient` con `BaseAddress` leído de `appsettings.json` (clave `ApiBaseUrl`)
  - `ApiService` y `SpService` como servicios de DI (`AddScoped`)
- `appsettings.json` contiene la clave `ApiBaseUrl` configurable (default: `http://localhost:5034`)
- `Properties/launchSettings.json` define el puerto del frontend (5200)
- Los servicios están en `Services/` (ApiService.cs, SpService.cs)
- Los componentes están en `Components/` con subcarpetas `Layout/` y `Pages/`
- Los assets estáticos están en `wwwroot/` (CSS, Bootstrap, favicon)
- El proyecto compila con `dotnet build` sin errores ni warnings

## Método de Verificación (Verification Method)
Inspección + Demostración — Verificar la estructura de directorios, inspeccionar el `.csproj` y ejecutar `dotnet build` para confirmar compilación limpia.

## Más Información (More Information)
- Archivo de proyecto: `FrontBlazor_AppiGenericaCsharp.csproj`
- Startup: `Program.cs` — configuración de DI, HttpClient y middleware
- Opciones habilitadas: `<Nullable>enable</Nullable>`, `<ImplicitUsings>enable</ImplicitUsings>`
- Render mode: `InteractiveServer` declarado en cada página con `@rendermode InteractiveServer`
- Ver estructura completa en Apéndice D del SRS
