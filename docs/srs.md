# Especificación de Requerimientos de Software (Software Requirements Specification)
## FrontBlazor — Sistema CRUD Genérico con Facturación

Versión 1.0  
Preparado por Carlos Arturo Castro Castro  
Proyecto Educativo — Blazor Server + API REST Genérica  
2026-04-02

## Tabla de Contenido (Table of Contents)
<!-- TOC -->
* [1. Introducción (Introduction)](#1-introducción-introduction)
    * [1.1 Propósito del Documento (Document Purpose)](#11-propósito-del-documento-document-purpose)
    * [1.2 Alcance del Producto (Product Scope)](#12-alcance-del-producto-product-scope)
    * [1.3 Definiciones, Acrónimos y Abreviaturas (Definitions, Acronyms, and Abbreviations)](#13-definiciones-acrónimos-y-abreviaturas-definitions-acronyms-and-abbreviations)
    * [1.4 Referencias (References)](#14-referencias-references)
    * [1.5 Visión General del Documento (Document Overview)](#15-visión-general-del-documento-document-overview)
* [2. Visión General del Producto (Product Overview)](#2-visión-general-del-producto-product-overview)
    * [2.1 Perspectiva del Producto (Product Perspective)](#21-perspectiva-del-producto-product-perspective)
    * [2.2 Funciones del Producto (Product Functions)](#22-funciones-del-producto-product-functions)
    * [2.3 Restricciones del Producto (Product Constraints)](#23-restricciones-del-producto-product-constraints)
    * [2.4 Características de los Usuarios (User Characteristics)](#24-características-de-los-usuarios-user-characteristics)
    * [2.5 Supuestos y Dependencias (Assumptions and Dependencies)](#25-supuestos-y-dependencias-assumptions-and-dependencies)
    * [2.6 Distribución de Requerimientos (Apportioning of Requirements)](#26-distribución-de-requerimientos-apportioning-of-requirements)
* [3. Requerimientos (Requirements)](#3-requerimientos-requirements)
    * [3.1 Interfaces Externas (External Interfaces)](#31-interfaces-externas-external-interfaces)
    * [3.2 Funcionales (Functional)](#32-funcionales-functional)
    * [3.3 Calidad de Servicio (Quality of Service)](#33-calidad-de-servicio-quality-of-service)
    * [3.4 Cumplimiento (Compliance)](#34-cumplimiento-compliance)
    * [3.5 Diseño e Implementación (Design and Implementation)](#35-diseño-e-implementación-design-and-implementation)
* [4. Verificación (Verification)](#4-verificación-verification)
* [5. Apéndices (Appendixes)](#5-apéndices-appendixes)
<!-- TOC -->

## Historial de Revisiones (Revision History)

| Name | Date       | Reason For Changes                            | Version |
|------|------------|-----------------------------------------------|---------|
| Castro Castro, C. A. | 2026-04-02 | Documento inicial | 1.0 |

## 1. Introducción (Introduction)

Este documento especifica los requerimientos del sistema FrontBlazor, una aplicación web Blazor Server que implementa operaciones CRUD genéricas sobre múltiples entidades de un sistema de facturación comercial. El frontend consume una API REST genérica (ApiGenericaCsharp) que gestiona la persistencia en SQL Server.

### 1.1 Propósito del Documento (Document Purpose)

Este SRS define los requerimientos funcionales y no funcionales del frontend Blazor y su interacción con la API REST backend. Está dirigido a:

- **Desarrolladores** que implementan o extienden las páginas CRUD y servicios.
- **Profesor evaluador** que revisa la arquitectura, patrones y funcionamiento del sistema.
- **Estudiantes** que usan el proyecto como tutorial de referencia para Blazor Server + API REST.

El documento define *qué* debe hacer el sistema, no *cómo* está implementado. Los detalles de diseño se encuentran en el SDD complementario.

### 1.2 Alcance del Producto (Product Scope)

**FrontBlazor — Sistema CRUD Generico con Facturacion v1.0**

Aplicación web Blazor Server (.NET 9.0) que proporciona una interfaz de usuario interactiva para gestionar un sistema de facturación comercial con 10 entidades. El sistema cubre tres niveles de complejidad CRUD:

1. **CRUD simple** — Entidades independientes: Empresa, Persona, Producto, Rol, Ruta, Usuario
2. **CRUD con claves foráneas** — Entidades dependientes: Cliente (→ Persona, Empresa), Vendedor (→ Persona)
3. **Master-Detail** — Factura con detalle de productos, gestionado por stored procedures con control de stock vía triggers

**Propósito:** Demostrar patrones de consumo de API REST desde Blazor Server, incluyendo operaciones CRUD genéricas, manejo de relaciones FK, encriptación de campos, y transacciones master-detail con stored procedures.

**Incluye:** Frontend web, consumo de API REST, páginas CRUD para 10 entidades, dashboard de diagnóstico, navegación, validaciones de formulario.  
**Excluye:** Backend API (proyecto separado ApiGenericaCsharp), base de datos (SQL Server externo), autenticación/autorización de usuarios, despliegue en producción.

### 1.3 Definiciones, Acrónimos y Abreviaturas (Definitions, Acronyms, and Abbreviations)

| Término             | Definición                                                                                               |
|---------------------|----------------------------------------------------------------------------------------------------------|
| Blazor Server       | Framework de Microsoft para construir UIs web interactivas con C# usando una conexión SignalR al servidor |
| CRUD                | Create, Read, Update, Delete — las cuatro operaciones básicas de persistencia                            |
| API REST            | Interfaz de programación basada en HTTP con recursos identificados por URL                                |
| FK                  | Foreign Key — clave foránea que referencia la clave primaria de otra tabla                                |
| Master-Detail       | Patrón donde un registro padre (factura) tiene múltiples registros hijos (productos por factura)          |
| SP / Stored Procedure | Procedimiento almacenado en SQL Server que ejecuta lógica de negocio en la base de datos               |
| Trigger             | Disparador de base de datos que ejecuta lógica automática al insertar, actualizar o eliminar registros    |
| SignalR             | Biblioteca de comunicación en tiempo real que Blazor Server usa para mantener el estado de la UI          |
| DI                  | Dependency Injection — patrón de inyección de dependencias usado en ASP.NET Core                         |
| SRS                 | Software Requirements Specification — Especificación de Requerimientos de Software                       |
| SDD                 | Software Design Description — Descripción de Diseño de Software                                          |
| PK                  | Primary Key — clave primaria que identifica de forma única un registro                                    |
| JSON                | JavaScript Object Notation — formato de intercambio de datos entre frontend y API                         |
| HttpClient          | Clase .NET para realizar peticiones HTTP                                                                  |
| IJSRuntime          | Interfaz de Blazor para invocar funciones JavaScript desde C# (ej: diálogos confirm)                     |

### 1.4 Referencias (References)

#### Normas y estándares

- IEEE. (1998). *IEEE Std 830-1998: IEEE Recommended Practice for Software Requirements Specifications*. Institute of Electrical and Electronics Engineers. https://doi.org/10.1109/IEEESTD.1998.88286 *(Reemplazado por ISO/IEC/IEEE 29148; incluido como referencia histórica)*
- ISO/IEC/IEEE. (2018). *ISO/IEC/IEEE 29148:2018: Systems and software engineering — Life cycle processes — Requirements engineering* (2.ª ed.). International Organization for Standardization. https://www.iso.org/standard/72089.html *(Edición vigente; reemplaza a IEEE 830 y a la edición 2011)*
- van Lamsweerde, A. (2009). *Requirements Engineering: From System Goals to UML Models to Software Specifications*. John Wiley & Sons. *(Base de la taxonomía de requerimientos no funcionales utilizada en la plantilla)*

#### Documentación del framework

- Microsoft. (2024). *ASP.NET Core Blazor*. Microsoft Learn. https://learn.microsoft.com/aspnet/core/blazor
- Microsoft. (2024). *What's new in .NET 9*. Microsoft Learn. https://learn.microsoft.com/dotnet/core/whats-new/dotnet-9
- Microsoft. (2024). *ASP.NET Core SignalR overview*. Microsoft Learn. https://learn.microsoft.com/aspnet/core/signalr/introduction
- Microsoft. (2024). *System.Text.Json overview*. Microsoft Learn. https://learn.microsoft.com/dotnet/standard/serialization/system-text-json/overview

#### Documentación del proyecto (informativa)

- Castro Castro, C. A. (2026). *Parte 1 — Conceptos fundamentales de Blazor*. [`../Parte1_ConceptosFundamentales.md`](../Parte1_ConceptosFundamentales.md)
- Castro Castro, C. A. (2026). *Parte 2 — Crear proyecto y configuración*. [`../Parte2_CrearProyectoYConfiguracion.md`](../Parte2_CrearProyectoYConfiguracion.md)
- Castro Castro, C. A. (2026). *Parte 3 — ApiService: consumo de API REST*. [`../Parte3_ApiService.md`](../Parte3_ApiService.md)
- Castro Castro, C. A. (2026). *Parte 4 — Layout y navegación*. [`../Parte4_LayoutYNavegacion.md`](../Parte4_LayoutYNavegacion.md)
- Castro Castro, C. A. (2026). *Parte 5 — CRUD Producto*. [`../Parte5_CrudProducto.md`](../Parte5_CrudProducto.md)
- Castro Castro, C. A. (2026). *Parte 6 — CRUD de las demás tablas*. [`../Parte6_CrudDemasTablas.md`](../Parte6_CrudDemasTablas.md)
- Castro Castro, C. A. (2026). *Parte 7 — Verificación final*. [`../Parte7_VerificacionFinal.md`](../Parte7_VerificacionFinal.md)
- Castro Castro, C. A. (2026). *Guía de uso — Entidades*. [`../GUIA_USO_ENTIDADES.md`](../GUIA_USO_ENTIDADES.md)
- Castro Castro, C. A. (2026). *Guía de uso — Procedimientos almacenados*. [`../GUIA_USO_PROCEDIMIENTOS.md`](../GUIA_USO_PROCEDIMIENTOS.md)
- Castro Castro, C. A. (2026). *Tutorial maestro-detalle*. [`../TUTORIAL_MAESTRO_DETALLE.md`](../TUTORIAL_MAESTRO_DETALLE.md)

#### Artefactos normativos del proyecto

- Castro Castro, C. A. (2026). *ApiGenericaCsharp — API REST genérica en C#*. GitHub. https://github.com/ccastro2050/ApiGenericaCsharp *(Backend API consumido por este frontend)*
- Castro Castro, C. A. (2026). *Script de base de datos — SQL Server*. [`../script_bd/bdfacturas_sqlserver.sql`](../script_bd/bdfacturas_sqlserver.sql)
- Castro Castro, C. A. (2026). *Script de base de datos — PostgreSQL*. [`../script_bd/bdfacturas_postgres.sql`](../script_bd/bdfacturas_postgres.sql)
- Castro Castro, C. A. (2026). *Script de base de datos — MySQL / MariaDB*. [`../script_bd/bdfacturas_mysql_mariadb.sql`](../script_bd/bdfacturas_mysql_mariadb.sql)
- jam01. (2025). *Markdown Software Requirements Specification (MSRS) Template*. GitHub. https://github.com/jam01/SRS-Template

### 1.5 Visión General del Documento (Document Overview)

- **Sección 1 (Introducción):** Propósito, alcance, glosario, referencias y convenciones del documento.
- **Sección 2 (Visión General del Producto):** Contexto del sistema, funciones principales, restricciones, usuarios, supuestos y plan de entregas.
- **Sección 3 (Requerimientos):** Requerimientos verificables organizados por interfaz, funcionalidad, calidad y diseño. Cada requerimiento referencia un archivo individual en [`requirements/`](requirements/).
- **Sección 4 (Verificación):** Matriz de trazabilidad con método de verificación y estado por requerimiento.
- **Sección 5 (Apéndices):** Diagramas de arquitectura, esquema de base de datos y estructura del proyecto.

Convenciones: Los IDs de requerimiento siguen el esquema `REQ-[AREA]-[NNN]` donde AREA ∈ {FUNC, INT, PERF, SEC, BUILD, MAINT}.

## 2. Visión General del Producto (Product Overview)

### 2.1 Perspectiva del Producto (Product Perspective)

FrontBlazor es el **frontend** de un sistema de dos capas: Blazor Server (presentación) + API REST genérica (lógica y datos). No reemplaza ningún sistema existente; es un proyecto educativo standalone.

El frontend se comunica exclusivamente vía HTTP/JSON con la API backend, sin acceso directo a la base de datos. La arquitectura completa es:

```
┌──────────────────────────────────────┐
│       BLAZOR SERVER (Puerto 5200)    │
│                                      │
│  Paginas (.razor)                    │
│    ├── 6 CRUD simple                 │
│    ├── 2 CRUD con FK                 │
│    ├── 1 Master-Detail (Factura)     │
│    └── 1 Dashboard (Home)            │
│                                      │
│  Servicios                           │
│    ├── ApiService  (CRUD generico)   │
│    └── SpService   (Stored Procs)    │
└──────────────┬───────────────────────┘
               │ HTTP (GET/POST/PUT/DELETE)
               │ JSON payloads
               ▼
┌──────────────────────────────────────┐
│  API REST: ApiGenericaCsharp (:5034) │
│                                      │
│  /api/{tabla}                        │
│  /api/{tabla}/{clave}/{valor}        │
│  /api/diagnostico/conexion           │
│  /api/procedimientos/ejecutarsp      │
└──────────────┬───────────────────────┘
               │ SQL / Stored Procedures
               ▼
┌──────────────────────────────────────┐
│  SQL Server                          │
│  bdfacturas_sqlserver_local          │
│                                      │
│  12 tablas, 3 triggers, 5 SPs       │
└──────────────────────────────────────┘
```

**Dependencia crítica:** El frontend requiere que la API REST esté en ejecución en el puerto configurado (`ApiBaseUrl` en `appsettings.json`). Sin la API, ninguna operación funciona.

### 2.2 Funciones del Producto (Product Functions)

**Dashboard y diagnóstico:**
- Mostrar información de conexión a la base de datos (proveedor, versión, servidor, usuario)

**CRUD simple (6 entidades independientes):**
- Listar registros con límite opcional
- Crear registro con formulario
- Editar registro existente
- Eliminar registro con confirmación

**CRUD con claves foráneas (2 entidades):**
- Todo lo del CRUD simple, más:
- Cargar y mostrar dropdowns de entidades relacionadas (Persona, Empresa)
- Resolver nombres de FK para mostrar en la lista

**Master-Detail — Factura (1 entidad compuesta):**
- Listar facturas con nombre de cliente/vendedor y conteo de productos
- Ver detalle de factura con tabla de productos
- Crear factura con selección de cliente, vendedor y filas dinámicas de productos
- Editar factura completa (cabecera + productos)
- Eliminar factura con cascada automática de productos

**CRUD con encriptación (Usuario):**
- Crear/actualizar usuario con opción de encriptar contraseña en el servidor

**Transversal:**
- Navegación lateral con acceso a todas las entidades
- Mensajes de éxito/error después de cada operación
- Diálogos de confirmación antes de eliminar
- Indicador de carga durante operaciones asíncronas

### 2.3 Restricciones del Producto (Product Constraints)

| Restricción                       | Tipo     | Descripción                                                                      |
|-----------------------------------|----------|----------------------------------------------------------------------------------|
| Framework                         | Mandatorio | Blazor Server sobre .NET 9.0                                                    |
| Render mode                       | Mandatorio | InteractiveServer en todas las páginas (conexión SignalR persistente)            |
| Backend API                       | Mandatorio | ApiGenericaCsharp en puerto configurable (default 5034)                          |
| Base de datos                     | Mandatorio | SQL Server con esquema `bdfacturas_sqlserver_local`                             |
| Sin paquetes NuGet externos       | Mandatorio | Solo dependencias implícitas del SDK .NET 9.0                                   |
| CSS                               | Mandatorio | Bootstrap 5 (incluido en wwwroot/lib)                                           |
| Sin acceso directo a BD           | Mandatorio | El frontend nunca ejecuta SQL; toda persistencia pasa por la API REST           |
| Datos dinámicos                   | Mandatorio | Todas las entidades se manejan como `Dictionary<string, object?>`, sin clases modelo tipadas |
| JS Interop                        | Limitado   | Solo para diálogos `confirm()` vía `IJSRuntime`                                 |
| Navegador                         | Informativo| Cualquier navegador moderno con soporte WebSocket (SignalR)                     |

### 2.4 Características de los Usuarios (User Characteristics)

| Rol                | Descripción                                                                                              | Frecuencia   |
|--------------------|----------------------------------------------------------------------------------------------------------|--------------|
| Operador comercial | Usuario final que gestiona empresas, personas, productos, clientes, vendedores y facturas vía la UI web. Conocimiento básico de navegación web. | Frecuente    |
| Administrador      | Gestiona usuarios y roles, incluyendo encriptación de contraseñas. Conocimiento intermedio.               | Ocasional    |
| Estudiante dev     | Desarrollador que estudia el código como tutorial de Blazor Server + API REST. Nivel principiante-intermedio en C#/.NET. | Referencia   |
| Profesor evaluador | Revisa arquitectura, patrones, código y funcionamiento del sistema completo.                              | Evaluación   |

### 2.5 Supuestos y Dependencias (Assumptions and Dependencies)

| Supuesto / Dependencia                                     | Tipo        | Impacto si falla                                          |
|------------------------------------------------------------|-------------|-----------------------------------------------------------|
| .NET 9.0 SDK instalado en el entorno de desarrollo         | Dependencia | El proyecto no compila                                    |
| SQL Server accesible con la BD `bdfacturas_sqlserver_local` | Dependencia | La API no puede conectarse; el frontend muestra errores   |
| API REST (ApiGenericaCsharp) corriendo en puerto 5034      | Dependencia | Todas las operaciones CRUD fallan                         |
| Script SQL ejecutado previamente (tablas, triggers, SPs)   | Dependencia | Operaciones fallan o retornan errores de objetos faltantes |
| Un solo usuario a la vez opera el sistema                  | Supuesto    | No hay control de concurrencia; posibles conflictos de datos |
| Volumen de datos pequeño (< 1000 registros por tabla)      | Supuesto    | Sin paginación server-side; el límite es opcional y client-side |
| Navegador moderno con WebSocket                            | Dependencia | Blazor Server requiere SignalR (WebSocket); sin soporte no funciona |
| Los datos de ejemplo del script SQL están cargados         | Supuesto    | Los dropdowns de FK pueden estar vacíos al inicio         |
| Puerto 5200 disponible para el frontend                    | Dependencia | El frontend no inicia                                     |

### 2.6 Distribución de Requerimientos (Apportioning of Requirements)

Los requerimientos se organizan por nivel de complejidad y entidad:

| Grupo                        | Entidades                                           | Requerimientos      | Prioridad |
|------------------------------|-----------------------------------------------------|---------------------|-----------|
| Dashboard                    | Home (diagnostico)                                  | REQ-FUNC-001        | Alta      |
| CRUD simple                  | Empresa, Persona, Producto, Rol, Ruta               | REQ-FUNC-002 a 006  | Alta      |
| CRUD con encriptación        | Usuario                                             | REQ-FUNC-007        | Alta      |
| CRUD con FK                  | Cliente, Vendedor                                   | REQ-FUNC-008 a 009  | Alta      |
| Master-Detail                | Factura + ProductosPorFactura                       | REQ-FUNC-010 a 014  | Alta      |
| Interfaz                     | Layout, NavMenu, formularios                        | REQ-INT-001 a 003   | Alta      |
| Rendimiento                  | Tiempos de respuesta, async                         | REQ-PERF-001        | Media     |
| Seguridad                    | Encriptación, confirmaciones                        | REQ-SEC-001         | Media     |
| Build / Estructura           | Proyecto .NET 9, servicios, configuración           | REQ-BUILD-001       | Alta      |
| Mantenibilidad               | Patrón CRUD reutilizable, servicios genéricos       | REQ-MAINT-001       | Media     |

## 3. Requerimientos (Requirements)

Cada requerimiento sigue el esquema de ID: `REQ-[AREA]-[NNN]` donde AREA ∈ {FUNC, INT, PERF, SEC, BUILD, MAINT}.

Todos los requerimientos individuales se encuentran en el directorio [`requirements/`](requirements/) como archivos markdown independientes.

### 3.1 Interfaces Externas (External Interfaces)

#### 3.1.1 Interfaces de Usuario (User Interfaces)

- [REQ-INT-001](requirements/REQ-INT-001.md) — Layout principal con navegación lateral
- [REQ-INT-002](requirements/REQ-INT-002.md) — Formularios CRUD con alternancia lista/formulario
- [REQ-INT-003](requirements/REQ-INT-003.md) — Mensajes de retroalimentación y diálogos de confirmación

#### 3.1.2 Interfaces de Hardware (Hardware Interfaces)

No aplica. Aplicación web sin interacción con hardware.

#### 3.1.3 Interfaces de Software (Software Interfaces)

El frontend se integra con una sola interfaz de software: la API REST ApiGenericaCsharp.

| Interfaz             | Protocolo | Endpoints                                                                  |
|----------------------|-----------|----------------------------------------------------------------------------|
| CRUD genérico        | HTTP/JSON | `GET/POST /api/{tabla}`, `PUT/DELETE /api/{tabla}/{clave}/{valor}`         |
| Diagnóstico          | HTTP/JSON | `GET /api/diagnostico/conexion`                                            |
| Stored Procedures    | HTTP/JSON | `POST /api/procedimientos/ejecutarsp`                                      |

Autenticación: ninguna (API abierta en entorno local de desarrollo).

### 3.2 Funcionales (Functional)

**Dashboard**

- [REQ-FUNC-001](requirements/REQ-FUNC-001.md) — Mostrar diagnóstico de conexión a base de datos

**CRUD Simple (entidades independientes)**

- [REQ-FUNC-002](requirements/REQ-FUNC-002.md) — CRUD Empresa (código PK, nombre)
- [REQ-FUNC-003](requirements/REQ-FUNC-003.md) — CRUD Persona (código PK, nombre, email, teléfono)
- [REQ-FUNC-004](requirements/REQ-FUNC-004.md) — CRUD Producto (código PK, nombre, stock, valorunitario)
- [REQ-FUNC-005](requirements/REQ-FUNC-005.md) — CRUD Rol (id auto PK, nombre)
- [REQ-FUNC-006](requirements/REQ-FUNC-006.md) — CRUD Ruta (ruta PK, descripción)

**CRUD con encriptación**

- [REQ-FUNC-007](requirements/REQ-FUNC-007.md) — CRUD Usuario con opción de encriptar contraseña

**CRUD con claves foráneas**

- [REQ-FUNC-008](requirements/REQ-FUNC-008.md) — CRUD Cliente (FK → Persona, Empresa)
- [REQ-FUNC-009](requirements/REQ-FUNC-009.md) — CRUD Vendedor (FK → Persona)

**Master-Detail (Factura)**

- [REQ-FUNC-010](requirements/REQ-FUNC-010.md) — Listar facturas con resumen
- [REQ-FUNC-011](requirements/REQ-FUNC-011.md) — Ver detalle de factura con productos
- [REQ-FUNC-012](requirements/REQ-FUNC-012.md) — Crear factura con productos (SP insert)
- [REQ-FUNC-013](requirements/REQ-FUNC-013.md) — Editar factura con productos (SP update)
- [REQ-FUNC-014](requirements/REQ-FUNC-014.md) — Eliminar factura con cascada (SP delete)

### 3.3 Calidad de Servicio (Quality of Service)

#### 3.3.1 Rendimiento (Performance)

- [REQ-PERF-001](requirements/REQ-PERF-001.md) — Operaciones asíncronas sin bloqueo de UI

#### 3.3.2 Seguridad (Security)

- [REQ-SEC-001](requirements/REQ-SEC-001.md) — Encriptación de campos sensibles delegada al backend

#### 3.3.3 Fiabilidad (Reliability)

El sistema muestra mensajes de error descriptivos cuando la API no responde o retorna errores. Los try-catch en los servicios capturan excepciones HTTP y las presentan al usuario sin terminar la aplicación.

#### 3.3.4 Disponibilidad (Availability)

No aplica formalmente. Sistema de desarrollo local. La disponibilidad depende de que la API y SQL Server estén en ejecución.

#### 3.3.5 Observabilidad (Observability)

El sistema muestra el estado de conexión a la base de datos en la página Home (diagnóstico). No hay logging estructurado ni métricas en el frontend; se utiliza el nivel de log por defecto de ASP.NET Core (`Information` / `Warning`).

### 3.4 Cumplimiento (Compliance)

Proyecto académico. Debe cumplir con:
- Compilación exitosa en .NET 9.0
- Uso exclusivo de bibliotecas del SDK (sin NuGet externo)
- Scripts SQL compatibles con SQL Server, PostgreSQL y MySQL/MariaDB

### 3.5 Diseño e Implementación (Design and Implementation)

#### 3.5.1 Instalación (Installation)

Prerrequisitos: .NET 9.0 SDK, SQL Server con BD creada, API ApiGenericaCsharp en ejecución.

```bash
# 1. Crear BD y ejecutar script
sqlcmd -S localhost -i script_bd/bdfacturas_sqlserver.sql

# 2. Iniciar API backend (proyecto separado)
cd ../ApiGenericaCsharp && dotnet run

# 3. Iniciar frontend Blazor
cd FrontBlazor_AppiGenericaCsharpTutorial && dotnet run
# Abrir http://localhost:5200
```

#### 3.5.2 Construcción y Entrega (Build and Delivery)

- [REQ-BUILD-001](requirements/REQ-BUILD-001.md) — Estructura del proyecto .NET 9.0 Blazor Server

#### 3.5.3 Distribución (Distribution)

No aplica. Aplicación local de desarrollo.

#### 3.5.4 Mantenibilidad (Maintainability)

- [REQ-MAINT-001](requirements/REQ-MAINT-001.md) — Patrón CRUD reutilizable con servicio genérico

#### 3.5.5 Reusabilidad (Reusability)

Los servicios `ApiService` y `SpService` son completamente genéricos y reutilizables en cualquier proyecto que consuma la misma API REST. Las páginas CRUD siguen un patrón repetible para nuevas entidades.

#### 3.5.6 Portabilidad (Portability)

El frontend es portable a cualquier plataforma con .NET 9.0 (Windows, Linux, macOS). La BD tiene scripts para SQL Server, PostgreSQL y MySQL/MariaDB.

#### 3.5.7 Costo (Cost)

Sin costo. Herramientas gratuitas: .NET SDK, SQL Server Express/Developer, Visual Studio Code.

#### 3.5.8 Plazos (Deadline)

No definido. Proyecto educativo sin fecha límite formal.

#### 3.5.9 Prueba de Concepto (Proof of Concept)

No aplica.

#### 3.5.10 Gestión de Cambios (Change Management)

Los tutoriales (Parte1 a Parte7) documentan la construcción incremental del proyecto. Cambios se rastrean vía el historial del repositorio.

## 4. Verificación (Verification)

Cada requerimiento se verifica mediante uno de los siguientes métodos:
- **Demostración:** Ejecución de la funcionalidad en el sistema y observación del resultado.
- **Análisis:** Inspección del código fuente para verificar propiedades estructurales o de diseño.
- **Inspección:** Revisión de artefactos (archivos, configuración, estructura de directorios).

### 4.1 Matriz de Trazabilidad — Requerimientos Funcionales

| ID | Título | Método | Artefacto | Estado | Evidencia |
|----|--------|--------|-----------|--------|-----------|
| [REQ-FUNC-001](requirements/REQ-FUNC-001.md) | Diagnóstico de conexión BD | Demostración | Home.razor | Passed | Tarjeta con datos del servidor |
| [REQ-FUNC-002](requirements/REQ-FUNC-002.md) | CRUD Empresa | Demostración | Empresa.razor | Passed | 4 operaciones CRUD verificadas |
| [REQ-FUNC-003](requirements/REQ-FUNC-003.md) | CRUD Persona | Demostración | Persona.razor | Passed | 4 operaciones CRUD verificadas |
| [REQ-FUNC-004](requirements/REQ-FUNC-004.md) | CRUD Producto | Demostración | Producto.razor | Passed | Campos numéricos persisten correctamente |
| [REQ-FUNC-005](requirements/REQ-FUNC-005.md) | CRUD Rol | Demostración | Rol.razor | Passed | PK auto-incremental generada |
| [REQ-FUNC-006](requirements/REQ-FUNC-006.md) | CRUD Ruta | Demostración | Ruta.razor | Passed | 4 operaciones CRUD verificadas |
| [REQ-FUNC-007](requirements/REQ-FUNC-007.md) | CRUD Usuario + encriptación | Demostración | Usuario.razor | Passed | Contraseña almacenada como hash |
| [REQ-FUNC-008](requirements/REQ-FUNC-008.md) | CRUD Cliente (FK) | Demostración | Cliente.razor | Passed | Dropdowns FK, nombres resueltos en lista |
| [REQ-FUNC-009](requirements/REQ-FUNC-009.md) | CRUD Vendedor (FK) | Demostración | Vendedor.razor | Passed | Dropdown persona, nombre resuelto |
| [REQ-FUNC-010](requirements/REQ-FUNC-010.md) | Listar facturas | Demostración | Factura.razor | Passed | Tabla con resumen y conteo productos |
| [REQ-FUNC-011](requirements/REQ-FUNC-011.md) | Ver detalle factura | Demostración | Factura.razor | Passed | Cabecera + tabla de productos |
| [REQ-FUNC-012](requirements/REQ-FUNC-012.md) | Crear factura con productos | Demostración | Factura.razor | Passed | Factura creada, stock decrementado |
| [REQ-FUNC-013](requirements/REQ-FUNC-013.md) | Editar factura con productos | Demostración | Factura.razor | Passed | Productos reemplazados, stock ajustado |
| [REQ-FUNC-014](requirements/REQ-FUNC-014.md) | Eliminar factura con cascada | Demostración | Factura.razor | Passed | Factura eliminada, stock restaurado |

### 4.2 Matriz de Trazabilidad — Requerimientos de Interfaz

| ID | Título | Método | Artefacto | Estado | Evidencia |
|----|--------|--------|-----------|--------|-----------|
| [REQ-INT-001](requirements/REQ-INT-001.md) | Layout con navegación lateral | Demostración | MainLayout.razor, NavMenu.razor | Passed | 10 enlaces funcionales, activo resaltado |
| [REQ-INT-002](requirements/REQ-INT-002.md) | Alternancia lista/formulario | Demostración | Todas las páginas CRUD | Passed | Transiciones lista ↔ formulario correctas |
| [REQ-INT-003](requirements/REQ-INT-003.md) | Mensajes y confirmaciones | Demostración | Todas las páginas CRUD | Passed | Alertas Bootstrap + confirm() JS |

### 4.3 Matriz de Trazabilidad — Calidad de Servicio

| ID | Título | Método | Artefacto | Estado | Evidencia |
|----|--------|--------|-----------|--------|-----------|
| [REQ-PERF-001](requirements/REQ-PERF-001.md) | Operaciones async sin bloqueo | Análisis | ApiService.cs, SpService.cs, *.razor | Passed | Todos los métodos usan async/await |
| [REQ-SEC-001](requirements/REQ-SEC-001.md) | Encriptación delegada al backend | Análisis + Demostración | Usuario.razor, ApiService.cs | Passed | Sin lógica criptográfica en frontend |

### 4.4 Matriz de Trazabilidad — Diseño e Implementación

| ID | Título | Método | Artefacto | Estado | Evidencia |
|----|--------|--------|-----------|--------|-----------|
| [REQ-BUILD-001](requirements/REQ-BUILD-001.md) | Estructura .NET 9.0 Blazor Server | Inspección + Demostración | .csproj, Program.cs, estructura dirs | Passed | `dotnet build` sin errores, sin NuGet externo |
| [REQ-MAINT-001](requirements/REQ-MAINT-001.md) | Patrón CRUD reutilizable | Análisis | ApiService.cs, páginas CRUD | Passed | Servicio genérico, patrón replicable verificado |

## 5. Apéndices (Appendixes)

### A. Esquema de Base de Datos (resumen)

| Tabla               | PK                          | Tipo PK      | Campos                                        | FKs                                  |
|---------------------|-----------------------------|--------------|-----------------------------------------------|--------------------------------------|
| empresa             | codigo                      | VARCHAR(10)  | nombre                                        | —                                    |
| persona             | codigo                      | VARCHAR(10)  | nombre, email, telefono                       | —                                    |
| producto            | codigo                      | VARCHAR(10)  | nombre, stock, valorunitario                  | —                                    |
| rol                 | id                          | INT IDENTITY | nombre                                        | —                                    |
| ruta                | ruta                        | VARCHAR(100) | descripcion                                   | —                                    |
| usuario             | email                       | VARCHAR(100) | contrasena                                    | —                                    |
| cliente             | id                          | INT IDENTITY | credito, fkcodpersona, fkcodempresa           | persona.codigo, empresa.codigo       |
| vendedor            | id                          | INT IDENTITY | carnet, direccion, fkcodpersona               | persona.codigo                       |
| factura             | numero                      | INT IDENTITY | fecha, total, fkidcliente, fkidvendedor       | cliente.id, vendedor.id              |
| productosporfactura | fknumfactura + fkcodproducto | Compuesta    | cantidad, subtotal                            | factura.numero, producto.codigo      |
| rol_usuario         | fkemail + fkidrol            | Compuesta    | —                                             | usuario.email, rol.id                |
| rutarol             | ruta + rol                   | Compuesta    | —                                             | —                                    |

### B. Stored Procedures

| SP                                              | Operacion | Parametros principales                              |
|-------------------------------------------------|-----------|------------------------------------------------------|
| sp_insertar_factura_y_productosporfactura       | INSERT    | fkidcliente, fkidvendedor, productos (JSON)          |
| sp_consultar_factura_y_productosporfactura      | SELECT    | numero                                               |
| sp_listar_facturas_y_productosporfactura        | SELECT    | (ninguno)                                            |
| sp_actualizar_factura_y_productosporfactura     | UPDATE    | numero, fkidcliente, fkidvendedor, productos (JSON)  |
| sp_borrar_factura_y_productosporfactura         | DELETE    | numero                                               |

### C. Triggers

| Trigger                | Tabla               | Evento        | Accion                                                  |
|------------------------|---------------------|---------------|---------------------------------------------------------|
| trg_prodfact_insert    | productosporfactura | AFTER INSERT  | Valida stock, calcula subtotal, decrementa stock, recalcula total |
| trg_prodfact_update    | productosporfactura | AFTER UPDATE  | Valida stock, recalcula subtotal, ajusta stock, recalcula total   |
| trg_prodfact_delete    | productosporfactura | AFTER DELETE  | Restaura stock, recalcula total                         |

### D. Estructura del Proyecto

```
FrontBlazor_AppiGenericaCsharpTutorial/
├── FrontBlazor_AppiGenericaCsharp.csproj    (.NET 9.0)
├── Program.cs                                (Startup, DI, HttpClient)
├── appsettings.json                          (ApiBaseUrl: localhost:5034)
├── appsettings.Development.json
├── Properties/launchSettings.json            (Puerto 5200)
├── Services/
│   ├── ApiService.cs                         (CRUD generico via HTTP)
│   └── SpService.cs                          (Ejecutor de stored procedures)
├── Components/
│   ├── App.razor                             (HTML root)
│   ├── Routes.razor                          (Router config)
│   ├── _Imports.razor                        (Global usings)
│   ├── Layout/
│   │   ├── MainLayout.razor                  (Layout master con sidebar)
│   │   └── NavMenu.razor                     (Navegacion: 10 enlaces)
│   └── Pages/
│       ├── Home.razor          /             (Dashboard diagnostico)
│       ├── Empresa.razor       /empresa      (CRUD simple)
│       ├── Persona.razor       /persona      (CRUD simple)
│       ├── Producto.razor      /producto     (CRUD simple)
│       ├── Rol.razor           /rol          (CRUD simple)
│       ├── Ruta.razor          /ruta         (CRUD simple)
│       ├── Usuario.razor       /usuario      (CRUD + encriptacion)
│       ├── Cliente.razor       /cliente      (CRUD + FK)
│       ├── Vendedor.razor      /vendedor     (CRUD + FK)
│       ├── Factura.razor       /factura      (Master-Detail + SPs)
│       └── Error.razor         /Error        (Pagina de error)
├── wwwroot/
│   ├── app.css                               (Estilos custom)
│   ├── favicon.png
│   └── lib/bootstrap/                        (Bootstrap 5)
├── script_bd/
│   ├── bdfacturas_sqlserver.sql              (Esquema SQL Server)
│   ├── bdfacturas_postgres.sql               (Esquema PostgreSQL)
│   └── bdfacturas_mysql_mariadb.sql          (Esquema MySQL / MariaDB)
└── docs/                                     (Documentacion de ingenieria)
    ├── srs.md                                (Este documento)
    └── requirements/                         (Requerimientos individuales)
```
