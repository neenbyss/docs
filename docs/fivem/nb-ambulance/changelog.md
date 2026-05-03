# Changelog

Formato basado en [Keep a Changelog](https://keepachangelog.com/) — versiones siguen [SemVer](https://semver.org/lang/es/).

---

## [1.0.0] — 2026-05-03 — Lanzamiento publico

Primera version estable de **nb-ambulance**. Sistema medico completo, multi-hospital, configurable in-game, con todas las features que esperarias del clasico ambulancejob — y varias mas.

### ✨ Sistema medico completo

- 🏥 **Multi-hospital con creator in-game** — registra cualquier numero de hospitales sin tocar Lua. NUI Vue 3 con captura de coordenadas in-game (camina al sitio + pulsa E). Persiste en SQL. Live reload — los cambios aparecen al instante en todos los clientes sin reiniciar el recurso.
- ☠️ **Last Stand revivible** — al morir entras en un estado intermedio con timer de bleed-out. Pulsa `[G]` para llamar al 911, `[E]` para rendirte. Mientras estas downed, un paramedico puede reanimarte.
- 🦴 **Sistema de heridas por hueso** — 12 huesos con 4 tipos de daño independientes (fractura, sangrado, quemadura, asfixia). Cada arma causa el tipo correcto de daño. Cojeras si tienes daño en piernas, sangraras pasivamente si tienes hemorragia.
- 💉 **Tratamiento medico** — paramedicos abren una NUI con la lista de heridas del paciente y aplican items especificos (splint, bandage, tourniquet, burn dressing, mascara de oxigeno, medikit).
- 🫀 **Reanimacion CPR + Desfibrilador** — RCP sin item con 45% de exito, o defibrilador (consume item) con 90%. Recompensa al medico que reanima.
- 📞 **Dispatch en vivo** — `/911` o `[G]` crean llamadas con id, status (pendiente / en camino / cerrada). Panel del medico con take/close, GPS, broadcasts en tiempo real entre todos los medicos conectados.

### 🏥 Amenities del hospital

- 🗄️ **Stash medico** — caja de almacenamiento por hospital. Compartido (todo el job ve el mismo inventario) o personal (cada jugador tiene su propio espacio). Configurable in-game con slots y peso. Integracion automatica con ox_inventory.
- 🚑 **Garage de vehiculos** — NPC que entrega vehiculos a paramedicos. Whitelist por grado (`min_grade`) — solo doctores ven el helicoptero, paramedicos solo la ambulancia. Captura del NPC y del spawn point por separado.
- 💊 **Pharmacy** — tienda de items medicos por hospital. Configura precio + grado minimo por item. Integracion con shops de ox_inventory; fallback de buy event para inventarios genericos.
- 🎒 **Bolsa medica portatil** — item que abre un stash personal del medico (ligado a su personaje). Util para llevar consumibles sin volver al hospital.

### 🛡️ Compatibilidad PVP

- **Flag PVP-safe** — los scripts de PVP / arena / gulag pueden suspender el flujo de muerte de nb-ambulance con una linea (`exports['nb-ambulance']:setDeathDisabled(true)`). Cuando alguien muere durante PVP, no entra en laststand ni ve la pantalla de muerte — el script PVP recibe un evento y corre su propio respawn.
- **Zonas auto-detectadas** — define esferas en el mapa (Sandy Shores arena, Vinewood DM, etc.) y el flag se activa/desactiva al entrar y salir.

### ⌨️ Comandos comodos

Sin tener que escribir `/nb-ambulance <subcomando>` cada vez:

- `/911` — civiles / testigos llaman al 911 desde cualquier sitio
- `/dispatch` — medicos abren el panel de llamadas activas (o pulsa **F6**)
- `/treat` — abre el NUI de tratamiento del jugador mas cercano
- `/revive` — reanima al objetivo
- `/creator` — abre el editor de hospitales (admin)

Todos renombrables en `Config.Commands`. El umbrella `/nb-ambulance` queda como fallback.

### 🌐 Compatibilidad

- **ESX Legacy** + **QBCore** automaticamente via `nb-bridge`.
- **ox_inventory** auto-detectado para stashes, shops y bolsa medica. Items registrables con un solo `server.export` por item.
- **Otros inventarios** soportados via eventos genericos (`nb-ambulance:server:useHealItem` y `nb-ambulance:server:useMedicalBag`).
- **oxmysql** para persistencia.
- **Multi-idioma** EN + ES (estructura preparada para mas).

### 📚 Documentacion

- Pagina completa en docs.neenbyss.com con guias, configuracion, exports y diagramas.
- API con 20+ exports cubriendo death state, heridas, hospitales, dispatch y amenities.
- Statebags `nb_state`, `nb_isDead`, `nb_inLaststand`, `nb_disableDeath`, `nb_injuries` para integraciones cross-script.

### 🧰 Para desarrolladores

- Arquitectura modular — cada feature en su propio archivo cliente/server.
- 7 fases de roadmap entregadas: death detection / creator / laststand / heridas / revive / dispatch / amenities.
- 9 tablas SQL con FK CASCADE.
- Listo para Tebex (escrow_ignore configurado: shared/config + locales + database bridge + UI = abiertos para customizacion del cliente).

---

## [0.x] — En desarrollo

Versiones internas previas al lanzamiento publico, sin uso productivo. Toda la historia de desarrollo se consolida en la 1.0.0.
