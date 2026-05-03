# Dispatch / 911

Sistema de llamadas medicas con estado, panel en vivo y broadcast a todos los medicos en servicio. Reemplaza el viejo "fan-out de un solo tiro" — ahora cada llamada tiene id, lifecycle y se mantiene en sync entre todos los medicos conectados.

---

## Como se crean llamadas

Tres caminos:

| Origen | Como |
|--------|------|
| `[G]` en laststand | El jugador downed pulsa G mientras esta en writhe loop. Una sola llamada por sesion de laststand. |
| `/911` | Cualquier jugador, en cualquier estado, desde cualquier sitio. Util para civiles testigos. |
| `exports['nb-ambulance']:broadcastDistress(src, coords?)` | Otros scripts (telefonos, /me ICs, etc.) crean llamadas en nombre de cualquier jugador. |

El server crea la call:

```lua
{
    id           = 1,
    callerSrc    = 12,
    callerName   = 'John Doe',
    coords       = { x = ..., y = ..., z = ... },
    status       = 'pending',
    assignedTo   = nil,
    assignedName = nil,
    createdAt    = 1714699200,
    updatedAt    = 1714699200,
}
```

---

## Lifecycle

```
pending  ─→ acked  ─→ completed (eliminada)
   │
   └────────→ expired (15 min sin actividad, eliminada)
```

| Status | Significado |
|--------|-------------|
| `pending` | Recien creada. Visible para todos los medicos. |
| `acked` | Un medico hizo click "Tomar". Caller recibe toast `Medico X esta en camino`. |
| `completed` | Medico cerro la llamada. Caller recibe toast `Llamada cerrada por X`. La call se borra. |

Auto-expira despues de `Config.Dispatch.ExpireMs` (15 min default) sin updates.

Si el medico assignedTo se desconecta, la call vuelve a `pending` automaticamente. Si el caller se desconecta, sus calls se borran.

---

## Panel del medico

```
/dispatch       (alias directo)
F6              (keybind por defecto)
```

Solo abre para medicos (`Config.MedicalJobs`) o admins.

### Cards

Cada llamada se muestra como una card con:

- **Nombre del caller** + id de la call + timestamp HH:MM.
- **Badge de status**: pendiente (warning), en camino (primary), completada.
- **Si ya esta acked y eres tu**: la card se highlightea en rojo claro (mine).
- **Acciones**:
  - `GPS` — `SetNewWaypoint` a las coords del caller.
  - `Tomar` — solo en pending. Pasa a acked y te asigna.
  - `Cerrar` — solo en acked. Marca completed y borra.

### Live updates

El server hace broadcast de `:client:dispatchUpdate` con la lista completa cada vez que:

- Una call nueva se crea.
- Un medico hace ack o complete.
- Un medico assignedTo se desconecta (call vuelve a pending).
- Una call expira por timeout.

El panel re-renderiza al recibir el update — no hay polling.

---

## Caller-side

Al pulsar G o ejecutar /911:

1. Toast: `Llamada de auxilio enviada. Aguanta.`
2. Si **no hay medicos en servicio**: toast `No hay medicos en servicio - llamada no entregada.` y la call NO se crea.
3. Si la llamada se crea: blip rojo + GPS waypoint en el HUD de cada medico.
4. Cuando un medico hace ack: toast al caller `El medico X esta en camino.`
5. Cuando se cierra: toast `Llamada cerrada por X.`

---

## Compatibilidad con scripts externos

### Otros sistemas de 911 (telefonos, etc.)

Para que tu telefono / app dispare a nb-ambulance sin que tengas que migrar:

```lua
-- desde tu script (server-side)
exports['nb-ambulance']:broadcastDistress(src, customCoords)
```

`customCoords` es opcional — sin el, usa la posicion actual del player.

### Listener de llamadas

Si quieres registrar las llamadas en tu propio panel admin:

```lua
RegisterNetEvent('nb-ambulance:client:distressInbound', function(payload)
    -- payload.callerSrc, callerName, coords
end)
```

El cliente recibe este evento solo si esta en un job medico (lo filtramos server-side).

---

## Anti-spam (todavia no implementado)

`/911` no tiene cooldown actualmente. Si tu server lo necesita:

```lua
-- en tu propio script
local lastCall = {}
AddEventHandler('chatMessage', function(src, _, msg)
    if msg:sub(1, 4) == '/911' then
        local now = os.time()
        if lastCall[src] and now - lastCall[src] < 30 then
            CancelEvent()
            return
        end
        lastCall[src] = now
    end
end)
```

(En una proxima version se añadira `Config.Dispatch.Cooldown` integrado.)
