---
status: "passed"
date: 2026-04-02
---

# REQ-FUNC-007: CRUD Usuario con opción de encriptar contraseña

## Declaración (Statement)
El sistema shall permitir listar, crear, editar y eliminar registros de la entidad Usuario (email PK, contraseña). Adicionalmente, shall ofrecer una opción (checkbox) para encriptar la contraseña en el servidor antes de almacenarla.

## Justificación (Rationale)
Los usuarios representan las cuentas de acceso del sistema. La contraseña es un campo sensible que debe poder almacenarse encriptada para proteger las credenciales. La encriptación se delega al backend (API) para no exponer el algoritmo en el frontend.

## Criterios de Aceptación (Acceptance Criteria)
- **Listar:** Al acceder a `/usuario`, se muestran todos los registros con columnas email y contraseña (la contraseña puede aparecer como hash si fue encriptada). El límite es configurable.
- **Crear:** Formulario con campos email (texto) y contraseña (tipo password). Checkbox "Encriptar contraseña" disponible. Al guardar:
  - Si el checkbox está activo: `POST /api/usuario?camposEncriptar=contrasena`
  - Si no: `POST /api/usuario` (contraseña en texto plano)
- **Editar:** Se cargan los datos actuales. Email deshabilitado (PK). Contraseña editable. Checkbox de encriptación disponible. Al guardar:
  - Si el checkbox está activo: `PUT /api/usuario/email/{valor}?camposEncriptar=contrasena`
  - Si no: `PUT /api/usuario/email/{valor}`
- **Eliminar:** Diálogo de confirmación. `DELETE /api/usuario/email/{valor}`.
- El parámetro `camposEncriptar` se pasa como query string a la API, que realiza la encriptación server-side.

## Método de Verificación (Verification Method)
Demostración — Crear un usuario con y sin encriptación. Verificar que:
1. Sin encriptación: la contraseña se almacena tal cual.
2. Con encriptación: la contraseña se muestra como hash en la lista.

## Más Información (More Information)
- Página: `Components/Pages/Usuario.razor` (ruta: `/usuario`)
- Servicio: `ApiService` — métodos `CrearAsync("usuario", datos, "contrasena")` y `ActualizarAsync("usuario", "email", valor, datos, "contrasena")`
- Tabla BD: `usuario` (email VARCHAR 100 PK, contrasena VARCHAR 200 NOT NULL)
- El campo VARCHAR 200 de contraseña permite almacenar tanto texto plano como hashes
- La lógica de encriptación reside completamente en la API backend, no en el frontend
