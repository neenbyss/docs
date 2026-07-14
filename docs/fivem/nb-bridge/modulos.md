# Modulos

nb-bridge esta dividido en modulos independientes. Cada uno expone metodos bajo un namespace de la tabla `Bridge` (`bridge.player`, `bridge.inventory`, etc.) con nombres en **camelCase**.

---

## Acceso: el export `get()`

La unica forma de usar nb-bridge desde v2.0.0 es pedir la tabla `Bridge` con el export `get()`. Llamalo **una sola vez por archivo**, al inicio, y reutiliza la variable local:

```lua
-- fxmanifest.lua
dependencies { 'nb-bridge' }

-- server.lua / client.lua
local bridge = exports['nb-bridge']:get()
```

`bridge` es la tabla `Bridge` completa. Desde ahi accedes a cualquier namespace: `bridge.player.getJob(source)`, `bridge.notify.send(source, 'msg', 'success')`, etc.

Ademas de los namespaces, `bridge` expone tres propiedades globales (no namespaced, siguen en PascalCase):

| Propiedad | Tipo | Valor |
|-----------|------|-------|
| `bridge.Framework` | `string` | `'ESX'`, `'QBCore'` o `'QBX'` |
| `bridge.FrameworkObject` | `table` | Objeto crudo del framework detectado |
| `bridge.InventorySystem` | `string\|nil` | Sistema de inventario activo (se resuelve ~500ms despues del arranque) |

> El patron viejo `shared_scripts { '@nb-bridge/loader.lua' }`, que inyectaba un `Bridge` global directamente en tu recurso, esta **deprecado** desde v2.0.0. Usa siempre `get()`.

---

## Player

Gestion de jugadores, permisos, dinero, trabajo, gang y metadata. Auto-detecta ESX, QBCore y QBX.

### Server

```lua
-- Jugador
bridge.player.getPlayer(source)              -- xPlayer (ESX) o Player (QBCore/QBX)
bridge.player.getIdentifier(source)          -- license (ESX) o citizenid (QBCore/QBX)
bridge.player.getSSN(source)                 -- ESX only
bridge.player.getPlayerName(source)

-- Permisos
bridge.player.getGroup(source)               -- 'god' | 'superadmin' | 'admin' | 'mod' | 'user'
bridge.player.setGroup(source, group)        -- ESX only
bridge.player.isAdmin(source)                -- usa Config.AdminGroups

-- Dinero
bridge.player.addMoney(source, type, amount, reason)     -- type: 'cash' o 'bank'
bridge.player.removeMoney(source, type, amount, reason)
bridge.player.setMoney(source, type, amount, reason)
bridge.player.getMoney(source, type)
bridge.player.getAccounts(source)            -- { cash = N, bank = N, ... }

-- Trabajo y gang
bridge.player.getJob(source)                 -- { name, label, grade, grade_name, grade_label, grade_salary, onDuty }
bridge.player.setJob(source, job, grade, onDuty)
bridge.player.getGang(source)                -- QBCore/QBX only, nil en ESX
bridge.player.setGang(source, gangName, grade) -- QBCore/QBX only

-- Metadata
bridge.player.setMeta(source, key, value, subKey)
bridge.player.getMeta(source, key, subKey)
bridge.player.clearMeta(source, key, subKey)

-- Varios
bridge.player.getAllPlayers()                -- number[] de source IDs online
bridge.player.getPlayTime(source)            -- ESX only
bridge.player.setCoords(source, coords)
bridge.player.getCoords(source)
bridge.player.triggerClientEvent(source, event, ...)
bridge.player.playerVar(source, key, value)  -- ESX only
bridge.player.executeCommand(source, command) -- ESX only
bridge.player.createBill(src, targetId, amount, desc, jobName)
bridge.player.registerCommand(name, group, cb, suggestion)

-- Eventos
bridge.player.onPlayerLoaded(function(source, identifier) end)
bridge.player.onPlayerUnloaded(function(source) end)     -- QBX: debounced 1s
bridge.player.onMoneyChanged(function(source, moneyType, amount, newBalance, changeSource) end)
```

`bridge.player.onMoneyChanged` (v2.2.0) dispara con **cualquier** cambio de dinero, venga o no de `bridge.player.*` — escucha directamente los eventos nativos del framework. `changeSource` es una tabla estructurada (ej. `{ tag = 'bridge' }`) que indica el origen del cambio; usala para evitar reentradas si tu propio handler tambien mueve dinero, en vez de parsear el string de `reason`.

### Client

Los metodos de cliente **no** reciben `source` — operan sobre el jugador local.

```lua
bridge.player.getPlayerData()
bridge.player.getJob()                       -- mismo formato canonico que el server
bridge.player.getGang()                      -- QBCore/QBX only
bridge.player.getMoney(type)
bridge.player.getAccounts()
bridge.player.getIdentifier()
bridge.player.getPlayerName()
bridge.player.getGroup()                     -- QBCore/QBX siempre retorna 'user'; no usar para gates de seguridad

-- Eventos
bridge.player.onPlayerLoaded(function() end)
bridge.player.onPlayerUnloaded(function() end)
bridge.player.onJobUpdate(function(job) end)
bridge.player.onGangUpdate(function(gang) end) -- QBCore/QBX only
```

### Ejemplos

```lua
-- Server: bonus + notificacion
bridge.player.registerCommand('bonus', 'admin', function(source)
    local name = bridge.player.getPlayerName(source)
    bridge.player.addMoney(source, 'bank', 5000, 'bonus_evento')
    bridge.notify.send(source, name .. ' recibio $5,000', 'success')
end)

-- Client: reaccionar a cambio de trabajo
bridge.player.onJobUpdate(function(job)
    print('Nuevo trabajo: ' .. job.name)
end)
```

---

## Notify

Sistema unificado de notificaciones. Auto-detecta ox_lib, ESX, QBCore, QBX o native.

```lua
-- Server
bridge.notify.send(source, 'Factura pagada', 'success')
bridge.notify.send(source, 'Dinero insuficiente', 'error')

-- Client
bridge.notify.show('Item recibido', 'info')
```

**Tipos soportados:** `'success'`, `'error'`, `'info'`, `'warning'`.

---

## Inventory

Abstraccion multi-inventario. Soporta `ox_inventory`, `qb-inventory`, `qs-inventory`, `origen_inventory` y defaults del framework. En servidores QBX, `bridge.InventorySystem` siempre resuelve a `'ox_inventory'`.

### Server

```lua
-- Items
bridge.inventory.addItem(source, 'water', 3)                          -- retorna boolean
bridge.inventory.addItem(source, 'weapon_pistol', 1, { serial = 'X' })
bridge.inventory.removeItem(source, 'bread', 1)
bridge.inventory.hasItem(source, 'lockpick', 1)
bridge.inventory.canCarry(source, 'water', 5)
bridge.inventory.getItemMetadata(source, 'weapon_pistol')  -- ox_inventory only, nil en el resto

-- Stashes
bridge.inventory.registerStash('police_evidence', 'Evidence Locker', 'police', vec3(x, y, z))
bridge.inventory.isStashRegistered('police_evidence')
bridge.inventory.forceOpenStash(source, 'police_evidence')

-- Inventario de otro jugador
bridge.inventory.forceOpenPlayerInventory(source, targetServerId)

-- Catalogo
bridge.inventory.getAllItems()                        -- items definidos en el inventario

-- Items usables
bridge.inventory.registerUsableItem('bread', function(source, item)
    TriggerClientEvent('myscript:eatBread', source)
end)
bridge.inventory.isUsableItemRegistered('bread')      -- boolean
```

> **`bridge.inventory.registerUsableItem`** abstrae `ESX.RegisterUsableItem`, `QBCore.Functions.CreateUseableItem` y `exports.qbx_core:CreateUseableItem`. Es idempotente — registrar el mismo item dos veces no hace nada. Cuando `origen_inventory` esta activo, registra **solo** a traves de `exports.origen_inventory:CreateUseableItem` (nunca ademas del framework) para evitar que el callback dispare dos veces.

### Client

```lua
bridge.inventory.openStash('police_evidence')
bridge.inventory.openPlayerInventory(targetServerId)
bridge.inventory.getItemCount('water')
bridge.inventory.getImagePath()              -- retorna pattern NUI para iconos, usar con string.format
```

---

## Vehicle

### Shared

```lua
bridge.vehicle.normalizePlate('ABC 123 ')              -- retorna 'ABC 123'
```

### Server

```lua
bridge.vehicle.generatePlate()                          -- 8 chars random
bridge.vehicle.giveVehicle(source, 'adder', props)      -- inserta en owned_vehicles / player_vehicles
bridge.vehicle.getVehicleOwnerName('ABC12345')          -- retorna 'John Doe' o nil
```

### Client

```lua
bridge.vehicle.resolveModelHash('adder')

bridge.vehicle.spawnVehicle('adder', coords, heading, props, plate, function(vehicle)
    if vehicle then print('Spawned: ' .. vehicle) end
end)

local props = bridge.vehicle.getVehicleProperties(vehicle)
bridge.vehicle.setVehicleProperties(vehicle, props)

bridge.vehicle.getVehicleLabel('adder')                 -- retorna 'Adder'
```

---

## Callback

Sistema request/response cliente <-> servidor sin tener que registrar exports. **Namespace obligatorio** en el nombre para evitar colisiones entre recursos.

### Server

```lua
bridge.callback.register('nb-garages:getVehicles', function(source, respond, garageId)
    local vehicles = GetVehiclesForPlayer(source, garageId)
    respond(vehicles)
end)
```

### Client

```lua
bridge.callback.trigger('nb-garages:getVehicles', function(vehicles)
    if not vehicles then return end -- timeout de 15s dispara cb(nil), nil-guardear siempre
    print('Got ' .. #vehicles .. ' vehicles')
end, garageId)
```

---

## License

Solo server. Recupera identidad y licencias. Auto-detecta: `bcs_licensemanager`, `okokLicenses`, `esx_license`, metadata de QBCore/QBX, default de ESX.

```lua
local identity = bridge.license.getIdentity(source)
-- { firstname = 'John', lastname = 'Doe', dob = '1990-01-15', sex = 'M' }

local driver = bridge.license.getDriverLicense(source)
-- { hasLicense = true, label = 'Driver License' }

local weapon = bridge.license.getWeaponLicense(source)
-- { hasLicense = false, label = '' }
```

---

## Progress

Solo client. Usa `ox_lib` si esta disponible, cae a animacion nativa (no cancelable).

```lua
-- Progress bar simple
local completed = bridge.progress.show(5000, 'Buscando...')
if completed then print('Hecho') end

-- Con animacion
local completed = bridge.progress.show(3000, 'Reparando...', {
    dict = 'mini@repair',
    name = 'fixing_a_player',
})
```

Devuelve `true` si termino, `false` si se cancelo (solo ox_lib soporta cancelacion — el fallback nativo siempre retorna `true`).

---

## Event

Namespace de conveniencia para hooks de ciclo de vida de recurso y jugador. Los metodos disponibles difieren entre server y client.

### Server

```lua
bridge.event.onPlayerLoaded(function(source) end)
bridge.event.onPlayerUnloaded(function(source) end)   -- QBX: debounced 1s
bridge.event.onResourceStart(function(resourceName) end)
bridge.event.onResourceStop(function(resourceName) end)
bridge.event.onSelfStart(function() end)               -- solo cuando ARRANCA tu propio recurso
bridge.event.onSelfStop(function() end)                -- solo cuando PARA tu propio recurso
```

### Client

```lua
bridge.event.onPlayerLoaded(function() end)
bridge.event.onPlayerUnloaded(function() end)
bridge.event.onPlayerSpawned(function() end)  -- ESX: playerSpawned; QBCore/QBX: QBCore:Client:OnPlayerLoaded
bridge.event.onResourceStart(function(resourceName) end)
bridge.event.onResourceStop(function(resourceName) end)
```

`bridge.event.onPlayerLoaded` / `onPlayerUnloaded` son wrappers de conveniencia de `bridge.player.onPlayerLoaded` / `onPlayerUnloaded` — hacen exactamente lo mismo. Usa `bridge.event.*` para dejar claro que el codigo es de ciclo de vida.

```lua
-- server: limpieza al detener el recurso
bridge.event.onSelfStop(function()
    -- liberar recursos, cerrar conexiones, etc.
end)
```

---

## Log — auditoria (v2.2.0)

Registro de auditoria de produccion — **no** esta gateado por `Config.Debug` / `BridgeConfig.Debug`. `bridge.log` existe para eventos que siempre tienen que quedar registrados (cambios de dinero, acciones de admin, spawns de items, acceso a stashes, etc.), a diferencia de `Debugger()`, que es una herramienta de desarrollo que podes silenciar.

```lua
bridge.log.createLog(category, title, message, data, mention)
```

Despacha al pipeline de logs que ya tenga el servidor, sin que el sitio de la llamada tenga que ramificar por `bridge.Framework`:

1. **qb-log** — si el recurso `qb-log` esta corriendo (ecosistema QBCore/QBX), via `qb-log:server:CreateLog`.
2. **Webhook de Discord** — `BridgeConfig.Logs.Webhooks[category]` o `.default`; funciona tambien en ESX, que no tiene un recurso de logs nativo. `data` se renderiza como campos del embed.
3. **`Debugger` fallback** — si no hay ningun sink configurado.

Retorna `true` cuando un sink real (qb-log o webhook) proceso el log, `false` en el fallback de `Debugger`.

```lua
bridge.log.createLog('money', 'Retiro de banco',
    ('%s retiro $%s del banco'):format(bridge.player.getPlayerName(source), amount),
    {
        source     = source,
        identifier = bridge.player.getIdentifier(source),
        amount     = amount,
        newBalance = bridge.player.getMoney(source, 'bank'),
        reason     = reason,
    }
)
```

Pasa un `data` estructurado, no solo un mensaje en texto plano — el sink de webhook lo renderiza como campos individuales del embed, utiles para buscar y reconciliar despues.

> **SEGURIDAD:** nunca commitees una URL de webhook real como `BridgeConfig.Logs.Webhooks.default`. Dejala vacia y configurala por servidor (o via convar). Publicar un webhook real en el repo es una fuga de credenciales.

---

## UI — hooks de ciclo de vida (v2.2.0)

Solo client. Contrato generico para que recursos con menus (stashes, tiendas, mesas de crafteo, etc.) nunca tengan que ramificar por framework ni por sistema de inventario en su propio codigo de apertura/cierre.

```lua
bridge.ui.beforeOpening(type)   -- antes de abrir tu propio menu/NUI
bridge.ui.afterClosing(type)    -- justo despues de que tu menu/NUI se cierra
bridge.ui.beforeAction(action)  -- antes de una accion bloqueante (crafteo, hackeo, lockpicking, etc.)
bridge.ui.afterAction(action, ok) -- justo despues de que la accion termina, exitosa o no
```

`beforeOpening` cierra `ox_inventory` si estaba abierto (evita dos UI superpuestas). `beforeAction` / `afterAction` setean y liberan `LocalPlayer.state.invBusy` via statebag, para que `ox_inventory` y otros recursos que respeten ese statebag rechacen movimiento de items concurrente. `type` y `action` son identificadores libres tuyos — nb-bridge no los valida, solo existen para que tus pares queden faciles de gregar y loguear.

```lua
-- Client: menu de stash propio
bridge.ui.beforeOpening('stash')
-- ... abrir tu NUI / menu de ox_lib ...

bridge.ui.afterClosing('stash')
```

```lua
-- Client: accion bloqueante de crafteo
bridge.ui.beforeAction('crafting')
local completed = bridge.progress.show(5000, 'Crafteando...', anim)
bridge.ui.afterAction('crafting', completed)
```

Estos hooks son redefinibles desde `overrides/client/`.

---

## Diagnostics

Solo server. Retorna un snapshot del estado en runtime de nb-bridge.

```lua
local diag = bridge.diagnostics()
-- {
--   version = '2.2.0',
--   framework = 'QBX',
--   inventorySystem = 'ox_inventory',
--   side = 'server',
--   uptime = 142300,
--   features = { ox_lib = true, ox_inventory = true, oxmysql = true, ... },
--   missing = {},
-- }
```

Tambien esta disponible como export directo, sin pasar por `get()`:

```lua
local diag = exports['nb-bridge']:diagnostics()
print(diag.framework, diag.inventorySystem)
```

### Comando `/nbdiag`

Admin o consola. Sin argumentos — imprime el snapshot completo en la consola del servidor y lo envia al chat de admin.

---

## Namespaces y exports — vision general

nb-bridge **ya no publica un export por metodo**. Los unicos dos exports de FiveM son:

| Export | Retorna | Uso |
|--------|---------|-----|
| `exports['nb-bridge']:get()` | La tabla `Bridge` completa | Llamalo una vez por archivo, guardalo en `local bridge` |
| `exports['nb-bridge']:diagnostics()` | Snapshot de diagnostico | Callable directo sin pasar por `get()` |

Todo lo demas se llama a traves de la tabla que retorna `get()`. Referencia rapida de que namespace vive en que lado:

| Namespace | Lado | Contenido |
|-----------|------|-----------|
| `bridge.player` | Server + Client | Jugadores, dinero, trabajos, gangs, permisos, eventos |
| `bridge.inventory` | Server + Client | Items, stashes, items usables |
| `bridge.vehicle` | Server + Client | Matriculas, spawn, propiedades |
| `bridge.notify` | Server + Client | Notificaciones |
| `bridge.callback` | Server + Client | Callbacks de servidor |
| `bridge.license` | Solo server | Identidad y licencias |
| `bridge.progress` | Solo client | Barras de progreso |
| `bridge.event` | Server + Client | Hooks de ciclo de vida |
| `bridge.log` | Solo server | Auditoria *(v2.2.0)* |
| `bridge.ui` | Solo client | Hooks de UI *(v2.2.0)* |

Para la firma completa de cada metodo, parametros y ejemplos extendidos, la referencia tecnica en ingles vive en `docs/api.md` dentro del repositorio de nb-bridge.
