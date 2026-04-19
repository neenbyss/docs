# nb-cardealer

Sistema de **concesionarios multiples** con flujo completo: un jugador lleva su vehiculo al punto de venta → el staff del concesionario acepta / rechaza / contra-oferta → si se acuerda, el vehiculo pasa al stock del concesionario → un empleado lo coloca en showroom con un precio → un cliente lo compra.

> Version actual: **v1.6.0** — incluye locks anti-duplicacion, rate limiting, validaciones server-side, deposit/withdraw restringido a boss/admin.

---

## Caracteristicas

- 🏢 **Multi-concesionario** — sin limite, cada uno asociado a un trabajo (`cardealer`, `bennys`, ...).
- 🤝 **Negociacion** — el staff puede aceptar, rechazar o hacer contra-oferta. El jugador decide.
- 🗂️ **Stock independiente** — los vehiculos comprados viven en `nb_dealer_stock`, **no** en `owned_vehicles` del jugador.
- 🏎️ **Showroom persistente** — vehiculos colocados en el mundo con etiqueta 3D. Se mantienen tras reinicios.
- 💼 **Boss menu** — gestionar requests, stock, cuentas y puntos de ubicacion sin tocar config.
- 🛡️ **Panel admin** — crear y borrar concesionarios.
- 💰 **Cuentas de sociedad** — dinero compartido via `esx_addonaccount` (sociedad `society_<job>`). Deposito/retiro solo por boss o admin (v1.6).
- 🔔 **Logs en Discord** — webhook opcional con eventos: sell request, acceptance, contra-oferta, compra en showroom.
- 🔒 **Seguridad v1.6** — locks de concurrencia, cooldowns por jugador, validacion de distancia server-side, plate lock (validacion de propietario).

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy (v1.x+) |
| **Base de datos** | oxmysql |
| **Sociedades** | esx_society + esx_addonaccount |
| **QBCore** | Parcial — requiere editar `framework.lua` (las funciones helper estan centralizadas) |
| **Target (opcional)** | no (usa markers propios) |

> **QBCore**: el recurso esta centrado en ESX, pero `framework.lua` es **abierto** (no encriptado) y tiene wrappers bien delimitados. Adaptarlo a QBCore es viable sin reescribir el core.

---

## Secciones

- **[Instalacion](instalacion.md)** — Dependencias, SQL, orden en server.cfg.
- **[Configuracion](configuracion.md)** — Opciones, locales y logs.
- **[Flujo de venta](flujo-venta.md)** — Sell request -> stock -> showroom.
- **[Panel admin y boss](paneles.md)** — Crear concesionarios, gestionar requests, stock y dinero.
- **[Comandos](comandos.md)** — Todos los comandos y sus permisos.
