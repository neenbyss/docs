# Exports

nb-garages expone una API server y client. Ademas se registra como proveedor de `qb-vehiclekeys` — los recursos que esperen esa API funcionan sin cambios.

---

## Server

### Garajes

```lua
local garages = exports['nb-garages']:GetGarages()
-- table[]: lista cacheada de todos los garajes.

local g = exports['nb-garages']:GetGarage('downtown_public')
-- table|nil: una definicion de garaje con todos sus campos.

local can = exports['nb-garages']:CanAccessGarage(source, 'police_mission_row')
-- boolean: respeta restriction_type + value + funciones custom en bridge/.

local ok = exports['nb-garages']:CreateGarage({
    garage_id = 'pillbox_public',
    label     = 'Pillbox Public',
    type      = 'public',
    coords    = vector3(214.8, -809.4, 30.5),
    spawn_coords = vector3(220.0, -805.0, 30.8),
    vehicle_type = 'car',
    category     = 'car',
})
-- boolean.

local ok = exports['nb-garages']:DeleteGarage('pillbox_public')
-- boolean.
```

### Vehiculos

```lua
local vehicles = exports['nb-garages']:GetPlayerVehicles(source, 'downtown_public')
-- table[]: los coches del jugador en ese garaje.

local ok = exports['nb-garages']:StoreVehicle(source, 'downtown_public', 'ABC12345', props)
-- boolean: guarda coche + props.

local ok, data = exports['nb-garages']:TakeOutVehicle(source, vehicleId, 'downtown_public')
-- boolean + tabla con el vehicle data.

exports['nb-garages']:SendToDepot(plate, fee, reason)
-- void: mueve al depot.
```

### Casa

```lua
local ok, err = exports['nb-garages']:RegisterHouseGarage(source, {
    garage_id   = 'house_42',
    label       = 'Casa de Tony',
    coords      = vector3(x, y, z),
    spawn_coords= vector3(sx, sy, sz),
    house_id    = 42,
})

exports['nb-garages']:UnregisterHouseGarage(source, 'house_42')
exports['nb-garages']:RefreshHouseGarages(source)
```

### Llaves

```lua
exports['nb-garages']:GiveKeys(playerId, plate)   -- boolean
exports['nb-garages']:RemoveKeys(playerId, plate) -- boolean
exports['nb-garages']:HasKeys(playerId, plate)    -- boolean

-- Alias compatibles con recursos que esperan qb-vehiclekeys:
exports['qb-vehiclekeys']:HasKeys(playerId, plate)
```

### Diagnostico

```lua
local stats = exports['nb-garages']:GetPersistenceStats()
-- { tracked, db_rows, stale, impounded }
```

---

## Client

```lua
exports['nb-garages']:OpenGarage('downtown_public')
exports['nb-garages']:CloseGarage()

exports['nb-garages']:StoreCurrentVehicle('downtown_public')
-- guarda el vehiculo actual si estas a menos de Config.Garage.StoreDistance.

exports['nb-garages']:HasKeys(plate) -- boolean
exports['qb-vehiclekeys']:HasKeys(plate) -- alias
```

---

## Ejemplos

### Dar coche a un ganador de evento

```lua
-- server
local plate = Bridge.GeneratePlate()
Bridge.GiveVehicle(source, 'adder', { plate = plate })
exports['nb-garages']:GiveKeys(source, plate)
```

### Impoundar todos los coches del barrio

```lua
-- server (por un policia)
for _, ped in pairs(GetAllPeds()) do
    local veh = GetVehiclePedIsIn(ped, false)
    if veh ~= 0 then
        local plate = Bridge.NormalizePlate(GetVehicleNumberPlateText(veh))
        exports['nb-garages']:SendToDepot(plate, 1500, 'operativo_policial')
    end
end
```

### Verificar acceso antes de mostrar un blip custom

```lua
-- server
local ok = exports['nb-garages']:CanAccessGarage(source, 'bunker_vagos')
if not ok then return end
TriggerClientEvent('my-script:showBlip', source)
```

---

## Callbacks de admin (via `Bridge.TriggerServerCallback`)

Para recursos avanzados que reemplazen la UI del creador, los 11 callbacks de admin estan disponibles bajo el prefijo `nb-garages:admin:*`. Todos validan con `Bridge.IsAdmin`. Consulta el codigo en `server/panel.lua` para los payloads exactos — no se considera API estable publica.

---

## Compatibilidad qb-vehiclekeys

nb-garages se registra con:

```lua
-- fxmanifest.lua
provide 'qb-vehiclekeys'
```

y emite/escucha los eventos de qb-vehiclekeys. Cualquier recurso que dependa de `'qb-vehiclekeys'` o de los eventos `qb-vehiclekeys:*` seguira funcionando sin cambios.
