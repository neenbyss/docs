# nb-garages

Sistema de garajes **multi-tipo** con **creador in-game**, **persistencia de vehiculos** y **sistema de llaves** drop-in compatible con `qb-vehiclekeys`. Cubre garajes publicos, sociedad, trabajo, impound (deposito), casas y garajes custom (gang / VIP).

> Version actual: **v2.0.0** (incluye el sistema de llaves unificado).

---

## Caracteristicas

- 🏠 **6 tipos de garaje** — `public`, `society`, `job`, `depot`, `house`, `custom`.
- 🛠️ **Creador in-game** — panel Vue 3 para CRUD de garajes (coords, spawn, blip, marker, restricciones).
- 💾 **Persistencia de vehiculos** — los coches permanecen en el mundo tras reiniciar, con culling configurable.
- 🚓 **Impound / deposito** — deteccion de vehiculos abandonados, fees por clase, skip cuando el owner esta online.
- 🔑 **Keys system** — lock/unlock, hotwire, lockpick (con tiers por clase de vehiculo), carjack (multiplicador por arma + tipo de driver). Reemplaza `qb-vehiclekeys` declarandose `provide 'qb-vehiclekeys'`.
- 🏢 **Flota de sociedad** — garajes de trabajo con vehiculos compartidos y control por grado minimo.
- 🎯 **Motores de minigame intercambiables** — native, ox_lib, qb-minigames o ps-ui.
- 🌐 **Multi-framework** — ESX + QBCore via nb-bridge. Reutiliza `owned_vehicles` / `player_vehicles` sin crear tablas paralelas.
- 🛎️ **Compatibilidad con target / ox_lib** — markers o target framework a eleccion.
- 🌎 **i18n** — EN / ES.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy (1.9+) / QBCore |
| **Base de datos** | oxmysql (MariaDB 10.0+ recomendado) |
| **Bridge** | nb-bridge |
| **Minigames** (opcional) | ox_lib, qb-minigames, ps-ui o nativo |
| **Target** (opcional) | qb-target, qtarget |
| **House keys** (opcional) | qb-houses, o cualquier sistema con export |

---

## Secciones

- **[Instalacion](instalacion.md)** — Requisitos, SQL, orden en server.cfg.
- **[Configuracion](configuracion.md)** — Todas las opciones globales.
- **[Tipos de garaje](tipos-garage.md)** — Que hace cada tipo y cuando usarlo.
- **[Sistema de llaves](llaves.md)** — Lock/unlock, hotwire, lockpick, carjack.
- **[Persistencia e impound](persistencia.md)** — Como se guardan los coches y cuando se envian al deposito.
- **[Comandos](comandos.md)** — Comandos de admin y de jugador.
- **[Exports](exports.md)** — API server + client.
