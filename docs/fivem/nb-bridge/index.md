# nb-bridge

Capa centralizada de abstraccion de framework para todos los recursos de Neenbyss. Una sola dependencia unifica el acceso a ESX, QBCore y QBX (qbx_core), sistemas de inventario, notificaciones, vehiculos, licencias, callbacks, barras de progreso, auditoria y hooks de ciclo de vida.

> **Descarga:** [github.com/neenbyss/nb-bridge](https://github.com/neenbyss/nb-bridge) — lee la seccion de [Instalacion](instalacion.md) para los pasos. Usa siempre la release mas reciente (**v2.2.0** al momento de escribir esto).

---

## Por que usar nb-bridge

Antes de nb-bridge cada recurso `nb-*` copiaba su propia carpeta `bridge/` con el mismo codigo duplicado para detectar el framework, enviar notificaciones, manejar inventario, etc. Eso causaba:

- Inconsistencias cuando uno se actualizaba y otros no.
- Mantenimiento doloroso: arreglar un bug significaba tocar 7+ scripts.
- Espacio perdido: los mismos 500+ lines copiados en cada recurso.

Con nb-bridge todos los recursos `nb-*` dependen de un unico recurso. Una sola actualizacion beneficia a todos los scripts.

---

## Caracteristicas

- 🧠 **Auto-deteccion** — framework (ESX / QBCore / QBX), inventario (ox / qb / qs / origen / default), notificaciones (ox_lib / ESX / QBCore / QBX / native), licencias (bcs / okok / esx_license / metadata).
- 🔁 **API namespaced** — un unico `local bridge = exports['nb-bridge']:get()` expone `bridge.player`, `bridge.inventory`, `bridge.vehicle`, `bridge.notify`, `bridge.callback`, `bridge.license`, `bridge.progress`, `bridge.event`, `bridge.log` y `bridge.ui`, con metodos camelCase.
- 🧱 **Modulos independientes** — player, inventory, vehicle, notify, callback, license, progress, event, log, ui, diagnostics.
- 🧩 **Overrides** — customiza cualquier funcion namespaced sin modificar los archivos base, solo añadiendo un `.lua` en `overrides/`.
- ⚖️ **Cascada de configuracion** — tu `Config` tiene prioridad; si no existe, cae a `BridgeConfig`.
- 📤 **Export unico** — `exports['nb-bridge']:get()` entrega la tabla `Bridge` completa lista para usar. Tambien existe `exports['nb-bridge']:diagnostics()` para inspeccionar el estado en runtime sin pasar por `get()`. Ya no hay un export por metodo.
- 🧾 **Auditoria** — `bridge.log.createLog()` escribe en qb-log, un webhook de Discord o el `Debugger` como fallback, sin depender de `Debug` — es un registro de produccion, no un print de desarrollo.
- 🧩 **Hooks de ciclo de vida** — `bridge.event.*` unifica `onPlayerLoaded`, `onPlayerUnloaded`, `onPlayerSpawned`, `onResourceStart/Stop` y `onSelfStart/Stop` en un solo namespace.
- 🧾 **Lua 5.4** nativo.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **FiveM** | Artifacts 5181+ recomendado |
| **Framework** | ESX Legacy, QBCore o QBX (qbx_core) |
| **Base de datos** | oxmysql (para vehicle + licencias) |
| **Inventario (opcional)** | ox_inventory, qb-inventory, qs-inventory, origen_inventory |
| **ox_lib (opcional)** | Mejora notificaciones y progress bars; QBX siempre lo trae de base |
| **Webhook de Discord (opcional)** | Habilita embeds de auditoria en `bridge.log`; vacio por defecto, se configura por servidor |

---

## Secciones

- **[Instalacion](instalacion.md)** — Descarga desde GitHub, orden en `server.cfg`, verificacion con el export `get()`.
- **[Configuracion](configuracion.md)** — `BridgeConfig`, cascada con el `Config` del consumidor, incluida la config de logs.
- **[Modulos](modulos.md)** — API completa de player, inventory, vehicle, notify, callback, license, progress, event, log, ui y diagnostics.
- **[Overrides](overrides.md)** — Como sustituir funciones namespaced sin tocar los archivos base.
- **[Changelog](changelog.md)** — Historial de versiones.

---

## Consumidores (recursos que dependen de nb-bridge)

- [nb-crafting](../nb-crafting/index.md)
- [nb-cars](../nb-cars/index.md)
- [nb-jobmanagers](../nb-jobmanagers/index.md)
- [nb-vip](../nb-vip/index.md)
- [nb-consumibles](../nb-consumibles/index.md)

Todos ellos requieren nb-bridge en `server.cfg` **antes** que el recurso consumidor.
