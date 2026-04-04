---
status: "passed"
date: 2026-04-02
---

# REQ-INT-001: Layout principal con navegación lateral

## Declaración (Statement)
El sistema shall presentar un layout maestro con una barra de navegación lateral (sidebar) que contenga enlaces a las 10 páginas del sistema: Home, Empresa, Persona, Producto, Rol, Ruta, Usuario, Cliente, Vendedor y Facturas. El enlace activo se resaltará visualmente.

## Justificación (Rationale)
La navegación lateral persistente permite al operador acceder a cualquier entidad del sistema desde cualquier página sin pasos intermedios, reduciendo el número de clics y mejorando la experiencia de usuario.

## Criterios de Aceptación (Acceptance Criteria)
- El sidebar es visible en todas las páginas (definido en `MainLayout.razor`)
- Contiene exactamente 10 enlaces de navegación con las rutas: `/`, `/empresa`, `/persona`, `/producto`, `/rol`, `/ruta`, `/usuario`, `/cliente`, `/vendedor`, `/factura`
- Cada enlace muestra un texto descriptivo de la entidad
- El enlace correspondiente a la página actual se resalta usando la clase CSS de Bootstrap `active` (via `NavLink` con `Match`)
- El sidebar es colapsable en pantallas pequeñas (responsive con Bootstrap)
- El área de contenido principal ocupa el espacio restante a la derecha del sidebar

## Método de Verificación (Verification Method)
Demostración — Navegar a cada una de las 10 páginas usando los enlaces del sidebar. Verificar que el enlace activo se resalta y que el contenido de cada página se carga correctamente.

## Más Información (More Information)
- Layout: `Components/Layout/MainLayout.razor`
- Navegación: `Components/Layout/NavMenu.razor`
- Componente Blazor: `<NavLink>` con atributos `href` y `Match`
- CSS: Bootstrap 5 (incluido en `wwwroot/lib/bootstrap/`)
