# nb-gangwars

Sistema de captura de territorios para bandas. Las gangs compiten por zonas activas mediante un esquema de **3 fases** con cooldown entre cada una: mantener la zona durante las fases consecutivas otorga la captura, un premio en efectivo a la gang ganadora y la zona sale de rotacion hasta el proximo ciclo.

---

## Caracteristicas

- 🟥 **Zonas activas rotativas** — N zonas simultaneas (por defecto 4), configurables por el admin desde el panel.
- 🕒 **Mecanica de 3 fases** — cada fase dura X segundos (default 40s) con cooldown entre fases (default 60s). Abandonar la zona durante una fase reinicia el progreso de la gang atacante.
- 💸 **Recompensa configurable** por zona, depositada en la cuenta de la gang via funcion custom (`AddGangMoney`).
- 🗺️ **Blips dinamicos** con parpadeo y color configurables; se ocultan al capturar la zona.
- 🛠️ **Panel Vue 3 admin** para CRUD de zonas, activar/desactivar, teleportar a zona, forzar reinicio de rotacion.
- 🌎 **Multi-idioma** — EN / ES / FR.
- 🧱 **Server-authoritative** — posicion validada en el servidor cada N segundos, evita spoofing de presencia.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy / QBCore (via nb-bridge) |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge |
| **Sistema de gangs** | Personalizable (plantillas para nb-jobscreator, qb-management y societies ESX) |

---

## Secciones

- **[Instalacion](instalacion.md)** — Requisitos, base de datos y comprobacion.
- **[Configuracion](configuracion.md)** — Opciones y tres funciones custom de integracion con tu sistema de gangs.
- **[Panel admin](panel-admin.md)** — Crear, editar, activar y teletransportarte a zonas.
- **[Integracion](integracion.md)** — Enganchar tu sistema de gangs y recompensas.
