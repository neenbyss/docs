# Changelog

Formato basado en [Keep a Changelog](https://keepachangelog.com/) — versiones siguen [SemVer](https://semver.org/lang/es/).

---

## [1.2.0] — 2026-07-18

### ✨ Exports para integraciones

- 🔌 **API de solo lectura** — `getWarehouse`, `getWarehouses`, `getWarehousesByOwner`, `hasAccess`, `isManager` y `getEffectiveCapacity` en el servidor; `getWarehouse`/`getWarehouses` en el cliente. Pensada para paneles de facción, HUDs, overlays de dispatch o estadísticas externas.
- 🔒 Ningún export expone la contraseña de un almacén ni la identidad completa del dueño. No hay exports de escritura a propósito — comprar, rentar, subir de nivel, robar o administrar contraseña siguen siendo flujos exclusivos del propio servidor.

---

## [1.1.0] — 2026-07-18

Primer release funcional de **nb-warehouse** — el recurso no arrancaba correctamente antes de esta versión.

### ✨ Almacenes de jugador

- 🛒 **Compra y renta** — almacenes públicos pueden marcarse en venta y/o en alquiler. La renta expira sola y revierte el almacén si no se renueva.
- ⬆️ **Mejora de nivel** — el dueño primario invierte dinero para subir el nivel: más capacidad/peso y menor probabilidad de robo.
- 🥷 **Robo server-validado** — skillcheck de `ox_lib` en el cliente, validado siempre en el servidor con una ventana de tiempo plausible más un roll de probabilidad ponderado. Un cliente modificado no puede falsear un robo exitoso.
- 🔑 **Contraseña de acceso** — el dueño pone/cambia/quita una contraseña para dar acceso a otros jugadores sin tocar la whitelist de job/gang. Solo el dueño puede modificarla.

### 🗄️ Base de datos autoinstalable

- Las tablas y columnas se crean/actualizan solas al arrancar el recurso — ya no hace falta importar el `.sql` a mano.

### 🐛 Arreglos

- El recurso no arrancaba: seis módulos del servidor existían en disco pero nunca se cargaban.
- El panel admin perdía silenciosamente los campos de renta/nivel/robo al guardar un almacén.
- Los almacenes comprados/rentados por un jugador (`owner_type='player'`) eran inaccesibles para su propio dueño.

### 🔒 Seguridad

- El panel admin ya no expone la contraseña real de ningún almacén, solo si tiene una configurada.
- El resultado de un robo ya no se confía ciegamente del cliente — se verifica siempre en el servidor.
- El dueño (o un jugador en whitelist) ya no puede resetear su propio cooldown de robo llamando el evento directamente.

---

## [1.0.0] — En desarrollo

Versión interna previa a la 1.1.0, con seis módulos del servidor escritos pero nunca conectados al `fxmanifest.lua` — el recurso no realizaba ninguna de sus funciones. Toda la historia de desarrollo se consolida en la 1.1.0.
