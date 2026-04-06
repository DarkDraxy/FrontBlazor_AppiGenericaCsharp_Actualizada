# Descripción de Diseño de Software (Software Design Description)
## FrontBlazor — Sistema CRUD Genérico con Facturación

Versión 1.0  
Preparado por Carlos Arturo Castro Castro  
Proyecto Educativo — Blazor Server + API REST Genérica  
2026-04-02

## Tabla de Contenido (Table of Contents)
<!-- TOC -->
* [1. Introducción (Introduction)](#1-introducción-introduction)
    * [1.1 Propósito (Purpose)](#11-propósito-purpose)
    * [1.2 Alcance (Scope)](#12-alcance-scope)
    * [1.3 Contexto del Diseño (Design Context)](#13-contexto-del-diseño-design-context)
    * [1.4 Referencias (References)](#14-referencias-references)
* [2. Arquitectura del Sistema (System Architecture)](#2-arquitectura-del-sistema-system-architecture)
    * [2.1 Vista General (Overview)](#21-vista-general-overview)
    * [2.2 Diagrama de Capas (Layer Diagram)](#22-diagrama-de-capas-layer-diagram)
    * [2.3 Flujo de Comunicación (Communication Flow)](#23-flujo-de-comunicación-communication-flow)
    * [2.4 Decisiones Arquitectónicas (Architectural Decisions)](#24-decisiones-arquitectónicas-architectural-decisions)
* [3. Diseño de Componentes (Component Design)](#3-diseño-de-componentes-component-design)
    * [3.1 Capa de Servicios (Services Layer)](#31-capa-de-servicios-services-layer)
    * [3.2 Capa de Presentación — Layout (Presentation Layer — Layout)](#32-capa-de-presentación--layout-presentation-layer--layout)
    * [3.3 Capa de Presentación — Páginas (Presentation Layer — Pages)](#33-capa-de-presentación--páginas-presentation-layer--pages)
    * [3.4 Configuración e Inicio (Configuration and Startup)](#34-configuración-e-inicio-configuration-and-startup)
* [4. Patrones de Diseño (Design Patterns)](#4-patrones-de-diseño-design-patterns)
    * [4.1 Patrón CRUD Simple (Simple CRUD Pattern)](#41-patrón-crud-simple-simple-crud-pattern)
    * [4.2 Patrón CRUD con Claves Foráneas (FK CRUD Pattern)](#42-patrón-crud-con-claves-foráneas-fk-crud-pattern)
    * [4.3 Patrón Master-Detail (Master-Detail Pattern)](#43-patrón-master-detail-master-detail-pattern)
* [5. Diseño de Datos (Data Design)](#5-diseño-de-datos-data-design)
    * [5.1 Modelo de Datos Dinámico (Dynamic Data Model)](#51-modelo-de-datos-dinámico-dynamic-data-model)
    * [5.2 Diagrama Entidad-Relación (Entity-Relationship Diagram)](#52-diagrama-entidad-relación-entity-relationship-diagram)
    * [5.3 Lógica de Negocio en Base de Datos (Database Business Logic)](#53-lógica-de-negocio-en-base-de-datos-database-business-logic)
* [6. Diseño de Interfaces (Interface Design)](#6-diseño-de-interfaces-interface-design)
    * [6.1 Interfaz con API REST (REST API Interface)](#61-interfaz-con-api-rest-rest-api-interface)
    * [6.2 Interfaz con JavaScript (JS Interop Interface)](#62-interfaz-con-javascript-js-interop-interface)
* [7. Consideraciones Transversales (Cross-Cutting Concerns)](#7-consideraciones-transversales-cross-cutting-concerns)
<!-- TOC -->

## Historial de Revisiones (Revision History)

| Nombre | Fecha      | Motivo del Cambio     | Versión |
|--------|------------|-----------------------|---------|
| Castro Castro, C. A. | 2026-04-02 | Documento inicial | 1.0 |

---

## 1. Introducción (Introduction)

### 1.1 Propósito (Purpose)

Este documento describe el diseño de software del frontend Blazor Server del Sistema CRUD Genérico con Facturación. Define la arquitectura, los componentes, los patrones de diseño y las interfaces que implementan los requerimientos especificados en el [SRS](srs.md).

Está dirigido a desarrolladores que necesitan entender, mantener o extender el sistema, y a evaluadores que revisan las decisiones de diseño.

### 1.2 Alcance (Scope)

Este SDD cubre exclusivamente el **frontend Blazor Server** (puerto 5200). No cubre el diseño interno de la API REST backend (ApiGenericaCsharp) ni la base de datos, los cuales se tratan como cajas negras con interfaces definidas.

### 1.3 Contexto del Diseño (Design Context)

El diseño está condicionado por las restricciones definidas en el SRS (Sección 2.3):

- Blazor Server con render mode InteractiveServer (conexión SignalR persistente)
- Sin paquetes NuGet externos — solo SDK .NET 9.0
- Datos manejados como `Dictionary<string, object?>` (sin clases modelo tipadas)
- Comunicación exclusiva vía HTTP/JSON con la API REST
- Bootstrap 5 como framework CSS

### 1.4 Referencias (References)

- Castro Castro, C. A. (2026). *Especificación de Requerimientos de Software — FrontBlazor*. [`srs.md`](srs.md)
- Castro Castro, C. A. (2026). *Requerimientos individuales*. [`requirements/`](requirements/)
- Castro Castro, C. A. (2026). *ApiGenericaCsharp — API REST genérica en C#*. GitHub. https://github.com/ccastro2050/ApiGenericaCsharp *(Backend API consumido por este frontend)*
- jam01. (2025). *Markdown Software Requirements Specification (MSRS) Template*. GitHub. https://github.com/jam01/SRS-Template *(Plantilla base para la documentación de requerimientos)*
- Microsoft. (2024). *ASP.NET Core Blazor*. https://learn.microsoft.com/aspnet/core/blazor
- Microsoft. (2024). *Blazor Server hosting model*. https://learn.microsoft.com/aspnet/core/blazor/hosting-models#blazor-server
- Microsoft. (2024). *Dependency injection in ASP.NET Core*. https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection
- Gamma, E., Helm, R., Johnson, R. & Vlissides, J. (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley. *(Referencia general de patrones)*

---

## 2. Arquitectura del Sistema (System Architecture)

### 2.1 Vista General (Overview)

El sistema sigue una **arquitectura de 3 capas** donde el frontend Blazor Server actúa como capa de presentación, la API REST como capa de lógica/datos, y SQL Server como capa de persistencia.

Dentro del frontend, la organización interna sigue un patrón de **2 capas**:

1. **Capa de Servicios** — `ApiService` y `SpService` encapsulan toda la comunicación HTTP
2. **Capa de Presentación** — Componentes Razor (páginas y layout) que consumen los servicios vía DI

### 2.2 Diagrama de Capas (Layer Diagram)

```
┌─────────────────────────────────────────────────────────────────┐
│                    NAVEGADOR (Browser)                          │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  DOM renderizado por Blazor Server vía SignalR            │  │
│  │  ← Solo HTML/CSS, no hay lógica de negocio en el browser │  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────────┘
                           │ SignalR (WebSocket)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              BLAZOR SERVER (.NET 9.0, Puerto 5200)             │
│                                                                 │
│  ┌─── Capa de Presentación ──────────────────────────────────┐  │
│  │                                                           │  │
│  │  Layout/                    Pages/                        │  │
│  │  ├── MainLayout.razor       ├── Home.razor        (/)     │  │
│  │  └── NavMenu.razor          ├── Empresa.razor             │  │
│  │                             ├── Persona.razor             │  │
│  │  App.razor                  ├── Producto.razor            │  │
│  │  Routes.razor               ├── Rol.razor                 │  │
│  │  _Imports.razor             ├── Ruta.razor                │  │
│  │                             ├── Usuario.razor             │  │
│  │                             ├── Cliente.razor             │  │
│  │                             ├── Vendedor.razor            │  │
│  │                             ├── Factura.razor             │  │
│  │                             └── Error.razor               │  │
│  └───────────────┬───────────────────────────────────────────┘  │
│                  │ @inject (DI)                                  │
│  ┌───────────────▼───────────────────────────────────────────┐  │
│  │  Capa de Servicios                                        │  │
│  │  ├── ApiService.cs    (CRUD genérico → HttpClient)        │  │
│  │  └── SpService.cs     (Stored Procs → HttpClient)         │  │
│  └───────────────┬───────────────────────────────────────────┘  │
│                  │ HttpClient                                    │
│  ┌───────────────▼───────────────────────────────────────────┐  │
│  │  Configuración                                            │  │
│  │  ├── Program.cs         (DI, HttpClient, middleware)      │  │
│  │  ├── appsettings.json   (ApiBaseUrl)                      │  │
│  │  └── launchSettings.json (Puerto 5200)                    │  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTP/JSON
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              API REST: ApiGenericaCsharp (Puerto 5034)          │
│              [Caja negra — fuera del alcance de este SDD]       │
└──────────────────────────┬──────────────────────────────────────┘
                           │ SQL / Stored Procedures
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              SQL Server — bdfacturas_sqlserver_local            │
│              [Caja negra — fuera del alcance de este SDD]       │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Flujo de Comunicación (Communication Flow)

#### Flujo CRUD típico (ej: crear un registro)

```
Usuario          Navegador          Blazor Server           API REST           SQL Server
  │                 │                    │                     │                    │
  │ Click "Guardar" │                    │                     │                    │
  │────────────────→│                    │                     │                    │
  │                 │ SignalR event      │                     │                    │
  │                 │───────────────────→│                     │                    │
  │                 │                    │ POST /api/{tabla}   │                    │
  │                 │                    │────────────────────→│                    │
  │                 │                    │                     │ INSERT INTO tabla   │
  │                 │                    │                     │───────────────────→│
  │                 │                    │                     │    resultado       │
  │                 │                    │                     │←───────────────────│
  │                 │                    │  JSON response      │                    │
  │                 │                    │←────────────────────│                    │
  │                 │  DOM diff (SignalR)│                     │                    │
  │                 │←───────────────────│                     │                    │
  │ UI actualizada  │                    │                     │                    │
  │←────────────────│                    │                     │                    │
```

#### Flujo Master-Detail (ej: crear factura)

```
Factura.razor                SpService                 API REST              SQL Server
     │                          │                         │                      │
     │ GuardarFactura()         │                         │                      │
     │ Serializar productos     │                         │                      │
     │ a JSON                   │                         │                      │
     │                          │                         │                      │
     │ EjecutarSpAsync(         │                         │                      │
     │   "sp_insertar_...",     │                         │                      │
     │   {cliente, vendedor,    │                         │                      │
     │    productos_json})      │                         │                      │
     │─────────────────────────→│                         │                      │
     │                          │ POST /api/proc/ejecutar │                      │
     │                          │────────────────────────→│                      │
     │                          │                         │ EXEC sp_insertar_... │
     │                          │                         │─────────────────────→│
     │                          │                         │                      │
     │                          │                         │  (SP inserta factura,│
     │                          │                         │   inserta productos, │
     │                          │                         │   triggers validan   │
     │                          │                         │   stock y calculan   │
     │                          │                         │   subtotales/total)  │
     │                          │                         │                      │
     │                          │                         │  JSON resultado      │
     │                          │                         │←─────────────────────│
     │                          │  JSON response          │                      │
     │                          │←────────────────────────│                      │
     │  (exito, resultados,     │                         │                      │
     │   mensaje)               │                         │                      │
     │←─────────────────────────│                         │                      │
     │                          │                         │                      │
     │ Mostrar mensaje éxito    │                         │                      │
     │ Volver a vista "listar"  │                         │                      │
```

### 2.4 Decisiones Arquitectónicas (Architectural Decisions)

| Decisión | Alternativas Consideradas | Justificación |
|----------|---------------------------|---------------|
| **Blazor Server** (no WASM) | Blazor WebAssembly, MVC, Razor Pages | Server permite ejecución C# en el servidor sin descargar el runtime al browser. Ideal para aplicaciones internas donde la latencia de red es mínima. |
| **Datos como Dictionary** (no modelos tipados) | Clases POCO por entidad, DTOs, records | El `Dictionary<string, object?>` permite un servicio genérico que opera sobre cualquier tabla sin crear una clase por entidad. Sacrifica type-safety por flexibilidad y eliminación de código repetitivo. |
| **Un servicio genérico** (no un servicio por entidad) | Repository pattern, servicio por tabla | `ApiService` con parámetro `tabla` reduce drásticamente la cantidad de código. La API ya es genérica, así que el frontend refleja la misma filosofía. |
| **Stored procedures** para Factura (no CRUD directo) | Transacciones HTTP múltiples, Unit of Work client-side | La lógica master-detail requiere atomicidad (insertar factura + N productos). Los SPs garantizan transaccionalidad en el servidor de BD. |
| **Sin paquetes NuGet** | MudBlazor, Blazorise, Radzen para UI | Mantiene la complejidad mínima y demuestra que el SDK .NET 9.0 + Bootstrap es suficiente para CRUD. |
| **JS Interop solo para confirm()** | Modal Blazor puro, componente de confirmación | `confirm()` nativo es la solución más simple para un diálogo sí/no. No justifica un componente completo para este caso de uso. |

---

## 3. Diseño de Componentes (Component Design)

### 3.1 Capa de Servicios (Services Layer)

#### 3.1.1 ApiService

**Archivo:** `Services/ApiService.cs`  
**Responsabilidad:** Encapsular todas las operaciones CRUD genéricas contra la API REST.  
**Inyección:** Registrado como `AddScoped<ApiService>` en `Program.cs`.  
**Dependencia:** `HttpClient` (inyectado por constructor).

```
┌──────────────────────────────────────────────────────────────┐
│                        ApiService                            │
├──────────────────────────────────────────────────────────────┤
│ - _http : HttpClient                                         │
├──────────────────────────────────────────────────────────────┤
│ + ListarAsync(tabla, limite?) : List<Dict>                   │
│ + CrearAsync(tabla, datos, camposEncriptar?) : (bool, string)│
│ + ActualizarAsync(tabla, clave, valor, datos, enc?) : (bool, │
│                                                      string)│
│ + EliminarAsync(tabla, clave, valor) : (bool, string)        │
│ + ObtenerDiagnosticoAsync() : Dict<string, string>?          │
├──────────────────────────────────────────────────────────────┤
│ Endpoints consumidos:                                        │
│   GET    /api/{tabla}[?limite=N]                             │
│   POST   /api/{tabla}[?camposEncriptar=campo]                │
│   PUT    /api/{tabla}/{clave}/{valor}[?camposEncriptar=campo]│
│   DELETE /api/{tabla}/{clave}/{valor}                        │
│   GET    /api/diagnostico/conexion                           │
└──────────────────────────────────────────────────────────────┘
```

**Detalles de implementación:**
- Serialización JSON con `PropertyNameCaseInsensitive = true` para compatibilidad con la API
- Los métodos de escritura (Crear, Actualizar) retornan tupla `(bool exito, string mensaje)`
- El método Eliminar retorna tupla `(bool exito, string mensaje)`
- Manejo de errores: try-catch sobre `HttpRequestException` con mensaje descriptivo
- El parámetro `camposEncriptar` se añade como query string cuando no es null

#### 3.1.2 SpService

**Archivo:** `Services/SpService.cs`  
**Responsabilidad:** Ejecutar stored procedures arbitrarios contra la API REST.  
**Inyección:** Registrado como `AddScoped<SpService>` en `Program.cs`.  
**Dependencia:** `HttpClient` (inyectado por constructor).

```
┌──────────────────────────────────────────────────────────────┐
│                         SpService                            │
├──────────────────────────────────────────────────────────────┤
│ - _http : HttpClient                                         │
├──────────────────────────────────────────────────────────────┤
│ + EjecutarSpAsync(nombreSP, parametros?) :                   │
│     (bool exito, List<Dict> resultados, string mensaje)      │
├──────────────────────────────────────────────────────────────┤
│ Endpoint consumido:                                          │
│   POST /api/procedimientos/ejecutarsp                        │
│   Payload: { "nombreSP": "...", ...parametros }              │
└──────────────────────────────────────────────────────────────┘
```

**Detalles de implementación:**
- Construye un `Dictionary` con `nombreSP` + parámetros aplanados al mismo nivel
- Retorna tupla de 3 elementos: éxito, lista de resultados y mensaje
- Maneja ambas variantes de la respuesta API: `"resultados"` y `"Resultados"` (case-insensitive)
- Los SPs de factura retornan JSON anidado que el frontend parsea en `Factura.razor`

### 3.2 Capa de Presentación — Layout (Presentation Layer — Layout)

#### 3.2.1 Jerarquía de Componentes

```
App.razor
 └── Routes.razor
      └── MainLayout.razor                    ← Layout maestro
           ├── NavMenu.razor                  ← Sidebar de navegación
           │    ├── NavLink → /               (Home)
           │    ├── NavLink → /empresa
           │    ├── NavLink → /persona
           │    ├── NavLink → /producto
           │    ├── NavLink → /rol
           │    ├── NavLink → /ruta
           │    ├── NavLink → /usuario
           │    ├── NavLink → /cliente
           │    ├── NavLink → /vendedor
           │    └── NavLink → /factura
           └── @Body                          ← Contenido de la página activa
```

#### 3.2.2 App.razor

Componente raíz que define el esqueleto HTML5:
- Incluye `<link>` a Bootstrap CSS y `app.css`
- Incluye `<script>` de Blazor Server (`_framework/blazor.web.js`)
- Renderiza `<Routes />`

#### 3.2.3 MainLayout.razor

Layout maestro que define la estructura visual:
- Sidebar fijo a la izquierda con `NavMenu`
- Área de contenido principal (`@Body`) a la derecha
- Responsive vía clases Bootstrap

#### 3.2.4 NavMenu.razor

Barra de navegación lateral con 10 `<NavLink>`:
- Usa `Match="NavLinkMatch.All"` para resaltar el enlace activo
- Colapsable en pantallas pequeñas (toggle button)

### 3.3 Capa de Presentación — Páginas (Presentation Layer — Pages)

#### 3.3.1 Clasificación de Páginas

| Página | Ruta | Categoría | Servicios Inyectados |
|--------|------|-----------|----------------------|
| Home.razor | `/` | Dashboard | ApiService |
| Empresa.razor | `/empresa` | CRUD Simple | ApiService, IJSRuntime |
| Persona.razor | `/persona` | CRUD Simple | ApiService, IJSRuntime |
| Producto.razor | `/producto` | CRUD Simple | ApiService, IJSRuntime |
| Rol.razor | `/rol` | CRUD Simple | ApiService, IJSRuntime |
| Ruta.razor | `/ruta` | CRUD Simple | ApiService, IJSRuntime |
| Usuario.razor | `/usuario` | CRUD + Encriptación | ApiService, IJSRuntime |
| Cliente.razor | `/cliente` | CRUD + FK | ApiService, IJSRuntime |
| Vendedor.razor | `/vendedor` | CRUD + FK | ApiService, IJSRuntime |
| Factura.razor | `/factura` | Master-Detail | ApiService, SpService, IJSRuntime |
| Error.razor | `/Error` | Error | — |

#### 3.3.2 Anatomía de una Página CRUD

Todas las páginas CRUD comparten la siguiente estructura interna:

```
@page "/entidad"
@rendermode InteractiveServer
@inject ApiService Api
@inject IJSRuntime JSRuntime

┌─ Variables de estado ─────────────────────────┐
│ registros : List<Dictionary<string, object?>> │
│ cargando  : bool                              │
│ mostrarFormulario : bool                      │
│ editando  : bool                              │
│ mensaje   : string                            │
│ exito     : bool                              │
│ limite    : int?                              │
│ campo*    : (campos específicos de la entidad)│
└───────────────────────────────────────────────┘

┌─ Métodos del ciclo de vida ───────────────────┐
│ OnInitializedAsync() → CargarRegistros()      │
│ CargarRegistros()    → Api.ListarAsync(tabla)  │
│ NuevoRegistro()      → mostrarFormulario=true  │
│ EditarRegistro(reg)  → cargar campos, edit=true│
│ GuardarRegistro()    → Crear o Actualizar      │
│ EliminarRegistro(reg)→ confirm + Eliminar      │
│ Cancelar()           → mostrarFormulario=false │
└───────────────────────────────────────────────┘

┌─ Markup Razor ────────────────────────────────┐
│ @if (cargando) → Spinner                      │
│ @if (mensaje)  → Alert éxito/error            │
│ @if (!mostrarFormulario) → TABLA + botones    │
│ @else          → FORMULARIO + Guardar/Cancelar│
└───────────────────────────────────────────────┘
```

### 3.4 Configuración e Inicio (Configuration and Startup)

#### 3.4.1 Program.cs — Pipeline de Inicio

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Blazor Server
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();

// 2. HttpClient con BaseAddress configurable
builder.Services.AddScoped(sp => new HttpClient {
    BaseAddress = new Uri(builder.Configuration["ApiBaseUrl"]
        ?? "http://localhost:5034")
});

// 3. Servicios de negocio
builder.Services.AddScoped<ApiService>();
builder.Services.AddScoped<SpService>();

var app = builder.Build();

// 4. Middleware pipeline
app.UseStaticFiles();
app.UseAntiforgery();
app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();

app.Run();
```

#### 3.4.2 Diagrama de Dependencias (DI)

```
Program.cs (registra)
    │
    ├── HttpClient ──────────┐
    │   BaseAddress: ApiBaseUrl│
    │                        │
    ├── ApiService ──────────┤ (depende de HttpClient)
    │                        │
    └── SpService ───────────┘ (depende de HttpClient)

Páginas Razor (consumen vía @inject):
    ├── @inject ApiService Api
    ├── @inject SpService Sp        (solo Factura)
    └── @inject IJSRuntime JSRuntime (para confirm)
```

---

## 4. Patrones de Diseño (Design Patterns)

### 4.1 Patrón CRUD Simple (Simple CRUD Pattern)

**Aplica a:** Empresa, Persona, Producto, Rol, Ruta

**Diagrama de estados de la vista:**

```
            NuevoRegistro()          GuardarRegistro()
                  │                        │
     ┌────────────▼────────────┐    ┌──────▼───────┐
     │                         │    │              │
 ───→│   VISTA LISTA           │───→│ VISTA FORM   │
     │                         │←───│ (nuevo/editar)│
     │ • Tabla de registros    │    │              │
     │ • Botones Nuevo/Editar/ │    │ • Campos     │
     │   Eliminar              │    │ • Guardar    │
     │ • Control de límite     │    │ • Cancelar   │
     └─────────────────────────┘    └──────────────┘
           Cancelar() / GuardarRegistro() exitoso
```

**Flujo de datos:**

```
                     ApiService
                         │
    ListarAsync("tabla") │ CrearAsync / ActualizarAsync / EliminarAsync
           │             │             │
           ▼             │             ▼
    ┌──────────┐         │      ┌──────────┐
    │ registros│ ◀───────┘      │ campos*  │
    │ (lista)  │                │ (form)   │
    └──────────┘                └──────────┘
         │                           │
         ▼                           ▼
    Vista Lista              Vista Formulario
```

### 4.2 Patrón CRUD con Claves Foráneas (FK CRUD Pattern)

**Aplica a:** Cliente, Vendedor

**Extiende** el patrón CRUD Simple con:

1. **Carga de tablas relacionadas** en `OnInitializedAsync()`:
   - Cliente: carga `persona` y `empresa`
   - Vendedor: carga `persona`

2. **Dropdowns** en el formulario (`<select>`) poblados con las listas relacionadas

3. **Resolución de nombres** en la vista lista:
   - Métodos `ObtenerNombrePersona(codigo)` y `ObtenerNombreEmpresa(codigo)`
   - Buscan en las listas cargadas el nombre correspondiente al código FK

```
OnInitializedAsync()
    │
    ├── CargarRegistros()      → Lista principal
    ├── Api.ListarAsync("persona")  → Lista para dropdown
    └── Api.ListarAsync("empresa")  → Lista para dropdown (Cliente)
```

### 4.3 Patrón Master-Detail (Master-Detail Pattern)

**Aplica a:** Factura (+ ProductosPorFactura)

**Diagrama de estados (3 vistas):**

```
                                    VerFactura()
    ┌───────────────┐          ┌───────────────┐
    │               │─────────→│               │
    │  VISTA LISTA  │          │ VISTA DETALLE │
    │  "listar"     │←─────────│ "ver"         │
    │               │  Volver()│               │
    └───────┬───────┘          └───────┬───────┘
            │                          │
            │ MostrarFormularioNueva() │ EditarFactura()
            │                          │
            ▼                          ▼
    ┌──────────────────────────────────┐
    │        VISTA FORMULARIO          │
    │        "formulario"              │
    │                                  │
    │  ┌─ Cabecera ─────────────────┐  │
    │  │ Dropdown Cliente           │  │
    │  │ Dropdown Vendedor          │  │
    │  └────────────────────────────┘  │
    │                                  │
    │  ┌─ Detalle (filas dinámicas) ┐  │
    │  │ [Producto ▼] [Cantidad] [-]│  │
    │  │ [Producto ▼] [Cantidad] [-]│  │
    │  │ [+ Agregar Producto]       │  │
    │  └────────────────────────────┘  │
    │                                  │
    │  [Guardar]  [Cancelar]           │
    └──────────────────────────────────┘
            │
            │ GuardarFactura() / Cancelar()
            ▼
    Regresa a VISTA LISTA
```

**Clases auxiliares internas:**

```
┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐
│   ProductoFila  │  │   ClienteInfo   │  │  VendedorInfo    │
├─────────────────┤  ├─────────────────┤  ├──────────────────┤
│ Codigo : string │  │ Id : int        │  │ Id : int         │
│ Cantidad : int  │  │ Nombre : string │  │ Nombre : string  │
└─────────────────┘  │ Credito : string│  │ Carnet : string  │
                     └─────────────────┘  └──────────────────┘
```

**Flujo de serialización de productos:**

```
List<ProductoFila>  ──serialize──→  JSON string  ──POST──→  API  ──→  SP
                                   [{"codigo":"PR001",      ejecutarsp
                                     "cantidad":2},
                                    {"codigo":"PR003",
                                     "cantidad":3}]
```

---

## 5. Diseño de Datos (Data Design)

### 5.1 Modelo de Datos Dinámico (Dynamic Data Model)

El frontend **no define clases modelo** por entidad. Todos los datos se manejan como:

```csharp
List<Dictionary<string, object?>>   // Lista de registros
Dictionary<string, object?>          // Un registro individual
```

**Ventajas:**
- Un solo servicio genérico para todas las entidades
- No requiere actualizar el frontend al agregar columnas en la BD
- Flexibilidad total para manejar cualquier tabla

**Desventajas:**
- Sin validación de tipos en tiempo de compilación
- Acceso a campos por string (propenso a errores de typo)
- IntelliSense limitado en las páginas Razor

**Excepción:** La página `Factura.razor` define 3 clases internas tipadas (`ProductoFila`, `ClienteInfo`, `VendedorInfo`) para manejar la complejidad del formulario master-detail con filas dinámicas.

### 5.2 Diagrama Entidad-Relación (Entity-Relationship Diagram)

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ empresa  │     │ persona  │     │ producto │
│──────────│     │──────────│     │──────────│
│ codigo PK│◀──┐ │ codigo PK│◀──┐ │ codigo PK│◀────────────┐
│ nombre   │   │ │ nombre   │   │ │ nombre   │             │
└──────────┘   │ │ email    │   │ │ stock    │             │
               │ │ telefono │   │ │ valorunit│             │
               │ └──────────┘   │ └──────────┘             │
               │                │                          │
               │    ┌───────────┴──────────┐               │
               │    │                      │               │
          ┌────┴────▼──┐          ┌────────▼──┐            │
          │ cliente    │          │ vendedor  │            │
          │────────────│          │───────────│            │
          │ id PK (AI) │◀──┐     │ id PK (AI)│◀──┐       │
          │ credito    │   │     │ carnet    │   │       │
          │ fkcodpers  │   │     │ direccion │   │       │
          │ fkcodempre │   │     │ fkcodpers │   │       │
          └────────────┘   │     └───────────┘   │       │
                           │                     │       │
                     ┌─────┴─────────────────────┘       │
                     │                                   │
               ┌─────▼──────┐     ┌──────────────────┐   │
               │ factura    │     │productosporfact  │   │
               │────────────│     │──────────────────│   │
               │ numero PK  │◀───→│ fknumfactura FK  │───┘
               │ fecha      │     │ fkcodproducto FK │
               │ total      │     │ cantidad         │
               │ fkidcliente│     │ subtotal         │
               │ fkidvended │     └──────────────────┘
               └────────────┘        (PK compuesta)

Tablas auxiliares:
┌───────────┐  ┌──────────┐  ┌──────────┐
│ rol       │  │rol_usuar │  │ rutarol  │
│───────────│  │──────────│  │──────────│
│ id PK(AI) │◀→│fkidrol FK│  │ ruta PK  │
│ nombre    │  │fkemail FK│  │ rol  PK  │
└───────────┘  └────┬─────┘  └──────────┘
                    │
               ┌────▼─────┐
               │ usuario  │
               │──────────│
               │ email PK │
               │contrasena│
               └──────────┘
```

### 5.3 Lógica de Negocio en Base de Datos (Database Business Logic)

La lógica de negocio de facturación reside en la BD, no en el frontend:

#### Triggers (automáticos en productosporfactura)

| Trigger | Evento | Acciones |
|---------|--------|----------|
| `trg_prodfact_insert` | AFTER INSERT | 1. Valida stock ≥ cantidad (ROLLBACK si no). 2. Calcula `subtotal = cantidad × valorunitario`. 3. Decrementa `producto.stock`. 4. Recalcula `factura.total` = Σ subtotales. |
| `trg_prodfact_update` | AFTER UPDATE | 1. Valida stock (considerando devolución de cantidad anterior). 2. Recalcula subtotal. 3. Ajusta stock (devuelve anterior + decrementa nuevo). 4. Recalcula total. |
| `trg_prodfact_delete` | AFTER DELETE | 1. Restaura `producto.stock += cantidad`. 2. Recalcula `factura.total`. |

#### Stored Procedures (invocados desde Factura.razor vía SpService)

| SP | Operación | Lógica |
|----|-----------|--------|
| `sp_insertar_factura_y_productosporfactura` | CREATE | Inserta factura → parsea JSON de productos → inserta cada línea (triggers calculan subtotal y stock) → retorna JSON resultado |
| `sp_consultar_factura_y_productosporfactura` | READ | Consulta factura + JOIN productos con nombres → retorna JSON anidado |
| `sp_listar_facturas_y_productosporfactura` | READ ALL | Consulta todas las facturas + productos con nombres de cliente/vendedor → retorna JSON array |
| `sp_actualizar_factura_y_productosporfactura` | UPDATE | Actualiza cabecera → DELETE productos antiguos (trigger restaura stock) → INSERT nuevos (trigger decrementa stock) → retorna JSON |
| `sp_borrar_factura_y_productosporfactura` | DELETE | DELETE factura → CASCADE elimina productos (trigger restaura stock) → retorna JSON confirmación |

---

## 6. Diseño de Interfaces (Interface Design)

### 6.1 Interfaz con API REST (REST API Interface)

#### 6.1.1 Contratos de ApiService

| Método | HTTP | URL | Request Body | Response |
|--------|------|-----|-------------|----------|
| `ListarAsync` | GET | `/api/{tabla}[?limite=N]` | — | `List<Dict>` (JSON array) |
| `CrearAsync` | POST | `/api/{tabla}[?camposEncriptar=c]` | `Dict` (JSON object) | `{ "mensaje": "..." }` |
| `ActualizarAsync` | PUT | `/api/{tabla}/{clave}/{valor}[?camposEncriptar=c]` | `Dict` (JSON object) | `{ "mensaje": "..." }` |
| `EliminarAsync` | DELETE | `/api/{tabla}/{clave}/{valor}` | — | `{ "mensaje": "..." }` |
| `ObtenerDiagnosticoAsync` | GET | `/api/diagnostico/conexion` | — | `{ "proveedor", "baseDatos", "version", ... }` |

#### 6.1.2 Contrato de SpService

| Método | HTTP | URL | Request Body | Response |
|--------|------|-----|-------------|----------|
| `EjecutarSpAsync` | POST | `/api/procedimientos/ejecutarsp` | `{ "nombreSP": "...", ...params }` | `{ "resultados": [...], "mensaje": "..." }` |

#### 6.1.3 Configuración JSON

```csharp
JsonSerializerOptions options = new() {
    PropertyNameCaseInsensitive = true  // API retorna en minúsculas
};
```

### 6.2 Interfaz con JavaScript (JS Interop Interface)

Única interacción con JavaScript:

```csharp
// Diálogo de confirmación antes de eliminar
bool confirmar = await JSRuntime.InvokeAsync<bool>(
    "confirm",
    "¿Está seguro de que desea eliminar este registro?"
);
```

No se definen funciones JavaScript adicionales. Solo se usa la función nativa `confirm()` del navegador.

---

## 7. Consideraciones Transversales (Cross-Cutting Concerns)

### 7.1 Manejo de Errores (Error Handling)

| Capa | Estrategia |
|------|-----------|
| Servicios (`ApiService`, `SpService`) | try-catch sobre `HttpRequestException`. Retorna tupla `(false, mensaje_error)` en lugar de propagar excepciones. |
| Páginas (`.razor`) | Evalúan `bool exito` retornado por los servicios. Muestran `mensaje` en alert Bootstrap rojo (error) o verde (éxito). |
| Blazor Server | `Error.razor` como página de error global. Logging por defecto de ASP.NET Core. |

### 7.2 Estado de la Aplicación (Application State)

- **Sin estado compartido** entre páginas. Cada página carga sus datos en `OnInitializedAsync()`.
- **Sin persistencia client-side** (no localStorage, no sessionStorage, no cookies de datos).
- **Estado de la UI** vive en las variables de instancia de cada componente Razor, mantenido en memoria del servidor vía la conexión SignalR.
- Al recargar la página (F5), el estado se reinicia completamente.

### 7.3 Rendimiento (Performance)

- Todas las operaciones HTTP son `async/await` (ver [REQ-PERF-001](requirements/REQ-PERF-001.md))
- El parámetro `limite` en `ListarAsync` permite controlar la cantidad de registros cargados
- No hay caché de datos: cada navegación recarga desde la API
- Las listas de FK (persona, empresa) se cargan una vez por inicialización de página

### 7.4 Seguridad (Security)

- Sin autenticación ni autorización en el frontend
- Encriptación delegada al backend (ver [REQ-SEC-001](requirements/REQ-SEC-001.md))
- JS Interop limitado a `confirm()` (superficie de ataque mínima)
- No se almacenan credenciales ni tokens en el frontend
