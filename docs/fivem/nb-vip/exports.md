# Exports

Otros recursos pueden interactuar con nb-vip mediante exports server-side.

---

## GetPlayerCoins

Obtiene el balance de coins de un jugador.

```lua
local coins = exports['nb-vip']:GetPlayerCoins(source)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |

**Returns:** `number` — Balance actual de coins. `0` si el jugador no existe.

---

## AddPlayerCoins

Agrega coins al balance de un jugador.

```lua
local success = exports['nb-vip']:AddPlayerCoins(source, 500, 'premio evento')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |
| `amount` | number | Cantidad de coins a agregar |
| `reason` | string | *(opcional)* Razon de la transaccion (por defecto `'external'`) |

**Returns:** `boolean` — `true` si se agregaron correctamente.

---

## RemovePlayerCoins

Remueve coins del balance de un jugador.

```lua
local success = exports['nb-vip']:RemovePlayerCoins(source, 200, 'compra custom')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |
| `amount` | number | Cantidad de coins a remover |
| `reason` | string | *(opcional)* Razon de la transaccion (por defecto `'external'`) |

**Returns:** `boolean` — `true` si se removieron correctamente.

---

## Ejemplo: dar coins como recompensa

```lua
-- En tu recurso (server-side)
RegisterNetEvent('mi_script:server:darRecompensa', function()
    local src = source
    local ok = exports['nb-vip']:AddPlayerCoins(src, 1000, 'recompensa mision')
    if ok then
        print('Se dieron 1000 coins al jugador ' .. src)
    end
end)
```

Asegurate de que **nb-vip** este en `dependencies` de tu `fxmanifest.lua`:

```lua
dependencies {
    'nb-vip',
}
```
