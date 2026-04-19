# Exports

nb-actions expone dos exports client-side para que otros recursos lean el estado de esposado / arrastre.

---

## IsPlayerHandcuffed

```lua
local cuffed = exports['nb-actions']:IsPlayerHandcuffed()
```

**Returns:** `boolean` — `true` si el jugador local esta esposado.

**Uso tipico:** bloquear acciones en otros scripts (abrir puertas, conducir, robar) mientras el jugador esta esposado.

```lua
-- En tu script
if exports['nb-actions']:IsPlayerHandcuffed() then
    Bridge.ShowNotification('No puedes hacer esto esposado', 'error')
    return
end
```

---

## IsPlayerDragged

```lua
local dragged = exports['nb-actions']:IsPlayerDragged()
```

**Returns:** `boolean` — `true` si el jugador esta siendo arrastrado por otro.

---

## Eventos que puedes escuchar

| Evento | Payload | Momento |
|--------|---------|---------|
| `nb-actions:client:arrestConfirmed` | `targetServerId` | Cuando el servidor confirma un arresto |
| `nb-actions:client:toggleHandcuff` | — | El jugador local acaba de ser esposado o liberado |
| `nb-actions:client:toggleDrag` | `draggerServerId` | El jugador local empieza/termina de ser arrastrado |
| `nb-actions:client:putInVehicle` | `vehNetId` | El jugador acaba de ser metido en un vehiculo |
| `nb-actions:client:takeFromVehicle` | — | El jugador ha sido sacado de un vehiculo |

---

## Eventos de servidor (para callers programaticos)

Ejecutar acciones **sin pasar por el panel** es posible emitiendo los eventos server-side. Mismas validaciones de permiso se aplican.

| Evento | Payload |
|--------|---------|
| `nb-actions:handcuffPlayer` | `targetServerId` |
| `nb-actions:dragPlayer` | `targetServerId` |
| `nb-actions:putInVehicle` | `targetServerId, vehNetId` |
| `nb-actions:takeFromVehicle` | `targetServerId` |
| `nb-actions:checkIdentity` | `targetServerId` |
| `nb-actions:checkDriverLicense` | `targetServerId` |
| `nb-actions:checkWeaponLicense` | `targetServerId` |
| `nb-actions:checkVehicleOwner` | `plate` |
| `nb-actions:searchPlayer` | `targetServerId` |
| `nb-actions:searchDeadPlayer` | `targetServerId` |

> Estos eventos viven en `server/main.lua` y validan `PlayerCanDoAction(src, actionId)`. Si tu recurso los dispara con `TriggerServerEvent` sin que el usuario tenga permiso, el evento se descarta silenciosamente.

---

## Callback disponible

```lua
Bridge.TriggerServerCallback('nb-actions:getAllowedActions', function(actions)
    -- actions = {
    --     handcuff = true,
    --     drag = true,
    --     checkIdentity = false,
    --     ...
    -- }
end)
```

Util para cachear permisos en un HUD / menu personalizado y no abrir el UI de nb-actions.

---

## Ejemplo: custom dispatch

```lua
-- Cuando un jugador es esposado, enviar alerta a /alerts
AddEventHandler('nb-actions:client:arrestConfirmed', function(targetServerId)
    local coords = GetEntityCoords(PlayerPedId())
    TriggerServerEvent('cd_dispatch:AddNotification', {
        job_table = { 'police' },
        coords    = coords,
        title     = 'Arresto realizado',
        message   = ('Target #%d arrestado'):format(targetServerId),
    })
end)
```
