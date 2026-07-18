# nb-warehouse

Almacenes de **facción/negocio** (control de acceso por job/gang), almacenes
**ilegales de mercado negro** (probabilidad configurable de alarma/aviso a
policía) y almacenes **de jugador** (compra, renta, mejora de nivel, robo
server-validado, contraseña de acceso). Los admins crean, editan y eliminan
almacenes 100% in-game vía NUI — sin tocar Lua, sin reiniciar el recurso.

---

## Características

- 🏭 **Creator in-game** — registra almacenes de cualquier tipo desde una NUI Vue 3, con captura de coordenadas in-game (camina al sitio + pulsa `[E]`). Persiste en SQL, live sync a todos los clientes sin reiniciar el recurso.
- 🔐 **Control de acceso server-authoritative** — job/gang con grado mínimo, dueño individual, whitelist adicional por jugador, o acceso público con precio. Nunca se confía en el cliente.
- 🛒 **Compra y renta** — almacenes públicos pueden marcarse en venta y/o en alquiler. La renta expira automáticamente y revierte el almacén si no se renueva a tiempo.
- ⬆️ **Mejora de nivel** — el dueño primario invierte dinero para subir el nivel: más capacidad/peso de stash y menor probabilidad de robo.
- 🥷 **Robo server-validado** — otros jugadores pueden intentar robar un almacén marcado como robable mediante un skillcheck de `ox_lib`. El resultado se valida siempre en el servidor (ventana de tiempo + roll ponderado) — un cliente modificado no puede falsear un robo exitoso.
- 🔑 **Contraseña de acceso** — el dueño puede poner una contraseña para que otros jugadores entren sin estar en la whitelist de job/gang. Solo el dueño puede cambiarla o eliminarla.
- 🚨 **Alarma / aviso a policía** — almacenes `illegal` pueden disparar un hook de dispatch genérico (`nb-warehouse:policeAlert`) al ser abiertos, independiente del sistema de robo.
- 🗄️ **Base de datos autoinstalable** — las tablas y columnas se crean/actualizan solas al arrancar el recurso, sin necesidad de importar un `.sql` a mano.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy / QBCore / QBX (vía `nb-bridge`) |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge v2.3.0+ |
| **ox_lib** | requerido (menús contextuales, diálogos y skillcheck del robo) |
| **Inventario** | cualquiera soportado por nb-bridge — capacidad por-almacén (incluido el bonus de nivel) solo en `ox_inventory` |

---

## Secciones

- **[Instalación](instalacion.md)** — dependencias, base de datos autoinstalable, `ensure` order.
- **[Configuración](configuracion.md)** — todos los bloques de `Config`.
- **[Comandos](comandos.md)** — admin, gestión del dueño, keybindings.
- **[Exports](exports.md)** — API de solo lectura para integraciones.
- **[Changelog](changelog.md)** — historial de versiones.
