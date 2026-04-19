# Modulos

nb-bridge esta dividido en modulos independientes. Cada uno expone funciones bajo el namespace `Bridge.*`. Tambien estan disponibles como exports (`exports['nb-bridge']:FuncName`).

---

## Framework

Auto-detecta ESX o QBCore y entrega una API comun.

### Server

```lua
-- Player
Bridge.GetPlayer(source)              -- xPlayer (ESX) o Player (QBCore)
Bridge.GetIdentifier(source)          -- license (ESX) o citizenid (QBCore)
Bridge.GetSSN(source)                 -- ESX only
Bridge.GetPlayerName(source)

-- Permisos
Bridge.GetGroup(source)               -- 'admin', 'user', ...
Bridge.SetGroup(source, group)        -- ESX only
Bridge.IsAdmin(source)                -- usa Config.AdminGroups

-- Dinero
Bridge.AddMoney(source, type, amount, reason)     -- type: 'cash' o 'bank'
Bridge.RemoveMoney(source, type, amount, reason)
Bridge.SetMoney(source, type, amount, reason)
Bridge.GetMoney(source, type)
Bridge.GetAccounts(source)            -- { cash = N, bank = N, ... }

-- Trabajo
Bridge.GetJob(source)                 -- { name, label, grade, grade_name, grade_label, grade_salary, onDuty }
Bridge.SetJob(source, job, grade, onDuty)
Bridge.GetGang(source)                -- QBCore only, nil en ESX

-- Metadata
Bridge.SetMeta(source, key, value, subKey)
Bridge.GetMeta(source, key, subKey)
Bridge.ClearMeta(source, key, subKey)

-- Varios
Bridge.GetPlayTime(source)            -- ESX only
Bridge.SetCoords(source, coords)
Bridge.GetCoords(source)
Bridge.TriggerClientEvent(source, event, ...)
Bridge.PlayerVar(source, key, value)
Bridge.ExecuteCommand(source, command) -- ESX only
Bridge.CreateBill(src, targetId, amount, desc, jobName)

-- Eventos
Bridge.OnPlayerLoaded(function(source, identifier) end)
```

### Client

```lua
Bridge.GetPlayerData()
Bridge.GetJob()                       -- mismo formato canonico que el server
Bridge.GetGang()
Bridge.GetMoney(type)
Bridge.GetAccounts()
Bridge.GetIdentifier()
Bridge.GetPlayerName()
Bridge.GetGroup()

-- Eventos
Bridge.OnPlayerLoaded(function() end)
Bridge.OnJobUpdate(function(job) end)
Bridge.OnGangUpdate(function(gang) end)
```

### Ejemplos

```lua
-- Server: bonus + notificacion
RegisterCommand('bonus', function(source)
    local name = Bridge.GetPlayerName(source)
    Bridge.AddMoney(source, 'bank', 5000, 'bonus_evento')
    Bridge.Notify(source, name .. ' recibio $5,000', 'success')
end)

-- Client: reaccionar a cambio de trabajo
Bridge.OnJobUpdate(function(job)
    print('Nuevo trabajo: ' .. job.name)
end)
```

---

## Notify

Sistema unificado de notificaciones. Auto-detecta ox_lib, ESX, QBCore o native.

```lua
-- Server
Bridge.Notify(source, 'Factura pagada', 'success')
Bridge.Notify(source, 'Dinero insuficiente', 'error')

-- Client
Bridge.ShowNotification('Item recibido', 'info')
```

**Tipos soportados:** `'success'`, `'error'`, `'info'`, `'warning'`.

---

## Inventory

Abstraccion multi-inventario. Soporta `ox_inventory`, `qb-inventory`, `qs-inventory` y defaults del framework.

### Server

```lua
-- Items
Bridge.AddItem(source, 'water', 3)                          -- retorna boolean
Bridge.AddItem(source, 'weapon_pistol', 1, { serial = 'X' })
Bridge.RemoveItem(source, 'bread', 1)
Bridge.HasItem(source, 'lockpick', 1)
Bridge.CanCarry(source, 'water', 5)

-- Stashes
Bridge.RegisterStash('police_evidence', 'Evidence Locker', 'police', vec3(x, y, z))
Bridge.IsStashRegistered('police_evidence')
Bridge.ForceOpenStash(source, 'police_evidence')

-- Inventario de otro jugador
Bridge.ForceOpenPlayerInventory(source, targetServerId)

-- Catalogo
Bridge.GetAllItems()                        -- items definidos en el inventario

-- Items usables (v1.1.0+)
Bridge.RegisterUsableItem('bread', function(source, item)
    TriggerClientEvent('myscript:eatBread', source)
end)
Bridge.IsUsableItemRegistered('bread')      -- boolean
```

> **`Bridge.RegisterUsableItem`** abstrae `ESX.RegisterUsableItem` y `QBCore.Functions.CreateUseableItem`. Se introdujo en v1.1.0 y lo usa internamente [nb-consumibles](../nb-consumibles/index.md).

### Client

```lua
Bridge.OpenStash('police_evidence')
Bridge.OpenPlayerInventory(targetServerId)
Bridge.GetItemCount('water')
Bridge.GetImagePath()              -- retorna pattern NUI para iconos
```

---

## Vehicle

### Shared

```lua
Bridge.NormalizePlate('ABC 123 ')              -- retorna 'ABC 123'
```

### Server

```lua
Bridge.GeneratePlate()                          -- 8 chars random
Bridge.GiveVehicle(source, 'adder', props)      -- inserta en owned_vehicles / player_vehicles
Bridge.GetVehicleOwnerName('ABC12345')          -- retorna 'John Doe' o nil
```

### Client

```lua
Bridge.ResolveModelHash('adder')

Bridge.SpawnVehicle('adder', coords, heading, props, plate, function(vehicle)
    if vehicle then print('Spawned: ' .. vehicle) end
end)

local props = Bridge.GetVehicleProperties(vehicle)
Bridge.SetVehicleProperties(vehicle, props)

Bridge.GetVehicleLabel('adder')                 -- retorna 'Adder'
```

---

## Callbacks

Sistema request/response cliente <-> servidor sin tener que registrar exports. **Namespace obligatorio** para evitar colisiones.

### Server

```lua
Bridge.CreateCallback('nb-garages:getVehicles', function(source, respond, garageId)
    local vehicles = GetVehiclesForPlayer(source, garageId)
    respond(vehicles)
end)
```

### Client

```lua
Bridge.TriggerServerCallback('nb-garages:getVehicles', function(vehicles)
    print('Got ' .. #vehicles .. ' vehicles')
end, garageId)
```

---

## Licenses

Solo server. Recupera identidad y licencias. Auto-detecta: `bcs_licensemanager`, `okokLicenses`, `esx_license`, metadata de QBCore, default de ESX.

```lua
local identity = Bridge.GetIdentity(source)
-- { firstname = 'John', lastname = 'Doe', dob = '1990-01-15', sex = 'M' }

local driver = Bridge.GetDriverLicense(source)
-- { hasLicense = true, label = 'Driver License' }

local weapon = Bridge.GetWeaponLicense(source)
-- { hasLicense = false, label = '' }
```

---

## Progress

Solo client. Usa `ox_lib` si esta disponible, cae a animacion nativa.

```lua
-- Progress bar simple
local completed = Bridge.Progress(5000, 'Buscando...')
if completed then print('Hecho') end

-- Con animacion
local completed = Bridge.Progress(3000, 'Reparando...', {
    dict = 'mini@repair',
    name = 'fixing_a_player',
})
```

Devuelve `true` si termino, `false` si se cancelo (solo ox_lib soporta cancelacion).

---

## Tabla completa de exports

| Export | Side | Modulo |
|--------|------|--------|
| `GetPlayer` | Server | Framework |
| `GetIdentifier` | Server | Framework |
| `GetSSN` | Server | Framework |
| `GetPlayerName` | Server / Client | Framework |
| `GetGroup` | Server / Client | Framework |
| `SetGroup` | Server | Framework |
| `IsAdmin` | Server | Framework |
| `AddMoney` / `RemoveMoney` / `SetMoney` / `GetMoney` / `GetAccounts` | Server | Framework |
| `GetJob` / `SetJob` / `GetGang` | Server / Client | Framework |
| `CreateBill` | Server | Framework |
| `OnPlayerLoaded` | Server / Client | Framework |
| `OnJobUpdate` / `OnGangUpdate` | Client | Framework |
| `Notify` | Server | Notify |
| `ShowNotification` | Client | Notify |
| `AddItem` / `RemoveItem` / `HasItem` / `CanCarry` | Server | Inventory |
| `RegisterStash` / `IsStashRegistered` / `ForceOpenStash` | Server | Inventory |
| `ForceOpenPlayerInventory` / `GetAllItems` | Server | Inventory |
| `RegisterUsableItem` / `IsUsableItemRegistered` | Server | Inventory *(v1.1.0+)* |
| `OpenStash` / `OpenPlayerInventory` / `GetItemCount` / `GetImagePath` | Client | Inventory |
| `NormalizePlate` | Both | Vehicle |
| `GeneratePlate` / `GiveVehicle` / `GetVehicleOwnerName` | Server | Vehicle |
| `ResolveModelHash` / `SpawnVehicle` / `GetVehicleProperties` / `SetVehicleProperties` / `GetVehicleLabel` | Client | Vehicle |
| `CreateCallback` | Server | Callbacks |
| `TriggerServerCallback` | Client | Callbacks |
| `GetIdentity` / `GetDriverLicense` / `GetWeaponLicense` | Server | Licenses |
| `Progress` | Client | Progress |
