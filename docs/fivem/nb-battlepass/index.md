# nb-battlepass

Battlepass estandalone para FiveM con **pase gratis** y **pase premium**. Configurable al 100% **in-game** desde el panel admin, y extensible desde codigo via un archivo publico sin tocar la logica de negocio.

---

## Caracteristicas

- 🎟️ **Free + Premium** tracks con recompensas independientes por nivel.
- 🛠️ **Admin panel in-game** — crea seasons, niveles y recompensas sin reiniciar el servidor.
- 🧩 **Tipos de recompensa** de fabrica: items, dinero (cash/bank/black), vehiculos.
- 🪄 **Custom rewards** — define tus propios tipos en un archivo publico (`shared/rewards.lua`). Util para integrar coins, VIP tags, llaves de casa, jobs, etc.
- 💰 **Premium pass flexible**:
    - Toggle global desde `Config`.
    - Toggle por season (`premium_purchasable`).
    - Hook `Config.Premium.CanPurchase` para integrar con cualquier sistema de monedas/credits externo.
    - Admin grant via panel o consola.
- 📈 **XP via export** — otros recursos otorgan XP llamando a `exports['nb-battlepass']:AddXP(src, amount, reason)`.
- 🌎 i18n EN/ES con `Locale()`.
- 🖥️ NUI moderno (Vue 3).

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy (1.9+) / QBCore *(standalone via nb-bridge)* |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge |
| **Inventario** | El que detecte nb-bridge (ox_inventory, qb-inventory, esx_inventory, etc.) |

---

## Filosofia: configurable vs. extensible

Existen **dos niveles** de personalizacion:

1. **In-game (sin codigo)** — Para staff: usar `/battlepassadmin` y editar seasons/niveles/recompensas en vivo.
2. **Codigo (sin tocar la logica de negocio)** — Para devs: editar `shared/rewards.lua` y `shared/config.lua` para añadir tipos de recompensa nuevos o integrar el premium con tu sistema de monedas.

La logica de XP, niveles, validaciones y claim flow esta **encriptada** (escrow). No necesitas tocarla.

---

## Secciones

- **[Instalacion](instalacion.md)** — Requisitos, BD y arranque.
- **[Configuracion](configuracion.md)** — Todas las opciones de `shared/config.lua`.
- **[Recompensas](recompensas.md)** — Tipos de recompensa builtin (item, money, vehicle, custom).
- **[Personalizacion](personalizacion.md)** — Como añadir recompensas custom sin tocar la logica.
- **[Exports](exports.md)** — API server-side: `AddXP`, `GrantPremium`, `GetPlayerData`.
- **[Integraciones](integraciones.md)** — Ejemplos: nb-vip, nb-coins, misiones diarias, kills, tiempo jugado, etc.
