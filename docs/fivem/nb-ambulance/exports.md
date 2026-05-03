# Exports

API completa para integraciones desde otros recursos.

---

## Cliente

### Death state

```lua
exports['nb-ambulance']:isDead()              -- bool: dead o respawning
exports['nb-ambulance']:isInLastStand()       -- bool: en writhe loop
exports['nb-ambulance']:getDeathState()       -- 'idle' | 'laststand' | 'dead' | 'respawning'
exports['nb-ambulance']:reviveLocal(opts)     -- forzar revive local del propio jugador
```

`opts` opcional: `{ health = 200, armor = 100 }`.

### PVP-safe

```lua
exports['nb-ambulance']:setDeathDisabled(bool)  -- flip flag uno mismo
exports['nb-ambulance']:isDeathDisabled()       -- bool
```

Ver **[PVP-safe death](pvp.md)** para detalles.

### Heridas

```lua
exports['nb-ambulance']:getInjuries()    -- tabla bone_id -> { fracture, bleeding, burn, suffocating }
exports['nb-ambulance']:hasInjuries()    -- bool
exports['nb-ambulance']:clearInjuries()  -- limpiar todas
```

### Hospitales

```lua
exports['nb-ambulance']:getHospitals()  -- snapshot del cache de hospitales activos
exports['nb-ambulance']:getNearestBed(coords)  -- (hospital, bed, distance)
```

### Revive

```lua
exports['nb-ambulance']:isReviving()  -- bool: el jugador local esta haciendo CPR/defib
```

---

## Server

### Death + revive

```lua
exports['nb-ambulance']:setDeathDisabled(playerId, bool)
exports['nb-ambulance']:isDeathDisabled(playerId)
exports['nb-ambulance']:revivePlayer(playerId, opts)  -- opts: { health, armor }
exports['nb-ambulance']:getLastDeath(playerId)        -- { time, weapon, killer } o nil
```

### Heridas

```lua
exports['nb-ambulance']:getInjuries(playerId)        -- snapshot
exports['nb-ambulance']:clearInjuries(playerId)      -- limpiar todas + restaurar 100hp
exports['nb-ambulance']:useHealItem(playerId, itemName)  -- aplicar item por nombre
```

`useHealItem` valida `Bridge.HasItem`, consume con `Bridge.RemoveItem` y aplica el efecto.

### Hospitales

```lua
exports['nb-ambulance']:getHospitals()    -- cache server actual
exports['nb-ambulance']:reloadHospitals() -- forzar reload desde DB + broadcast
```

### Dispatch

```lua
exports['nb-ambulance']:broadcastDistress(playerSrc, coords)
exports['nb-ambulance']:getActiveCalls()
```

`broadcastDistress` crea una llamada en nombre de `playerSrc`. Si `coords` es nil, usa la posicion actual del player.

### Items para inventarios

```lua
exports['nb-ambulance']:use_bandage(...)
exports['nb-ambulance']:use_splint(...)
exports['nb-ambulance']:use_tourniquet(...)
exports['nb-ambulance']:use_burn_dressing(...)
exports['nb-ambulance']:use_oxygen_mask(...)
exports['nb-ambulance']:use_medikit(...)
```

Estos siguen el contrato de ox_inventory (`event, item, inventory, slot, data`). Para registrarlos con ox:

```lua
['bandage'] = {
    label = 'Bandage', weight = 50,
    server = { export = 'nb-ambulance.use_bandage' },
}
```

---

## Eventos cliente

```lua
-- damage / death
RegisterNetEvent('nb-ambulance:client:died')              -- { weapon, killer }
RegisterNetEvent('nb-ambulance:client:enteredLastStand')  -- { weapon, killer }
RegisterNetEvent('nb-ambulance:client:revived')           -- { method, medic }
RegisterNetEvent('nb-ambulance:client:respawned')         -- vec4 target
RegisterNetEvent('nb-ambulance:client:deathIntercepted')  -- { weapon, killer } - flag PVP

-- distress
RegisterNetEvent('nb-ambulance:client:distressInbound')   -- { callerSrc, callerName, coords }

-- revive
RegisterNetEvent('nb-ambulance:client:reviveAttemptInbound')  -- { method, medicSrc, medicName }
```

---

## Eventos server (para integraciones reversas)

Si tu script quiere disparar acciones de nb-ambulance:

```lua
-- crear distress como otro caller
TriggerEvent('nb-ambulance:server:distressCall')  -- usa source como caller

-- aplicar item de heridas
TriggerEvent('nb-ambulance:server:useHealItem', 'bandage')

-- log de muerte (no recomendado, lo hace el cliente automaticamente)
TriggerEvent('nb-ambulance:server:logDeath', { weapon, killer, from_ls })

-- recargar hospitales (admin)
TriggerEvent('nb-ambulance:server:reloadHospitals')
```

---

## Statebags (lectura cross-script)

Cualquier script puede leer estos sin necesidad de exports:

```lua
Player(src).state.nb_state           -- 'idle' | 'laststand' | 'dead' | 'respawning'
Player(src).state.nb_isDead          -- bool
Player(src).state.nb_inLaststand     -- bool
Player(src).state.nb_disableDeath    -- bool (PVP-safe flag)
Player(src).state.nb_injuries        -- tabla de heridas
```

Util para HUDs, panel admin, integraciones con scripts de status / hambre / sed que necesitan saber si el player esta downed.
