# nb-bridge

Capa centralizada de abstraccion de framework para todos los recursos de Neenbyss. Una sola dependencia unifica el acceso a ESX y QBCore, sistemas de inventario, notificaciones, vehiculos, licencias, callbacks y barras de progreso.

> **Descarga:** [github.com/neenbyss/nb-bridge](https://github.com/neenbyss/nb-bridge) — lee la seccion de [Instalacion](instalacion.md) para los pasos. Usa siempre la release mas reciente.

---

## Por que usar nb-bridge

Antes de nb-bridge cada recurso `nb-*` copiaba su propia carpeta `bridge/` con el mismo codigo duplicado para detectar el framework, enviar notificaciones, manejar inventario, etc. Eso causaba:

- Inconsistencias cuando uno se actualizaba y otros no.
- Mantenimiento doloroso: arreglar un bug significaba tocar 7+ scripts.
- Espacio perdido: los mismos 500+ lines copiados en cada recurso.

Con nb-bridge todos los recursos `nb-*` dependen de un unico recurso. Una sola actualizacion beneficia a todos los scripts.

---

## Caracteristicas

- 🧠 **Auto-deteccion** — framework (ESX / QBCore), inventario (ox / qb / qs / default), notificaciones (ox_lib / ESX / QB / native), licencias (bcs / okok / esx_license / metadata).
- 🔁 **API unificada** — misma llamada `Bridge.*` funcione en el framework que funcione.
- 🧱 **Modulos independientes** — framework, inventory, notify, vehicle, callbacks, licenses, progress.
- 🧩 **Overrides** — customiza cualquier funcion sin modificar los archivos base, solo añadiendo un `.lua` en `overrides/`.
- ⚖️ **Cascada de configuracion** — tu `Config` tiene prioridad; si no existe, cae a `BridgeConfig`.
- 📤 **Exports FiveM** — cada funcion esta disponible tambien como export para scripts que no son nb-*.
- 🧾 **Lua 5.4** nativo.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **FiveM** | Artifacts 5181+ recomendado |
| **Framework** | ESX Legacy (1.9+) o QBCore |
| **Base de datos** | oxmysql (para vehicle + licencias) |
| **Inventario (opcional)** | ox_inventory, qb-inventory, qs-inventory, origen_inventory |
| **ox_lib (opcional)** | Mejora notificaciones y progress bars |

---

## Secciones

- **[Instalacion](instalacion.md)** — Descarga desde GitHub, orden en `server.cfg`, verificacion.
- **[Configuracion](configuracion.md)** — `BridgeConfig`, cascada con el `Config` del consumidor.
- **[Modulos](modulos.md)** — API completa de framework, inventory, notify, vehicle, callbacks, licenses, progress.
- **[Overrides](overrides.md)** — Como sustituir funciones sin tocar los archivos base.
- **[Changelog](changelog.md)** — Historial de versiones.

---

## Consumidores (recursos que dependen de nb-bridge)

- [nb-crafting](../nb-crafting/index.md)
- [nb-cars](../nb-cars/index.md)
- [nb-jobmanagers](../nb-jobmanagers/index.md)
- [nb-vip](../nb-vip/index.md)
- [nb-consumibles](../nb-consumibles/index.md)

Todos ellos requieren nb-bridge en `server.cfg` **antes** que el recurso consumidor.
