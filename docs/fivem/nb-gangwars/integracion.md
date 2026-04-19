# Integracion con tu sistema de gangs

nb-gangwars es agnostico respecto a tu sistema de gangs — se integra mediante tres funciones en `config.lua` que tu completas. Aqui van ejemplos listos para pegar.

---

## nb-jobscreator (default)

Por defecto el config ya asume que usas [nb-jobscreator](../nb-jobmanagers/index.md) (del suite de Neenbyss).

```lua
function GetPlayerGangClient()
    return exports['nb-jobscreator']:GetPlayerGang()
end

function GetPlayerGangServer(src)
    return exports['nb-jobscreator']:GetPlayerGang(src)
    -- debe devolver { name = 'vagos', label = 'Vagos' } o nil
end

function AddGangMoney(gangName, amount)
    exports['nb-jobscreator']:AddGangMoney(gangName, amount)
end
```

---

## ESX + societies

```lua
function GetPlayerGangClient()
    local xPlayer = ESX.GetPlayerData()
    if xPlayer and xPlayer.job and xPlayer.job.name ~= 'unemployed' then
        return xPlayer.job.name
    end
end

function GetPlayerGangServer(src)
    local xPlayer = ESX.GetPlayerFromId(src)
    if xPlayer then
        return { name = xPlayer.job.name, label = xPlayer.job.label }
    end
end

function AddGangMoney(gangName, amount)
    TriggerEvent('esx_addonaccount:getSharedAccount', 'society_' .. gangName, function(acc)
        if acc then acc.addMoney(amount) end
    end)
end
```

> Si usas un addon especifico de gangs (tipo `esx_gangs`), cambia `job` por `gang` en las llamadas y el prefijo de la society a `'gang_'`.

---

## QBCore + qb-management

```lua
function GetPlayerGangClient()
    local playerData = Bridge.GetPlayerData()
    if playerData and playerData.gang and playerData.gang.name ~= 'none' then
        return playerData.gang.name
    end
end

function GetPlayerGangServer(src)
    local player = Bridge.GetPlayer(src)
    if player and player.PlayerData.gang and player.PlayerData.gang.name ~= 'none' then
        return { name = player.PlayerData.gang.name, label = player.PlayerData.gang.label }
    end
end

function AddGangMoney(gangName, amount)
    exports['qb-management']:AddMoney(gangName, amount)
end
```

---

## Sistema custom

Cualquier sistema funciona mientras respetes las firmas:

- `GetPlayerGangClient() -> string|nil` — nombre interno.
- `GetPlayerGangServer(src) -> { name: string, label: string }|nil`.
- `AddGangMoney(gangName, amount) -> void` — deposita en la cuenta / wallet / lo que prefieras.

---

## Gotchas

- **Sin `AddGangMoney`**, la captura se registra pero la recompensa no llega — el script arranca sin errores y confunde al admin. Confirma con un toast/print que el dinero se deposito.
- **`GetPlayerGangClient` tiene que ser estable**: si tu sistema devuelve `nil` mientras el PlayerData carga, el jugador quedara fuera hasta refrescar. Valida tras `OnPlayerLoaded`.
- **Admin requerido**: el panel (`/gangzones`) depende de `Bridge.IsAdmin(src)`. Revisa tu `Config.AdminGroups` en nb-bridge o en el script consumidor.
- **Zonas sin gang atacante**: si todos los jugadores de una gang salen durante una fase, la captura se cancela. Esto es intencional para forzar presencia constante.
- **Un servidor authoritative**: el cliente solo informa; las transiciones de fase y la recompensa se calculan en el servidor. No confies en eventos client-side para cuadrar la economia.
