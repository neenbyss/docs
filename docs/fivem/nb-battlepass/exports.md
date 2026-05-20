# Exports

Otros recursos pueden interactuar con nb-battlepass mediante exports **server-side**.

---

## AddXP

Otorga XP a un jugador en la season activa. **Esta es la unica forma de hacer que un jugador suba de nivel.**

```lua
local ok = exports['nb-battlepass']:AddXP(source, 100, 'mision_diaria')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |
| `amount` | number | XP a añadir (debe ser > 0) |
| `reason` | string | *(opcional)* Etiqueta para debug. No se persiste. |

**Returns:** `boolean` — `true` si se otorgo, `false` si no hay season activa, identifier invalido o amount <= 0.

**Efectos secundarios:**

- Si el jugador sube de nivel, recibe una notificacion (si `Config.NotifyOnLevelUp = true`).
- Si tiene la UI abierta, se actualiza en tiempo real.
- El nivel hace cap en `season.max_level`.

---

## GrantPremium

Otorga el pase premium a un jugador en la season activa.

```lua
local ok = exports['nb-battlepass']:GrantPremium(source)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |

**Returns:** `boolean` — `true` si se otorgo, `false` si no hay season activa.

**Uso tipico:** integraciones con tienda Tebex, recompensas de evento, sistemas VIP, etc.

---

## GetPlayerData

Devuelve el estado completo del jugador en la season activa (util para HUDs externos).

```lua
local data = exports['nb-battlepass']:GetPlayerData(source)
```

**Returns:** `table` con esta forma:

```lua
{
    season = {
        id, name, description, max_level, xp_per_level,
        premium_enabled, premium_purchasable, premium_price,
    },
    tiers = { { id, level, free_reward, premium_reward }, ... },
    player = {
        xp          = 1250,
        level       = 1,
        has_premium = false,
        claims      = { { tier_id = 5, track = 'free' }, ... },
    },
    is_admin = false,
}
```

Si no hay season activa, `season = nil`.

---

## Ejemplos rapidos

### Dar XP por completar una mision

```lua
-- En tu script de misiones
RegisterNetEvent('misiones:completar', function(missionId)
    local src = source
    -- ... tu logica ...
    exports['nb-battlepass']:AddXP(src, 250, 'mision_' .. missionId)
end)
```

### Dar XP por tiempo jugado

```lua
CreateThread(function()
    while true do
        Wait(5 * 60 * 1000)  -- cada 5 min
        for _, src in ipairs(GetPlayers()) do
            exports['nb-battlepass']:AddXP(tonumber(src), 50, 'tiempo_jugado')
        end
    end
end)
```

### Tienda Tebex: otorgar premium cuando se compra

```lua
-- En tu integracion Tebex/donator
RegisterNetEvent('tebex:premium_purchased', function(playerId)
    exports['nb-battlepass']:GrantPremium(playerId)
end)
```

### Mostrar nivel en un HUD externo

```lua
-- Server
RegisterNetEvent('miHud:getBattlepass', function()
    local src = source
    local data = exports['nb-battlepass']:GetPlayerData(src)
    if data and data.player then
        TriggerClientEvent('miHud:setBattlepass', src, {
            level = data.player.level,
            xp    = data.player.xp,
        })
    end
end)
```

---

## Dependencia en `fxmanifest.lua`

Si tu recurso depende de nb-battlepass, declaralo:

```lua
dependencies {
    'nb-battlepass',
}
```

---

## ¿No esta lo que necesitas?

Si necesitas un export adicional (ej. resetear progreso, listar tops, leaderboards, etc.), abre un ticket en Tebex.
