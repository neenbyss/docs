# PVP-safe death

Permite que **otros scripts** (PVP, arenas, gulag, minijuegos, eventos staff) **suspendan el flujo de muerte** de nb-ambulance sin tocar el codigo. El jugador nunca entra en laststand ni ve la pantalla de muerte mientras el flag este activo — solo se restaura su HP en sitio y se dispara un evento que tu script puede capturar para correr su propio respawn.

---

## Por que existe

En servidores con muchos scripts de combate (battle royale, deathmatch, eventos), el flujo de muerte de un ambulancejob normal interrumpe la partida: el jugador entra en writhe loop, se le obliga a respawnear en hospital, paga tarifa, etc.

nb-ambulance da **tres mecanismos** para que tu script de PVP gane el control sobre la muerte de un jugador especifico, sin race conditions.

---

## Mecanismo 1: Export client

Desde el script PVP, en el **client del jugador afectado**:

```lua
-- al entrar a la arena
exports['nb-ambulance']:setDeathDisabled(true)

-- al salir
exports['nb-ambulance']:setDeathDisabled(false)
```

Equivalente server-side:

```lua
exports['nb-ambulance']:setDeathDisabled(playerId, true)
```

Internamente setea el statebag `nb_disableDeath` y `nb_disableDeath_manual` (asi las zonas auto no lo limpian al salir).

---

## Mecanismo 2: Statebag directo

Si prefieres no depender del export:

```lua
-- server
Player(src).state:set('nb_disableDeath', true, true)

-- client (de tu propio jugador)
LocalPlayer.state:set('nb_disableDeath', true, true)
```

nb-ambulance lee el statebag al momento de la muerte. Si esta `true`, intercepta antes de entrar en laststand:

1. `SetEntityHealth(ped, 200)`
2. `ClearPedBloodDamage(ped)`
3. `SetPlayerInvincible(false)`
4. Dispara `nb-ambulance:client:deathIntercepted` para que tu script tome el relevo.

---

## Mecanismo 3: Zonas configurables

Esferas en el mapa que activan el flag automaticamente al entrar y lo desactivan al salir.

```lua
Config.Pvp = {
    Zones = {
        { coords = vec3(2440.0, 4970.0, 47.0), radius = 60.0, label = 'sandy_arena' },
        { coords = vec3(-1100.0, 250.0, 70.0), radius = 80.0, label = 'mansion_dm' },
    },
    ZoneCheckInterval     = 750,   -- ms
    RestoreHealth         = 200,
    NotifyZoneTransitions = true,  -- toast al entrar/salir
}
```

Util para arenas estaticas. Las zonas y los flags manuales coexisten — si entras a una zona y luego sales, las llamadas manuales `setDeathDisabled(true)` siguen activas.

---

## Capturar la muerte interceptada

Cuando nb-ambulance intercepta, dispara `nb-ambulance:client:deathIntercepted` en el cliente afectado:

```lua
AddEventHandler('nb-ambulance:client:deathIntercepted', function(payload)
    -- payload = { weapon = hash, killer = serverId }

    -- corre tu propio flujo de respawn / muerte
    DoMyArenaRespawn()
end)
```

Esto te da el "killer + weapon" que ya capturo el listener de damage de nb-ambulance, asi no tienes que duplicar el listener en tu script.

---

## Ejemplo completo: minijuego de boxeo

```lua
-- server: tu-script/server.lua
RegisterNetEvent('boxing:start', function()
    local src = source
    Player(src).state:set('nb_disableDeath', true, true)
end)

RegisterNetEvent('boxing:end', function()
    local src = source
    Player(src).state:set('nb_disableDeath', false, true)
end)

-- client: tu-script/client.lua
AddEventHandler('nb-ambulance:client:deathIntercepted', function(payload)
    if not inBoxingMatch then return end

    -- knockout!
    showKOMessage()
    ResetBoxingRound()
end)
```

---

## Comando admin (debug)

```
/nb-ambulance-pvp <playerId> <0|1>
```

Solo desde consola/admin. Toggle manual del flag para testing.

---

## API completa

| Export | Lado | Que hace |
|--------|------|----------|
| `setDeathDisabled(bool)` | client | flip flag para uno mismo |
| `isDeathDisabled()` | client | bool |
| `setDeathDisabled(playerId, bool)` | server | flip flag de un target |
| `isDeathDisabled(playerId)` | server | bool |

| Statebag | Donde | Notas |
|----------|-------|-------|
| `Player(src).state.nb_disableDeath` | server-replicated | Source of truth. |
| `Player(src).state.nb_disableDeath_manual` | server-replicated | Indica que el flag fue puesto manualmente, no por zona. Las zonas no lo limpian al salir si esto es true. |
| `Player(src).state.nb_disableDeath_zone` | server-replicated | Indica que el flag esta activo por una zona. |

| Evento | Lado | Cuando |
|--------|------|--------|
| `nb-ambulance:client:deathIntercepted` | client | Una muerte fue interceptada. Payload: `{ weapon, killer }` |
