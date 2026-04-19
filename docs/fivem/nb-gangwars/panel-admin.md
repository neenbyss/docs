# Panel admin

Abre el panel con `/gangzones` (nombre configurable en `Config.AdminCommand`). Requiere que `Bridge.IsAdmin(src)` sea `true`.

El panel esta construido en Vue 3 (CDN, sin build step) y habla con el servidor via `Bridge.CreateCallback` de nb-bridge.

---

## Funcionalidades

### Listado de zonas

- Muestra cada zona con: nombre, tipo (cuadrada / circular), coordenadas, dimensiones, recompensa y estado en tiempo real (**Idle** / **Capturing** / **Cooldown** / **Captured**).
- Badges coloreados por estado, refresh automatico.

### Crear / editar zona

Campos del editor:

| Campo | Descripcion |
|-------|-------------|
| **Nombre** | Nombre visible en blips y notificaciones. |
| **Tipo** | `square` o `circle`. |
| **Coordenadas X/Y/Z** | Posicion del centro de la zona. Boton **"Usar mi posicion"** para auto-rellenar. |
| **Ancho / Alto** (square) | Tamano en metros del area. |
| **Radio** (circle) | Radio en metros. |
| **Rotacion** (square) | Grados en el eje Z. |
| **Recompensa** | Cantidad depositada en la gang ganadora al capturar (entero). |

Preview de blip en el mapa mientras editas.

### Acciones rapidas por zona

- **Activar / desactivar** — controla que zonas entran en rotacion.
- **Teletransportar** — te lleva al centro de la zona para ajustarla en el sitio.
- **Eliminar** — con confirmacion.

### Forzar rotacion

El boton **"Reiniciar rotacion"** limpia el pool actual y sortea `Config.ActiveZones` zonas aleatorias entre las activas. Util en eventos especiales.

---

## Callbacks usados (para referencia)

Todos registrados con `Bridge.CreateCallback` en el servidor:

- `nb_gangwars:getZones` — lista de zonas para clientes (al cargar).
- `nb_gangwars:admin:getZones` — lista completa (DB) para el panel.
- `nb_gangwars:admin:getActiveZones` — zonas en rotacion con su estado.
- `nb_gangwars:admin:createZone` / `updateZone` / `deleteZone`.
- `nb_gangwars:admin:activateZone` / `deactivateZone`.
- `nb_gangwars:admin:forceActivate` — reinicia rotacion.

Todas validan permiso con `Bridge.IsAdmin(src)`.

---

## Eventos de juego (no expuestos al admin, pero utiles para debugging)

| Evento | Direccion | Cuando |
|--------|-----------|--------|
| `nb_gangwars:playerEnteredZone` | Client -> Server | Gang entra en zona |
| `nb_gangwars:playerLeftZone` | Client -> Server | Gang sale de zona |
| `nb_gangwars:syncZones` | Server -> Client | Refresca zonas activas |
| `nb_gangwars:startProgress` | Server -> Client | Muestra la barra de fase / cooldown |
| `nb_gangwars:cancelProgress` | Server -> Client | Oculta la barra |
| `nb_gangwars:zoneCaptured` | Server -> Client | Overlay de victoria |

Si `Config.Debug = true`, los eventos se imprimen en consola.
