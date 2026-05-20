# Recompensas

Las recompensas son objetos JSON guardados en `nb_battlepass_tiers.free_reward` / `premium_reward`. Cada recompensa tiene un **tipo** que despacha a un handler en `shared/rewards.lua`.

---

## Estructura de una recompensa

```lua
{
    type   = 'item',         -- requerido (item, money, vehicle, custom...)
    label  = 'Pan x5',       -- texto mostrado al jugador en la UI
    icon   = 'icon-utensils' -- (opcional) clase Lucide
    -- ... campos especificos del tipo
}
```

El campo `type` se usa como key para buscar el handler en `Config.RewardHandlers`.

---

## Tipos builtin

### `item` — Items de inventario

Da items via `Bridge.Inventory.AddItem`. Soporta ox_inventory, qb-inventory, esx_inventory, etc. (lo que detecte nb-bridge).

```lua
{
    type   = 'item',
    name   = 'bread',        -- nombre del item en tu DB/inventario
    amount = 5,
    label  = '5x Pan',
    icon   = 'icon-utensils',
    metadata = nil           -- (opcional) metadata para ox_inventory
}
```

En la UI admin: selecciona **Type = item**, llena **Item name**, **Amount** y **Label**.

---

### `money` — Dinero del framework

Da dinero via `Bridge.Money.Add`.

```lua
{
    type    = 'money',
    account = 'bank',        -- 'cash', 'bank' o 'black_money'
    amount  = 5000,
    label   = '$5,000',
    icon    = 'icon-banknote'
}
```

---

### `vehicle` — Vehiculo

Lo agrega a la tabla de vehiculos del jugador (via `Bridge.Vehicles.Give`). Se genera una matricula `BPXXXXX`.

```lua
{
    type  = 'vehicle',
    model = 'adder',
    label = 'Adder',
    icon  = 'icon-car',
    props = nil              -- (opcional) tabla de modificaciones
}
```

---

### `custom` — Tu propia logica

Cualquier `type` que no sea uno de los anteriores se considera custom. Debes registrar un handler para ese tipo en `shared/rewards.lua`. Ver [Personalizacion](personalizacion.md).

```lua
{
    type     = 'vip_badge',
    duration = 7,
    label    = 'VIP 7 dias',
    icon     = 'icon-crown'
}
```

---

## Flujo interno de un claim

```
Jugador clickea recompensa en UI
    ↓
RegisterNUICallback('claim') → TriggerServerEvent('claim')
    ↓
[server/main.lua] Valida:
    - tier existe en la season activa
    - jugador alcanzo el nivel del tier
    - si es premium track, jugador tiene premium
    - no esta ya reclamada
    ↓
Llama Config.RewardHandlers[reward.type](src, reward)
    ↓
Handler devuelve (success, errMsg?)
    ↓
Si success: registra claim en nb_battlepass_claims + notifica
Si no:      muestra errMsg al jugador
```

!!! warning "Los handlers corren en el server"
    Nunca llames a un handler desde el cliente. El server valida primero y luego invoca al handler.

---

## Reglas del handler

Toda funcion en `Config.RewardHandlers[type]` debe:

1. Recibir `(src, reward)` donde `reward` es la tabla JSON completa.
2. Devolver `true` en exito.
3. Devolver `false, 'mensaje_de_error'` en fallo (el mensaje se muestra al jugador).
4. Correr **solo en el server** (`shared/rewards.lua` hace `if not IsDuplicityVersion() then return end`).

Ejemplo minimo:

```lua
Config.RewardHandlers['my_type'] = function(src, reward)
    if not reward.something then
        return false, 'invalid_data'
    end
    -- haz lo que sea
    return true
end
```

---

## ¿Que pasa si un tipo no tiene handler?

Si el admin configura un tier con `type = 'foo'` y no existe `Config.RewardHandlers['foo']`, al intentar reclamar:

- El server loguea (con Debugger): `No handler for type foo`
- El jugador recibe `server_error`

→ **Siempre registra tu handler ANTES de usar el tipo en un tier.**
