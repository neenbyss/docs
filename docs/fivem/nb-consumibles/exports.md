# Exports

Api para integrar nb-consumibles con otros recursos.

---

## Server

### GetItem

Obtiene la definicion completa de un item (incluye cadena de efectos).

```lua
local item = exports['nb-consumibles']:GetItem('bread')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `name` | string | Nombre interno del item. |

**Returns:** `table|nil` — Row con `id`, `name`, `label`, `category`, `enabled`, `effects[]`, etc. `nil` si no esta registrado.

---

### GetAllItems

Devuelve la lista de items registrados.

```lua
local items = exports['nb-consumibles']:GetAllItems()
```

**Returns:** `table[]` — Array de items (mismo shape que `GetItem`).

---

### RegisterItem

Persiste un item nuevo desde codigo sin usar el panel.

```lua
local id = exports['nb-consumibles']:RegisterItem({
    name              = 'energy_drink',
    label             = 'Energy Drink',
    category          = 'drink',
    enabled           = true,
    remove_on_use     = true,
    progress_ms       = 3500,
    progress_label_key= 'drink_progress',
    effects = {
        { effect_type = 'thirst',            params = { amount = 20, operation = 'add' },              execution_order = 1 },
        { effect_type = 'stamina_restore',   params = {},                                                execution_order = 2 },
        { effect_type = 'sprint_multiplier', params = { value = 1.15, duration = 15 },                  execution_order = 3 },
    },
})
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `data` | table | Shape identico al que envia el panel. Ver el schema en la pestana "Items" o en `bridge/database.lua:Bridge.DB.UpsertItem`. |

**Returns:** `number|nil` — id del item persistido. Tras el insert se dispara automaticamente el hot-reload.

---

### AddServerEffectHandler

Registra un handler de efecto server-side. Util para recursos que quieren anadir efectos que mutan estado del servidor (dinero, inventario custom, notificaciones a terceros...).

```lua
exports['nb-consumibles']:AddServerEffectHandler('wanted_level', function(source, params)
    local ped = GetPlayerPed(source)
    SetPlayerWantedLevel(source, tonumber(params.level) or 1, false)
    SetPlayerWantedLevelNow(source, false)
end)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `effectId` | string | Id del efecto (debe coincidir con el `id` en `EffectsCatalog`). |
| `handler` | function(source, params) | Handler a registrar. |

**Returns:** `boolean` — `true` si se registro, `false` si los parametros son invalidos.

> Si quieres que el efecto aparezca en el panel, anade su entrada en `shared/effects_catalog.lua`.

---

## Client

### AddEffectHandler

Registra un handler de efecto client-side.

```lua
exports['nb-consumibles']:AddEffectHandler('gravity_low', function(params)
    SetGravityLevel(1)              -- low gravity preset
    CreateThread(function()
        Wait((tonumber(params.duration) or 20) * 1000)
        SetGravityLevel(0)          -- back to default
    end)
end)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `effectId` | string | Id del efecto. |
| `handler` | function(params) | Recibe los params del efecto. |

**Returns:** `boolean`.

---

## Ejemplos de integracion

### Recompensa de mision -> consumible temporal

```lua
-- En tu recurso de misiones, al completar una mision:
exports['nb-consumibles']:RegisterItem({
    name = 'mission_loot_box_' .. missionId,
    label = 'Mission Reward',
    category = 'custom',
    progress_ms = 2000,
    effects = {
        { effect_type = 'money_reward', params = { account = 'cash', min = 500, max = 1500 } },
        { effect_type = 'give_item',    params = { item = 'lockpick', count = 3 } },
    },
})
exports['ox_inventory']:AddItem(source, 'mission_loot_box_' .. missionId, 1)
```

### Efecto custom ligado a un sistema de habilidades

```lua
-- client
exports['nb-consumibles']:AddEffectHandler('skill_xp_boost', function(p)
    local duration = (tonumber(p.duration) or 300) * 1000
    TriggerEvent('my-skills:xpMultiplier', tonumber(p.mult) or 2.0, duration)
end)
```

Luego desde el panel creas un item (`skill_potion`) con un efecto `skill_xp_boost` de params `{ mult = 2.0, duration = 600 }`.

---

## Eventos publicos

| Evento | Direccion | Descripcion |
|--------|-----------|-------------|
| `nb-consumibles:server:reload` | server -> server | Fuerza una relectura completa de la BD y re-propagacion. |
| `nb-consumibles:client:updateNeed` | server -> client | Avisa al HUD que `hunger` o `thirst` cambiaron. |
| `nb-consumibles:client:runEffects` | server -> client | Dispatch interno de efectos client. |
| `nb-consumibles:server:consumeFinished` | client -> server | El cliente reporta que la progressbar termino sin cancelar. |

> Los eventos marcados como "Dispatch interno" estan expuestos pero no se recomienda invocarlos directamente desde otro recurso — usa los exports.
