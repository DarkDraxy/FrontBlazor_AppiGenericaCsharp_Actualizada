---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-008: CRUD Cliente con claves foráneas

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Cliente, compuesta por id (PK, INT IDENTITY), crédito (DECIMAL 18,2), fkcodpersona (FK → persona.codigo, obligatoria) y fkcodempresa (FK → empresa.codigo, opcional). En la lista y en el formulario, las claves foráneas se resolverán mostrando el nombre de la persona y empresa asociadas.

## Justificación (Rationale)
El cliente es una entidad dependiente que vincula a una persona natural con una empresa y un límite de crédito. Es la primera entidad del sistema que introduce relaciones FK, lo que requiere cargar datos de tablas relacionadas para poblar dropdowns y resolver nombres en la vista de lista.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/cliente`, se muestran los registros con columnas: id, crédito, nombre de persona (resuelto desde FK), nombre de empresa (resuelto desde FK o vacío si es null). Límite configurable.
- **Crear:** Formulario con campos:
  - Crédito (numérico, step 0.01)
  - Persona (dropdown `<select>` con las personas cargadas desde `GET /api/persona`, obligatorio)
  - Empresa (dropdown `<select>` con las empresas cargadas desde `GET /api/empresa`, opcional)
  - Al guardar: `POST /api/cliente`
- **Editar:** Se cargan los datos actuales. Id deshabilitado (PK auto). Los dropdowns preseleccionan los valores actuales de FK. Al guardar: `PUT /api/cliente/id/{valor}`.
- **Eliminar:** Diálogo de confirmación. `DELETE /api/cliente/id/{valor}`.
- Al inicializar la página (`OnInitializedAsync`), se cargan las listas de personas y empresas para los dropdowns.
- Los métodos `ObtenerNombrePersona(codigo)` y `ObtenerNombreEmpresa(codigo)` resuelven los nombres a partir de las listas cargadas.

## Método de Verificación (Verification Method)
Demostración — Crear un cliente seleccionando persona y empresa desde los dropdowns. Verificar que la lista muestra los nombres resueltos y no los códigos FK. Editar cambiando la persona asociada. Eliminar y verificar que desaparece de la lista.

## Más Información (More Information)
- Página: `Components/Pages/Cliente.razor` (ruta: `/cliente`)
- Servicio: `ApiService` con tabla `"cliente"`, más `ListarAsync("persona")` y `ListarAsync("empresa")` para los dropdowns
- Tabla BD: `cliente` (id INT IDENTITY PK, credito DECIMAL 18,2 DEFAULT 0, fkcodpersona VARCHAR 10 FK → persona.codigo NOT NULL, fkcodempresa VARCHAR 10 FK → empresa.codigo NULLABLE)
- Patrón: CRUD con FK — extiende el patrón simple con carga de tablas relacionadas al inicializar y resolución de nombres en la vista
