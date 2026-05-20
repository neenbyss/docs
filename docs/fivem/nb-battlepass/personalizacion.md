# Personalizacion - Recompensas Custom

Esta es la parte mas poderosa de nb-battlepass: puedes añadir **cualquier tipo de recompensa** sin tocar la logica encriptada del recurso.

Todo se hace en **`shared/rewards.lua`** (archivo publico).

---

## Como funciona

1. Abres `shared/rewards.lua`.
2. Defines una funcion en `Config.RewardHandlers[tu_tipo]`.
3. Desde el panel admin, eliges **Type = custom** (o el nombre que registraste) y le pones los datos necesarios.
4. Cuando el jugador reclame, el server llama a tu funcion.

---

## Ejemplo 1 — Dar coins de nb-vip

**`shared/rewards.lua`:**

```lua
Config.RewardHandlers['coins'] = function(src, reward)
    local amount = tonumber(reward.amount) or 0
    if amount <= 0 then return false, 'invalid_data' end

    local ok = exports['nb-vip']:AddPlayerCoins(src, amount, 'battlepass_reward')
    if not ok then return false, 'server_error' end
    return true
end
```

**En el panel admin (Tier editor):**

| Campo | Valor |
|---|---|
| Type | `custom` |
| Custom ID | `coins` |
| Amount | `500` |
| Label | `500 Coins` |
| Icon | `icon-coins` |

!!! tip "Tip"
    El campo **Custom ID** del editor se guarda como `reward.id` en el JSON, pero el dispatcher usa `reward.type`. Si quieres que el dispatcher seleccione `coins`, ten en cuenta que el editor builtin envia `type = 'custom'`. Para registrar un tipo **propio** (no "custom"), edita el JSON directo en la DB o **añade el tipo al `<select>` del editor** (ver mas abajo).

### Hacer que aparezca en el dropdown del admin

Edita `ui/js/components/bp-admin.js`, busca el `<select v-model="reward.type">` y añade tu opcion:

```html
<select v-model="reward.type" class="form-select">
    <option value="item">item</option>
    <option value="money">money</option>
    <option value="vehicle">vehicle</option>
    <option value="coins">coins (nb-vip)</option>
    <option value="custom">custom</option>
</select>
```

Ahora el admin puede elegir directamente `coins` desde el dropdown.

---

## Ejemplo 2 — Otorgar VIP temporal

```lua
Config.RewardHandlers['vip_temp'] = function(src, reward)
    local days = tonumber(reward.days) or 7
    exports['nb-vip']:GrantVIP(src, days)
    Bridge.Notify(src, ('VIP %d dias activado'):format(days), 'success')
    return true
end
```

JSON en la DB:
```json
{ "type": "vip_temp", "days": 30, "label": "VIP 30 dias", "icon": "icon-crown" }
```

---

## Ejemplo 3 — Llave de casa (housing system)

```lua
Config.RewardHandlers['house_key'] = function(src, reward)
    if not reward.house_id then return false, 'invalid_data' end

    local identifier = Bridge.GetIdentifier(src)
    MySQL.insert.await(
        'INSERT INTO house_keys (house_id, identifier) VALUES (?, ?)',
        { reward.house_id, identifier }
    )
    return true
end
```

---

## Ejemplo 4 — Desbloquear job (whitelist)

```lua
Config.RewardHandlers['job_unlock'] = function(src, reward)
    if not reward.job then return false, 'invalid_data' end
    local identifier = Bridge.GetIdentifier(src)
    MySQL.insert.await(
        'INSERT IGNORE INTO job_whitelist (identifier, job) VALUES (?, ?)',
        { identifier, reward.job }
    )
    Bridge.Notify(src, ('Acceso al job %s desbloqueado'):format(reward.job), 'success')
    return true
end
```

---

## Ejemplo 5 — Boost de XP en otro sistema

```lua
Config.RewardHandlers['xp_boost'] = function(src, reward)
    local multiplier = tonumber(reward.multiplier) or 2
    local duration   = tonumber(reward.duration) or 3600  -- segundos
    exports['nb-skills']:ApplyXPBoost(src, multiplier, duration)
    return true
end
```

---

## Ejemplo 6 — Recompensa con multiples efectos

A veces quieres dar **varias cosas a la vez** (ej. nivel 50 da: vehiculo + dinero + un titulo). Define un handler "bundle":

```lua
Config.RewardHandlers['bundle'] = function(src, reward)
    for _, sub in ipairs(reward.items or {}) do
        local handler = Config.RewardHandlers[sub.type]
        if handler then
            local ok, err = handler(src, sub)
            if not ok then return false, err end
        end
    end
    return true
end
```

JSON:
```json
{
    "type": "bundle",
    "label": "Mega Recompensa",
    "icon": "icon-gift",
    "items": [
        { "type": "money", "account": "bank", "amount": 50000 },
        { "type": "vehicle", "model": "adder" },
        { "type": "item", "name": "diamond", "amount": 5 }
    ]
}
```

---

## Modificar el comportamiento del premium pass

La compra del premium tambien es un hook publico. Ver [Configuracion → Premium Pass](configuracion.md#premium-pass) para ejemplos de integracion con sistemas externos de monedas.

---

## Que NO puedes modificar (esta encriptado)

- Como se calcula el nivel (lineal: `floor(xp / xp_per_level)`)
- Como se valida un claim (nivel alcanzado + no reclamado + premium check)
- El registro en `nb_battlepass_claims`
- El flujo del NUI callback

Si necesitas cambiar la **formula de niveles** (ej. curva exponencial), abre un issue/ticket en Tebex y considera contratar custom development.

---

## Consejo final

> **Manten tu logica custom en `shared/rewards.lua` y nada mas.**

Asi cuando actualices nb-battlepass, no pierdes nada: ese archivo esta en `escrow_ignore` y NO se sobrescribe.
