# nb-shops

Sistema de tiendas **configurable desde dentro del juego**. Un wizard de 4 pasos permite al admin crear tiendas publicas (civil), restringidas a trabajo (business) o a gang (illegal) sin tocar ficheros de config.

---

## Caracteristicas

- 🧙 **Wizard admin de 4 pasos** — info → ubicacion → items → confirmar.
- 🏪 **Tres tipos de tienda** — `civil` (publica), `business` (job-lock), `illegal` (gang-lock).
- 📦 **Stock** — por item, ilimitado (`-1`) o finito con restock manual desde el panel.
- 💰 **Metodos de pago multiples** — cash, bank o **cualquier item como moneda** (oro, cripto, tokens custom).
- 🗂️ **Categorias personalizadas** por tienda.
- 🛒 **Carrito** — el jugador compone la compra antes de pagar.
- 🧭 **Markers + blips + peds** — NPCs con escenario, blips con colores por tipo, markers con animacion.
- 🖼️ **Iconos automaticos** — resuelve imagenes desde tu inventario (ox_inventory, qb-inventory, qs-inventory, origen_inventory) con override por item.
- 🛡️ **Job / Gang checks** — validacion client + server (anti-bypass).
- 🌐 **Multi-framework** — ESX + QBCore via nb-bridge.
- 🌎 **i18n** — EN / ES, extensible.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy (1.9+) / QBCore |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge |
| **Inventario** | ox_inventory, qb-inventory, qs-inventory, origen_inventory |

---

## Secciones

- **[Instalacion](instalacion.md)** — Requisitos, SQL, puesta en marcha.
- **[Configuracion](configuracion.md)** — `Config.*` (markers, blips, peds, iconos).
- **[Panel admin](panel-admin.md)** — El wizard de creacion paso a paso.
- **[Compra](compra.md)** — Como compra el jugador (marker / categoria / carrito).
- **[Exports y eventos](exports.md)** — API para consumir desde otros scripts.
