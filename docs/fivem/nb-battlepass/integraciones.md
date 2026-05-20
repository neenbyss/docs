# Integraciones

Ejemplos completos de como integrar nb-battlepass con otros sistemas. Copia, pega, adapta.

---

## 1. Misiones diarias (sistema externo)

Tu script de misiones llama al export cuando el jugador completa una mision.

```lua
-- nb-mydailies/server/main.lua
RegisterNetEvent('nb-mydailies:complete', function(missionId)
    local src = source

    local mission = Missions[missionId]
    if not mission then return end

    -- ... tu logica de validacion ...

    -- Otorgar XP al battlepass
    local xp = mission.battlepass_xp or 100
    exports['nb-battlepass']:AddXP(src, xp, 'daily_' .. missionId)

    -- Otorgar tu propia recompensa
    Bridge.Money.Add(src, 'bank', mission.cash_reward)
end)
```

---

## 2. Kills / PvP (otorgar XP por matar)

```lua
-- En tu sistema de combate o estadisticas
AddEventHandler('baseevents:onPlayerKilled', function(killerId, data)
    local src = source
    if not killerId or killerId == src then return end

    -- 50 XP por kill
    exports['nb-battlepass']:AddXP(killerId, 50, 'pvp_kill')
end)
```

!!! warning
    Para prevenir farmeo, valida que el killer y la victima no compartan IP/identifier, añade cooldown, etc.

---

## 3. Tiempo jugado

Da XP pasivo cada X minutos a todos los jugadores online.

```lua
CreateThread(function()
    while true do
        Wait(5 * 60 * 1000)  -- 5 min
        for _, src in ipairs(GetPlayers()) do
            src = tonumber(src)
            -- Opcional: verificar que no este AFK
            exports['nb-battlepass']:AddXP(src, 25, 'time_played')
        end
    end
end)
```

---

## 4. nb-vip → Comprar premium con coins

**`shared/config.lua`:**

```lua
Config.Premium = {
    PurchaseButtonEnabled = true,

    CanPurchase = function(src, season)
        local price = season.premium_price or 0
        if price <= 0 then return false end

        local coins = exports['nb-vip']:GetPlayerCoins(src) or 0
        if coins < price then
            return false, 'No tienes suficientes coins.'
        end
        return true
    end,

    OnPurchase = function(src, season)
        return exports['nb-vip']:RemovePlayerCoins(
            src, season.premium_price, 'battlepass_premium'
        )
    end,
}
```

En el panel admin, configura `Premium Price` con la cantidad de **coins**, no de dinero del banco.

---

## 5. nb-vip → Dar coins como recompensa

**`shared/rewards.lua`:**

```lua
Config.RewardHandlers['coins'] = function(src, reward)
    local amount = tonumber(reward.amount) or 0
    if amount <= 0 then return false, 'invalid_data' end
    return exports['nb-vip']:AddPlayerCoins(src, amount, 'battlepass') or false
end
```

JSON en la DB de un tier:
```json
{ "type": "coins", "amount": 250, "label": "250 Coins", "icon": "icon-coins" }
```

---

## 6. Tebex package → Grant premium

Cuando un jugador compra un paquete en tu tienda Tebex, otorgale premium.

```lua
-- En tu integracion Tebex
RegisterNetEvent('myTebex:packagePurchased', function(playerId, package)
    if package == 'battlepass_premium' then
        exports['nb-battlepass']:GrantPremium(playerId)
        Bridge.Notify(playerId, 'Pase premium activado!', 'success')
    end
end)
```

---

## 7. Discord webhook al subir de nivel

nb-battlepass no notifica a Discord por si solo. Si quieres logs, envuelve `AddXP`:

```lua
-- nb-mywrappers/server/main.lua
local function AddXPWithLog(src, amount, reason)
    local before = exports['nb-battlepass']:GetPlayerData(src)
    local oldLvl = before and before.player and before.player.level or 0

    exports['nb-battlepass']:AddXP(src, amount, reason)

    local after = exports['nb-battlepass']:GetPlayerData(src)
    local newLvl = after and after.player and after.player.level or 0

    if newLvl > oldLvl then
        PerformHttpRequest('https://discord.com/api/webhooks/...', function() end, 'POST',
            json.encode({ content = ('%s subio a nivel %d'):format(GetPlayerName(src), newLvl) }),
            { ['Content-Type'] = 'application/json' }
        )
    end
end

exports('AddXPWithLog', AddXPWithLog)
```

---

## 8. Multi-rewards (bundle) — recompensas combinadas

Añade un handler que ejecute varios subhandlers en cadena. Util para niveles de hito (10, 25, 50).

**`shared/rewards.lua`:**

```lua
Config.RewardHandlers['bundle'] = function(src, reward)
    for _, sub in ipairs(reward.items or {}) do
        local h = Config.RewardHandlers[sub.type]
        if h then
            local ok, err = h(src, sub)
            if not ok then return false, err end
        end
    end
    return true
end
```

Configura el tier con JSON:
```json
{
    "type": "bundle",
    "label": "Mega Drop Nivel 50",
    "icon": "icon-gift",
    "items": [
        { "type": "money",   "account": "bank", "amount": 100000 },
        { "type": "vehicle", "model": "adder" },
        { "type": "coins",   "amount": 1000 },
        { "type": "item",    "name": "diamond", "amount": 10 }
    ]
}
```

---

## 9. Jobs → XP por nomina

Cuando el jugador cobra su sueldo, dale XP proporcional.

```lua
-- En tu sistema de jobs / nomina
RegisterNetEvent('jobs:payday', function()
    local src = source
    local amount = GetPlayerSalary(src)
    Bridge.Money.Add(src, 'bank', amount)

    -- 1 XP por cada $100 de sueldo
    exports['nb-battlepass']:AddXP(src, math.floor(amount / 100), 'payday')
end)
```

---

## 10. Solo grant por admin (sin compra)

**`shared/config.lua`:**

```lua
Config.Premium = {
    PurchaseButtonEnabled = false,
    CanPurchase = function() return false end,
    OnPurchase  = function() return false end,
}
```

Y desde consola:

```cfg
battlepassadmin grant 12   # otorga premium al jugador con server ID 12
```

---

## Checklist al integrar

- [ ] ¿Mi recurso tiene `'nb-battlepass'` en `dependencies` de su `fxmanifest.lua`?
- [ ] ¿Estoy llamando los exports desde **server-side** unicamente?
- [ ] ¿Estoy validando `source` antes de pasarlo a `AddXP`?
- [ ] ¿Mis handlers custom **devuelven `true`/`false`** correctamente?
- [ ] ¿Mis handlers viven en `shared/rewards.lua` (no en `server/main.lua`)?
- [ ] Si modifique `Config.Premium`, ¿probe la compra end-to-end?
