# Comandos

Nombres personalizables en `Config.Commands.*`.

---

## Admin

| Comando | Args | Permiso | Accion |
|---------|------|---------|--------|
| `/nb_createdealer` | — | `Config.AdminGroups` | Abre un modal para crear un concesionario (nombre + job). |
| `/nb_deletedealer <jobName>` | nombre del job | `Config.AdminGroups` | Elimina el concesionario asociado al job (alt al panel). |
| `/nb_admin` | — | `Config.AdminGroups` | Abre el panel admin (listar / eliminar concesionarios). |

---

## Boss del concesionario

| Comando | Args | Permiso | Accion |
|---------|------|---------|--------|
| `/nb_bossmenu` | — | boss del job, o admin | Abre el panel de gestion del concesionario (peticiones, stock, dinero, ajustes). |

---

## Jugador

**No hay comandos exclusivos para jugador normal.** La interaccion es via markers:

- **Sell point** (posicion definida por el boss) — pulsa **E** en un coche para abrir el formulario de venta.
- **Showroom vehicle** — pulsa **E** cerca de un vehiculo colocado para comprar.

---

## Eventos para integracion

Si quieres lanzar flows desde otro recurso, estos eventos estan disponibles:

### Server -> Server

No hay exports. La integracion es via eventos (documentados en [Flujo de venta](flujo-venta.md) → seccion eventos).

### Server -> Client

| Evento | Payload | Descripcion |
|--------|---------|-------------|
| `nb_dealership:client:updateDealerships` | `dealerships[]` | Refresca cache de concesionarios en el cliente. |
| `nb_dealership:client:syncShowroom` | `showroomVehicles[]` | Re-sincroniza vehiculos en showroom. |
| `nb_dealership:client:refreshUI` | — | Refresca el menu abierto. |
| `nb_dealership:client:forceOpenMenu` | — | Fuerza abrir el boss menu (util tras crear concesionario). |
| `nb_dealership:client:refreshAdmin` | — | Refresca el panel admin. |
| `nb_dealership:client:openCreateDealerMenu` | — | Abre el modal de creacion. |

### Client -> Server

Todos validan permisos + rate limiting:

| Evento | Payload |
|--------|---------|
| `nb_dealership:server:createDealer` | `{ name, job }` |
| `nb_dealership:server:deleteDealership` | `dealershipId` |
| `nb_dealership:server:updateSellCoords` | `dealershipId, coords` |
| `nb_dealership:server:updateManageCoords` | `dealershipId, coords` |
| `nb_dealership:server:dealerAction` | `{ requestId, action: 'accept'\|'reject'\|'counter', price? }` |
| `nb_dealership:server:playerAcceptCounter` | `requestId, accepted (bool)` |
| `nb_dealership:server:addToShowroom` | `stockId, slot, price, coords` |
| `nb_dealership:server:removeFromShowroom` | `showroomId` |
| `nb_dealership:server:buyShowroomVehicle` | `showroomId` |
| `nb_dealership:server:societyDeposit` | `amount` |
| `nb_dealership:server:societyWithdraw` | `amount` |

---

## Callbacks (Framework.TriggerCallback)

| Callback | Payload | Descripcion |
|----------|---------|-------------|
| `nb_dealership:server:createSellRequest` | `data` | Crea la peticion de venta tras validacion. |
| `nb_dealership:server:getMyRequests` | — | Lista las peticiones del jugador (pendientes + counter). |
| `nb_dealership:server:getDealerRequests` | `dealershipId` | Lista peticiones pendientes del concesionario (boss). |
| `nb_dealership:server:getStock` | `dealershipId` | Lista stock del concesionario. |
| `nb_dealership:server:getDealershipDetails` | `dealershipId` | Detalles para el panel admin. |
| `nb_dealership:server:checkAdmin` | — | Verifica si el jugador es admin (para UI). |

> El namespace interno es `nb_dealership` aunque la carpeta se llame `nb-cardealer`. Es legacy — al integrar desde otros recursos, usa los strings tal cual.
