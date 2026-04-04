---
status: "passed"
date: 2026-04-02
---

# REQ-SEC-001: Encriptación de campos sensibles delegada al backend

## Declaración (Statement)
El sistema shall delegar la encriptación de campos sensibles (contraseñas) al backend API mediante el parámetro query string `camposEncriptar`. El frontend shall nunca implementar lógica de encriptación propia; solo indica qué campos deben encriptarse y la API ejecuta el algoritmo server-side.

## Justificación (Rationale)
Implementar encriptación en el frontend (JavaScript/WebAssembly) expondría el algoritmo y las claves en el navegador. Delegar al backend garantiza que el algoritmo permanece opaco para el cliente y que la encriptación se aplica consistentemente independientemente del frontend utilizado.

## Criterios de Aceptación (Acceptance Criteria)
- La página Usuario ofrece un checkbox "Encriptar contraseña" que controla si se envía el parámetro `camposEncriptar`
- Al crear con encriptación: `POST /api/usuario?camposEncriptar=contrasena`
- Al actualizar con encriptación: `PUT /api/usuario/email/{valor}?camposEncriptar=contrasena`
- Sin checkbox activo, la contraseña se envía en texto plano (sin parámetro query)
- El frontend no contiene ninguna función de hash, cifrado ni manipulación criptográfica
- Las contraseñas encriptadas se almacenan en el campo VARCHAR(200) que permite hashes de longitud variable
- `ApiService.CrearAsync()` y `ActualizarAsync()` aceptan el parámetro opcional `camposEncriptar` y lo añaden como query string

## Método de Verificación (Verification Method)
Análisis + Demostración — Inspeccionar el código del frontend para verificar la ausencia de lógica criptográfica. Crear un usuario con encriptación activada y verificar que la contraseña se almacena como hash (visible en la lista).

## Más Información (More Information)
- Página: `Components/Pages/Usuario.razor` — variable `bool encriptarContrasena`
- Servicio: `ApiService.cs` — parámetro `string? camposEncriptar` en `CrearAsync()` y `ActualizarAsync()`
- El query string se construye como: `?camposEncriptar={valor}` y se concatena a la URL del endpoint
- La lógica de encriptación (algoritmo, salt, iteraciones) reside completamente en ApiGenericaCsharp (backend)
