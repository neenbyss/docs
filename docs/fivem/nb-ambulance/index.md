# nb-ambulance

Sistema medico **multi-hospital, multi-job, configurable in-game** para FiveM. Reemplaza a `esx_ambulancejob` / `qb-ambulancejob` con una arquitectura modular: detector de muerte con flag PVP-safe, last stand revivible, heridas por hueso, panel de dispatch en vivo y creator NUI para registrar hospitales sin tocar Lua.

---

## Caracteristicas

- 🏥 **Creator in-game** — registra hospitales, camas, NPCs y job whitelist desde una NUI Vue 3, todo persistente en SQL con sync en vivo a todos los clientes.
- ☠️ **Last Stand revivible** — al morir entra en un estado intermedio con writhe loop y timer de bleed-out. Pulsa `[G]` para llamar al 911, `[E]` para rendirse.
- 🛡️ **PVP-safe** — flag por jugador (statebag + export + zonas auto) para que scripts PVP / arena / gulag puedan saltarse el flujo de muerte sin tocar nb-ambulance.
- 🦴 **Heridas por hueso** — 12 huesos × 4 tipos de daño (fractura, sangrado, quemadura, asfixia) con efectos de movimiento (cojera) y bleed tick pasivo.
- 💉 **Tratamiento medico** — NUI para paramedicos: lista huesos dañados con botones de items (splint, bandage, tourniquet, burn_dressing, medikit). Hooks para ox_inventory y cualquier inventario.
- 🫀 **Reanimacion** — RCP (sin item, 45% chance) o desfibrilador (consume item, 90% chance). Recompensa al medico, success roll del lado del server.
- 📞 **Dispatch en vivo** — `/911` o `[G]` crean llamadas con estado (pendiente / en camino / cerrada). Panel del medico con take/close, GPS, broadcasts en tiempo real.
- 💀 **Multi-hospital respawn** — al morir, respawn en la cama mas cercana entre los hospitales activos. Tarifa por hospital configurable.
- ⌨️ **Comandos directos** — `/911`, `/dispatch`, `/treat`, `/revive`, `/creator`. Sin prefijo `nb-ambulance` obligatorio. Renombrables, F6 por defecto al panel de dispatch.
- 🏥 **Amenities del hospital** — stash medico (compartido o personal), garage de vehiculos con whitelist por grado, pharmacy con shop ox_inventory, bolsa medica portatil con stash propio.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy / QBCore |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge |
| **Inventario (opcional)** | ox_inventory (auto-detectado) — hooks `use_<item>` listos |
| **Target (opcional)** | ninguno requerido — interaccion via prompt nativo |

> **No es compatible con `esx_ambulancejob` / `qb-ambulancejob`** — captura el evento de muerte y entra en conflicto. Retira el script viejo antes de instalar.

---

## Secciones

- **[Instalacion](instalacion.md)** — SQL, dependencias, ensure order.
- **[Configuracion](configuracion.md)** — todos los bloques de `Config`.
- **[Comandos](comandos.md)** — slash commands, alias, keybindings.
- **[Creator in-game](creator.md)** — flujo del editor de hospitales.
- **[Sistema de heridas](heridas.md)** — tracker, tratamiento, items.
- **[Dispatch / 911](dispatch.md)** — llamadas, panel medico, lifecycle.
- **[Amenities](amenities.md)** — stash, garage, pharmacy, bolsa medica.
- **[PVP-safe death](pvp.md)** — desactivar la muerte para arenas / minijuegos.
- **[Exports](exports.md)** — API completa para integraciones.
- **[Changelog](changelog.md)** — version log.
