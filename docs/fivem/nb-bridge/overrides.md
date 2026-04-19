# Overrides

Los overrides permiten **reemplazar cualquier funcion de Bridge** sin modificar los archivos base del recurso. Utiles cuando tu servidor usa un sistema propietario (notificaciones custom, inventario custom, sistema de gangs separado, etc.) o para adaptar el comportamiento sin perder los updates oficiales.

---

## Como funciona

nb-bridge carga los modulos primero y despues todos los archivos en `overrides/`:

```lua
-- fxmanifest.lua de nb-bridge
client_scripts {
    'modules/framework/client.lua',
    'modules/inventory/client.lua',
    'modules/progress/client.lua',
    'overrides/client/*.lua',    -- <- se cargan al final
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'modules/framework/server.lua',
    'modules/inventory/server.lua',
    'modules/licenses/server.lua',
    'overrides/server/*.lua',    -- <- se cargan al final
}
```

Como el `Bridge` global ya existe cuando tu override corre, **puedes reasignar cualquier funcion** simplemente sobreescribiendola.

---

## Donde colocar los overrides

| Tipo | Carpeta |
|------|---------|
| Overrides de client | `overrides/client/` |
| Overrides de server | `overrides/server/` |

Nombre del archivo libre — se cargan todos con glob `*.lua`.

---

## Ejemplo 1: Notificaciones custom

Si usas `mythic_notify`, `okokNotify` u otro sistema propietario:

`overrides/client/custom_notify.lua`:

```lua
function Bridge.ShowNotification(message, type)
    exports['mythic_notify']:DoHudText(type or 'info', message)
end
```

`overrides/server/custom_notify.lua`:

```lua
function Bridge.Notify(source, message, type)
    TriggerClientEvent('nb-bridge:client:notify', source, message, type)
end
```

Todos los recursos `nb-*` empezaran a usar automaticamente `mythic_notify` sin tocar su codigo.

---

## Ejemplo 2: Inventario custom

`overrides/server/custom_inventory.lua`:

```lua
function Bridge.AddItem(source, item, count, metadata, slot)
    return exports['my_inventory']:AddItem(source, item, count, metadata)
end

function Bridge.RemoveItem(source, item, count, metadata, slot)
    return exports['my_inventory']:RemoveItem(source, item, count)
end

function Bridge.HasItem(source, item, count)
    return exports['my_inventory']:HasItem(source, item, count or 1)
end
```

---

## Ejemplo 3: Sistema de gangs alternativo

Si usas `origen_ilegal` (o similar) para gangs y quieres que nb-crafting los detecte en el nivel de "trabajo":

`overrides/server/gang_system.lua`:

```lua
local originalGetJob = Bridge.GetJob

function Bridge.GetJob(source)
    local gangData = exports['origen_ilegal']:getPlayerGang(source)
    if gangData and gangData.id then
        return {
            name         = tostring(gangData.id),
            label        = gangData.name or 'Unknown',
            grade        = gangData.level or 0,
            grade_name   = tostring(gangData.level or 0),
            grade_label  = 'Level ' .. (gangData.level or 0),
        }
    end
    return originalGetJob(source)
end
```

Guarda la funcion original antes de sustituirla — de esta forma tu override queda como un **decorator** que añade logica encima y cae al original si la condicion no aplica.

---

## Ejemplo 4: Bloquear comandos peligrosos

`overrides/server/safe_set_money.lua`:

```lua
local MAX_AMOUNT = 1000000
local originalSetMoney = Bridge.SetMoney

function Bridge.SetMoney(source, moneyType, amount, reason)
    if amount > MAX_AMOUNT then
        print(('[nb-bridge override] blocked SetMoney %s $%d (limite $%d)'):format(source, amount, MAX_AMOUNT))
        return false
    end
    return originalSetMoney(source, moneyType, amount, reason)
end
```

---

## Buenas practicas

1. **Guarda la original** si quieres una cadena: `local originalX = Bridge.X`.
2. **Un archivo por proposito** — mas facil de mantener que un `_overrides.lua` gigante.
3. **Respeta la firma** — devuelve los mismos tipos que la funcion original para no romper consumidores.
4. **Documenta en comentarios** que override estas aplicando y por que — evita sorpresas cuando actualices nb-bridge.
5. **Evita mezclar client + server** en el mismo archivo — nb-bridge los carga en carpetas separadas.
6. **Revisa el changelog** de nb-bridge al actualizar — si una funcion cambia de firma, tu override necesita ajuste.

---

## Advertencias

- Los overrides se cargan **una sola vez** al arrancar nb-bridge. Si un recurso `nb-*` arranca antes que tu override esta resuelto, no hay problema — las llamadas `Bridge.X` resuelven dinamicamente.
- No toques los archivos de `modules/*.lua` directamente. Perderas los cambios al actualizar.
- Si necesitas una funcion **nueva** (no existente en nb-bridge), no hace falta override — puedes extender `Bridge` desde tu propio recurso:

```lua
-- En tu recurso, no en nb-bridge
function Bridge.MyCustomHelper(...)
    -- ...
end
```

Eso sigue siendo una adicion al `Bridge` global y evita acoplar tu logica al repositorio de nb-bridge.
