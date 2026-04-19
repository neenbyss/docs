# Comandos

Nombres reconfigurables en `Config.Commands.*` y `Config.Garage.CommandCreator`.

---

## Admin

| Comando | Args | Descripcion |
|---------|------|-------------|
| `/garagecreator` | — | Abre el panel Vue 3 de CRUD de garajes + flotas. |
| `/givecar [playerId] [model]` | id jugador, model spawn | Da un vehiculo al jugador + registra las llaves. |
| `/managevehicles [playerId]` | id jugador | Abre el gestor de los coches del jugador: spawn, delete, transferir a otro jugador. |

Todos requieren que `Bridge.IsAdmin(src)` sea `true`.

---

## Jugador

| Comando | Args | Descripcion |
|---------|------|-------------|
| `/myvehicles` | — | Lista tus coches agrupados por garaje, con flags (in-garage / out / stale / impounded). Permite recuperar vehiculos marcados como stale pagando la fee. |
| `/lockpick` | — | Intenta abrir el vehiculo mas cercano cerrado. Requiere el item configurado en `Config.Keys.Lockpick.Items.*`. |
| `/carjack` | — | Carjack al driver del vehiculo mas cercano con un arma apuntando. Probabilidad calculada por `WeaponChance * DriverMul`. |

---

## Keybinds

| Keybind | Accion | Opcion |
|---------|--------|--------|
| **L** | Lock / unlock del coche mas cercano propio | `Config.Keys.LockKeybind` |
| **G** | Toggle engine | `Config.Keys.EngineKeybind` |
| **H** | Hotwire | `Config.Keys.Hotwire.Keybind` |
| *(vacio)* | Dar llaves a otro jugador | `Config.Keys.GiveKeysKeybind` |

Todas se registran via `RegisterKeyMapping`, asi que el jugador puede rebindearlas en `Settings -> Keybinds -> FiveM`.

---

## Interaccion con el mundo

Cuando un marker de garaje esta visible:

- **E** para abrir el garaje (o para guardar si estas con un vehiculo cerca de la zona de store).
- **ESC** cierra la UI.

Si `Config.Garage.UseTarget = true`, el marker se desactiva y el sistema pasa a usar `qb-target` / `qtarget` (segun este instalado). Las acciones se muestran como opciones contextuales al apuntar.

---

## Callbacks disponibles (para referencia)

| Callback | Side | Descripcion |
|----------|------|-------------|
| `nb-garages:getVehicles` | Client -> Server | Lista de vehiculos del jugador para un garaje. |
| `nb-garages:takeOut` | Client -> Server | Sacar un vehiculo del garaje. Devuelve `(ok, vehicleData)`. |
| `nb-garages:keys:hasKeys` | Client -> Server | Comprobar llaves por plate. |
| `nb-garages:keys:getAll` | Client -> Server | Todas las placas a las que el jugador tiene llave. |
| `nb-garages:admin:*` | Client -> Server | 11 callbacks para el panel admin (crear, actualizar, borrar, flota, givecar, managevehicles...). Todos pasan por `Bridge.IsAdmin`. |
