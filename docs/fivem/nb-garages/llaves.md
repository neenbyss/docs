# Sistema de llaves

nb-garages incluye un sistema de llaves que **reemplaza `qb-vehiclekeys`** (se declara `provide 'qb-vehiclekeys'` en el manifest, asi los recursos que lo escuchen como dependencia seguiran funcionando).

> Si tenias `qb-vehiclekeys` instalado, **quitalo del server.cfg**. nb-garages cubre la API.

---

## Que hace

- **Lock / unlock** de vehiculos con la tecla (por defecto **L**).
- **Engine toggle** con tecla (por defecto **G**). Si `EngineRequiresKeys = true`, no arranca sin llaves.
- **Hotwire** — arrancar un coche robado con tiempo + cooldown + chance configurables.
- **Lockpick** — entrar a un coche cerrado con un item (`lockpick` o `advancedlockpick`). Con tiers por clase de vehiculo.
- **Carjack** — robar un coche a un NPC o jugador con una arma apuntando. Multiplicador por tipo de arma y tipo de driver.
- **Job-shared keys** — policias/ambulancias on-duty pueden conducir cualquier coche de flota de su trabajo.
- **Give keys** — compartir llaves con otro jugador (evento + keybind opcional).
- **Steal on death** — si matas a un NPC o jugador, puedes registrar sus llaves como tuyas.

---

## Config

Todas las opciones bajo `Config.Keys.*`.

### Basicos

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Enabled` | Activar el sistema | `true` |
| `LockKeybind` | Tecla para lock/unlock | `'L'` |
| `EngineKeybind` | Tecla para toggle engine | `'G'` |
| `GiveKeysKeybind` | Tecla opcional para dar llaves (texto vacio = desactivado) | `''` |
| `LockDistance` | Metros para poder cerrar el coche | `5.0` |
| `EngineRequiresKeys` | Bloquea el arranque sin llaves | `true` |
| `LockNPCDriving` | Lockear NPCs conduciendo | `true` |
| `LockNPCParked` | Lockear NPCs aparcados | `true` |
| `StealKeysFromDead` | Tomar llaves de un driver muerto | `true` |

### Hotwire

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Hotwire.Enabled` | Sistema on/off | `true` |
| `Hotwire.Keybind` | Tecla | `'H'` |
| `Hotwire.Chance` | Probabilidad de exito (0-1) | `0.5` |
| `Hotwire.Time` | ms del minigame | `10000` |
| `Hotwire.Cooldown` | ms entre intentos | `15000` |
| `Hotwire.AlertPolice` | Alerta dispatch al intentar | `true` |

### Lockpick

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Lockpick.Command` | Comando (ademas del key item) | `'lockpick'` |
| `Lockpick.Items.Normal` | Nombre del item basico | `'lockpick'` |
| `Lockpick.Items.Advanced` | Nombre del item avanzado | `'advancedlockpick'` |
| `Lockpick.BreakChance.Normal` | Chance de romper el item basico | `0.5` |
| `Lockpick.BreakChance.Advanced` | Chance de romper el avanzado | `0.1` |
| `Lockpick.BaseChance` | Chance base de exito | `0.5` |
| `Lockpick.ClassMul` | `[class] = { chanceMul, alarmChance, advancedOnly, difficulty }` | ver `config.lua` |

`ClassMul` te permite hacer que las superdeportivos exijan `advancedlockpick` (`advancedOnly = true`) y disparen alarma.

### Carjack

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Carjack.Enabled` | Sistema on/off | `true` |
| `Carjack.Command` | Comando / keybind texto | `''` -> `/carjack` |
| `Carjack.Time` | ms hasta completar | `7500` |
| `Carjack.Cooldown` | ms entre intentos | `10000` |
| `Carjack.WeaponChance` | `{ MELEE = 0.0, PISTOL = 0.3, SHOTGUN = 0.8, ... }` | ver config |
| `Carjack.DriverMul` | `{ random = 1.0, gang = 0.8, cop = 0.0 }` | ver config |

Ejemplo: con pistola a un conductor random, chance final = `0.3 * 1.0 = 0.3`. A un policia: `0.3 * 0.0 = 0` (imposible, las escolta ley no se carjackean).

### Job-shared keys

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `JobShared.Enabled` | Compartir llaves por trabajo | `false` |
| `JobShared.RequireOnDuty` | Exigir on-duty | `true` |
| `JobShared.Jobs` | `{ police = { 'police', 'police2' } }` | `{}` |

La clave de `Jobs` es **el trabajo del jugador**; el array son los **nombres de trabajo** marcados en los coches (`nb_job`). Asi un `police2` puede conducir un coche marcado como `police` sin pedir llaves.

---

## Eventos internos

### Server

| Evento | Descripcion |
|--------|-------------|
| `nb-garages:keys:server:give` | Registrar llaves a un source. |
| `nb-garages:keys:server:toggleLock` | Server authoritative lock toggle. |
| `nb-garages:keys:server:giveToPlayer` | Compartir llaves con otro jugador. |
| `nb-garages:keys:server:hotwire` | Cliente reporta hotwire exitoso (server trust con validacion). |
| `nb-garages:keys:server:lockpickBroke` | Destruye el item de lockpick. |
| `nb-garages:keys:server:giveOnTakeOut` | Auto-give al sacar del garaje. |

### Compat con qb-vehiclekeys

nb-garages escucha los eventos **de qb-vehiclekeys** para no romper otros recursos que disparen directo:

- `qb-vehiclekeys:server:AcquireVehicleKeys`
- `qb-vehiclekeys:server:GiveVehicleKeys`
- `qb-vehiclekeys:server:setVehLockState`
- `qb-vehiclekeys:client:AddKeys`
- `qb-vehiclekeys:client:RemoveKeys`

---

## Exports

### Server

```lua
exports['nb-garages']:GiveKeys(playerId, plate)   -- boolean
exports['nb-garages']:RemoveKeys(playerId, plate) -- boolean
exports['nb-garages']:HasKeys(playerId, plate)    -- boolean

-- Alias compatible:
exports['qb-vehiclekeys']:HasKeys(playerId, plate)
```

### Client

```lua
exports['nb-garages']:HasKeys(plate)              -- boolean
exports['qb-vehiclekeys']:HasKeys(plate)          -- alias
```

---

## Policia y dispatch

Si `Hotwire.AlertPolice = true`, al disparar un hotwire se llama a la funcion `Bridge.Keys.AlertPolice(source, vehicle)` que esta en `bridge/keys.lua` (archivo **abierto** para que customices a tu dispatch preferido: `cd_dispatch`, `ps-dispatch`, etc.).

```lua
-- bridge/keys.lua
function Bridge.Keys.AlertPolice(source, vehicle)
    TriggerEvent('cd_dispatch:AddNotification', {
        job_table = { 'police' },
        coords    = GetEntityCoords(vehicle),
        title     = 'Hotwire',
        message   = 'Intento de arranque sin llaves detectado.',
        flash     = false,
    })
end
```
