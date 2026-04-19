# nb-restaurants

Sistema de gestion de restaurantes **configurable in-game** con recetas, mesas de crafteo, autoservicio al cliente, facturacion por camarero y lavado de dinero opcional. Funciona **standalone** y se integra de forma **opt-in** con el resto de la suite Neenbyss (nb-jobmanagers, nb-billings, nb-shops, nb-consumibles).

> Compra y descarga: [neenbyss.tebex.io](https://neenbyss.tebex.io/). Usa la version v1.0.0 o superior.

---

## Caracteristicas

- 🏪 **Multi-restaurante** — cada uno asociado a un trabajo (job), con logo URL, toggles y modo de facturacion independiente.
- 🎛️ **Panel admin in-game** — CRUD de restaurantes, biblioteca de plantillas de recetas, editor de idiomas dinamicos y ajustes globales.
- 👥 **Panel staff in-game** — cada empleado ve su propio restaurante: marcadores, recetas e importacion desde plantillas.
- 🍽️ **6 tipos de marcador** — boss menu, billing desk, crafting station, self-service, warehouse, supplier.
- 🥘 **Plantillas de recetas** — biblioteca inicial con 15 clasicos (burger, pizza, taco, mojito...). El staff importa y luego personaliza.
- 🧑‍🍳 **Crafting** — consume ingredientes del almacen + personales, aplica cooldown y entrega el item al empleado.
- 🛒 **Autoservicio** — el boss abre/cierra, el staff rellena slots con stock finito o infinito, el cliente compra cash/bank.
- 🧾 **Billing desk** — el camarero factura al cliente cercano. Routing configurable entre `native`, `nb-billings` o `external`.
- 💼 **Boss menu** — empleados, sociedad (deposit/withdraw), toggle del autoservicio y panel de lavado.
- 🕵️ **Lavado de dinero opcional** — toggle admin por restaurante, tax configurable, anti-spam, webhook Discord.
- 🔌 **Integraciones opt-in** — auto-deteccion de nb-jobmanagers, nb-billings, nb-shops, nb-consumibles.
- 🌐 **Multi-framework** — ESX Legacy + QBCore via nb-bridge.
- 🌎 **i18n** — EN / ES estaticos + traducciones dinamicas editables desde el panel.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy (1.9+) / QBCore |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge |
| **Inventario** | ox_inventory (recomendado para stashes), qb-inventory, qs-inventory, origen_inventory |
| **Jobs (opcional)** | nb-jobmanagers |
| **Billing (opcional)** | nb-billings / esx_billing / qb-billing / okokBilling |
| **Shops (opcional)** | nb-shops |
| **Consumibles (opcional)** | nb-consumibles |

---

## Secciones

- **[Instalacion](instalacion.md)** — Requisitos, SQL, orden en `server.cfg`.
- **[Configuracion](configuracion.md)** — Todas las opciones de `shared/config.lua`.
- **[Flujos](flujos.md)** — Admin crea restaurante, staff configura, boss gestiona, cliente compra.
- **[Panel admin](panel-admin.md)** — Restaurantes, plantillas, idiomas, ajustes.
- **[Panel staff](panel-staff.md)** — Overview, marcadores (por kind), recetas, importar plantilla.
- **[Exports](exports.md)** — API server para integrar desde otros scripts.
- **[Changelog](changelog.md)** — Historial de releases.
