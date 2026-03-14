# Exports

Otros recursos pueden usar **nb-cars** como complemento para dar vehículos, consultar la BD o dar llaves. Todos los exports son **server-side**.

---

## GiveVehicleToPlayer

Da un vehículo a un jugador: spawn en cliente, guardado en BD y entrega de llaves según `Config.KeySystem`.

```lua
-- Desde otro recurso (server)
local success = exports['nb-cars']:GiveVehicleToPlayer(targetSource, 'zentorno', 'ABC 123')
-- Si no pasas matrícula, se genera una aleatoria
local ok = exports['nb-cars']:GiveVehicleToPlayer(targetSource, 'adder')
```

| Parámetro      | Tipo   | Descripción                                      |
|----------------|--------|--------------------------------------------------|
| `targetSource` | number | ID del jugador que recibe el vehículo           |
| `model`        | string | Modelo del vehículo (ej. `'zentorno'`, `'adder'`) |
| `plate`        | string | *(opcional)* Matrícula; si no se pasa, se genera una |

**Returns:** `boolean` — `true` si se envió el evento correctamente (jugador existe), `false` si el jugador no existe.

---

## DeleteVehicleByPlate

Elimina un vehículo de la base de datos por matrícula (owned_vehicles / player_vehicles según framework).

```lua
local deleted = exports['nb-cars']:DeleteVehicleByPlate('ABC 123')
if deleted then
    print('Vehículo eliminado de la BD')
end
```

| Parámetro | Tipo   | Descripción           |
|-----------|--------|-----------------------|
| `plate`  | string | Matrícula del vehículo |

**Returns:** `boolean` — `true` si se eliminó, `false` si no existía o error.

---

## GetPlayerVehicles

Devuelve la lista de vehículos que tiene registrados un jugador en la BD.

```lua
local vehicles = exports['nb-cars']:GetPlayerVehicles(playerId)
-- vehicles = { { plate = 'ABC 123', model = 'zentorno' }, ... }
```

| Parámetro      | Tipo   | Descripción        |
|----------------|--------|--------------------|
| `playerSource` | number | ID del jugador     |

**Returns:** `table` — Lista de `{ plate = string, model = string }`. `nil` si el jugador no existe.

---

## DoesPlateExist

Comprueba si una matrícula está registrada en la base de datos.

```lua
if exports['nb-cars']:DoesPlateExist('ABC 123') then
    print('La matrícula ya está en uso')
end
```

| Parámetro | Tipo   | Descripción           |
|-----------|--------|-----------------------|
| `plate`  | string | Matrícula a comprobar |

**Returns:** `boolean`

---

## GiveKeysToPlayer

Da las llaves de un vehículo a un jugador usando el sistema configurado en `Config.KeySystem` (brutal_keys, qb-vehiclekeys, etc.).

```lua
exports['nb-cars']:GiveKeysToPlayer(source, 'ABC 123', 'zentorno')
```

| Parámetro | Tipo   | Descripción                                      |
|-----------|--------|--------------------------------------------------|
| `source`  | number | ID del jugador                                  |
| `plate`  | string | Matrícula del vehículo                          |
| `model`  | string | *(opcional)* Modelo; algunos sistemas lo necesitan |

---

## GeneratePlate

Genera una matrícula aleatoria según el formato configurado (`Config.PlateFormat`).

```lua
local plate = exports['nb-cars']:GeneratePlate()
-- Ejemplo: 'KLM 456'
```

**Returns:** `string`

---

## RegisterVehicleAsOwned

Registra un vehículo como propiedad de un jugador en la BD **sin spawn**. Útil cuando otro script ya spawneó el vehículo o cuando solo quieres registrar la propiedad (compras, eventos, migraciones).

```lua
local props = { plate = 'ABC 123', model = 'zentorno', -- ... }
local ok = exports['nb-cars']:RegisterVehicleAsOwned(
    targetSource,
    'ABC 123',
    props,
    'zentorno',
    'car',   -- tipo (opcional, por defecto 'car')
    true     -- dar llaves (opcional, por defecto true)
)
```

| Parámetro      | Tipo   | Descripción                                      |
|----------------|--------|--------------------------------------------------|
| `targetSource` | number | ID del jugador dueño                            |
| `plate`       | string | Matrícula                                        |
| `vehicleProps`| table \| string | Props del vehículo (tabla o JSON)     |
| `model`       | string | Modelo del vehículo                             |
| `vehicleType` | string | *(opcional)* Tipo, por defecto `'car'`          |
| `giveKeys`    | boolean| *(opcional)* Si dar llaves; por defecto `true`   |

**Returns:** `boolean` — `true` si se insertó correctamente, `false` si el jugador no existe o datos inválidos.

---

## Ejemplo de uso: recompensa desde otro script

```lua
-- En tu recurso de recompensas/misiones (server)
RegisterNetEvent('mi_script:server:darPremioVehiculo', function()
    local src = source
    local ok = exports['nb-cars']:GiveVehicleToPlayer(src, 'adder')
    if ok then
        print('Premio enviado')
    else
        print('Jugador no encontrado')
    end
end)
```

Asegúrate de que **nb-cars** esté en `dependencies` de tu `fxmanifest.lua` si lo necesitas cargar después:

```lua
dependencies {
    'nb-cars',
    -- ...
}
```
