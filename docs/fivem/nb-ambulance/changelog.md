# Changelog

Formato basado en [Keep a Changelog](https://keepachangelog.com/) — versiones siguen [SemVer](https://semver.org/lang/es/).

---

## [0.1.0] — 2026-05-03

Primera version publica. Las 6 fases del roadmap inicial entregadas.

### Phase 1 — Death detection + PVP-safe

- Detector de muerte hibrido: `gameEventTriggered` + polling `IsEntityDead` como fallback para muertes no causadas por daño (caidas, ahogamiento).
- State machine `idle / laststand / dead / respawning` con transiciones claras.
- **PVP-safe flag** con tres mecanismos: export client/server, statebag directo, y zonas auto-detectadas configurables en `Config.Pvp.Zones`.
- Evento `nb-ambulance:client:deathIntercepted` para que scripts PVP corran su propio respawn.
- Fallback respawn en `Config.FallbackRespawn` cuando no hay hospitales registrados.

### Phase 2 — Creator in-game

- NUI Vue 3 con dos paneles (lista + editor) y 4 pestañas (Info / Camas / NPCs / Jobs).
- Captura de coordenadas in-game pulsando `[E]` (sin necesidad de coordsToVector externo).
- Persistencia en SQL con 4 tablas y FK CASCADE.
- Live reload sin reiniciar el recurso — cualquier cambio rebuiltea blips y NPCs en todos los clientes.
- Seed SQL de ejemplo para Pillbox Hill Medical.
- Comando admin `/creator` (alias de `/nb-ambulance creator`).

### Phase 3 — Last Stand + distress + revive admin

- Estado intermedio `laststand` antes de la muerte completa, con writhe loop animation y timer de bleed-out (default 5min).
- `[G]` durante laststand → llamada 911 con GPS + blip rojo a todos los medicos en servicio.
- `[E]` durante laststand (despues del `MinElapsedToGiveUp`) → progresa a death state.
- Comando `/revive [id]` para admins / medicos. Self-revive solo admin.
- Evento `nb-ambulance:client:revived` para integraciones.

### Phase 4 — Sistema de heridas

- Tracking per-bone × 4 tipos de daño (fracture, bleeding, burn, suffocating).
- 12 huesos curados por defecto, configurable.
- `Config.Injuries.WeaponDamageTypes` mapea armas vanilla → tipo de daño.
- Movement penalties: cojera + walk speed multiplier al tener daño en piernas/pies.
- Bleed tick pasivo que drena hp cada 8s mientras hay sangrado activo (skipped en laststand).
- 6 items de tratamiento configurables (`splint`, `bandage`, `tourniquet`, `burn_dressing`, `oxygen_mask`, `medikit`).
- NUI de tratamiento para paramedicos: `/treat [id]`.
- Hooks de items para ox_inventory (`use_<name>` exports) y para cualquier inventario via evento `:server:useHealItem`.

### Phase 5 — Reanimacion (CPR / defib)

- Prompt automatico al acercarse a un jugador en laststand (radio 2.5m configurable).
- **CPR**: sin item, 45% chance, 8s de animacion.
- **Defibrillator**: consume item, 90% chance, 5s. Auto-seleccionado si el medico tiene el item.
- Success roll server-side (no se puede tampear desde cliente).
- Recompensa al medico (`Config.Revive.Reward`) al banco.
- `RestoreHealth` post-revive configurable.
- Toast al paciente cuando un medico empieza un intento.

### Phase 6 — Dispatch state machine + NUI

- Calls con id, status (`pending` / `acked` / `completed`), assignedTo y timestamps.
- Auto-expiry tras `Config.Dispatch.ExpireMs` (15 min default).
- Cleanup automatico al desconectar caller (borra calls) o medico (devuelve a pending).
- Panel NUI con cards por llamada, status badges, GPS / Tomar / Cerrar buttons.
- Highlight visual de las calls asignadas a uno mismo.
- Live updates push (no polling) cada vez que cambia el estado.
- Caller recibe toasts: `medico en camino` (ack) y `caso cerrado` (complete).
- Comando `/dispatch` (alias `/nb-ambulance dispatch`) + keybind `F6` por defecto.

### Comandos directos

- Cada accion tiene un alias directo configurable en `Config.Commands`:
  `/911`, `/dispatch`, `/treat`, `/revive`, `/creator`, `/hospitals`, `/nbamb-reload`.
- Keybindings con `RegisterKeyMapping` (default F6 para dispatch).
- El umbrella `/nb-ambulance <sub>` queda como fallback.

### Compatibilidad

- ESX Legacy + QBCore via `nb-bridge` (`@nb-bridge/loader.lua`).
- ox_inventory + cualquier inventario generico.
- oxmysql (auto-detecta TINYINT(1) → boolean conversion).

### Conocidos / pendientes

- Garage de vehiculos por hospital — placeholder, no implementado todavia.
- Stash medico por hospital — placeholder.
- NPC roles `receptionist` / `duty` — solo se spawnean visualmente, sin interaccion (Phase 7+).
- `/911` sin cooldown integrado — implementacion via wrapper externo si se necesita.
