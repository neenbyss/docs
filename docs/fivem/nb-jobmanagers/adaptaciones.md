# Adaptar scripts externos

nb-jobmanagers esta diseñado para que puedas reemplazar sus sistemas internos (garaje, facturacion) por scripts externos de terceros sin tocar el codigo principal. Todo se hace desde los archivos **bridge**, que estan abiertos (no encriptados).

---

## Arquitectura de bridges

Cada sistema tiene su propio archivo bridge que actua como punto de conexion:

| Sistema | Archivo | Que hace |
|---------|---------|----------|
| Garaje | `bridge/garage.lua` | Sacar, guardar y listar vehiculos |
| Facturacion | `bridge/invoices.lua` | Crear, pagar, rechazar y cancelar facturas |
| Base de datos | `bridge/database.lua` | Queries a tablas del framework (jobs, grades) |
| Tablas propias | `bridge/database_custom.lua` | Queries a tablas nb_* (markers, dinero, vehiculos) |

Los archivos `client/`, `server/` y la UI nunca necesitan modificarse — solo cambias el bridge y el sistema externo se conecta.

---

## Garaje externo

Si usas un script de garaje externo (qs-advancedgarages, vms_garagesv2, cd_garage, etc.) y no quieres usar nb-garages ni el garaje interno, solo modificas `bridge/garage.lua`.

### Que funciones cambiar

Todas estan en la seccion client-side del archivo (dentro del bloque `else`):

| Funcion | Cuando se ejecuta |
|---------|-------------------|
| `Bridge.OpenGarage(garageId, garageType, jobName, coords)` | Jugador presiona E en un marker de garaje **a pie** |
| `Bridge.StoreVehicle(garageId, garageType, jobName)` | Jugador presiona E en un marker de garaje **en vehiculo** |

### Ejemplo: qs-advancedgarages

```lua
-- bridge/garage.lua (seccion client, dentro del bloque else)

function Bridge.OpenGarage(garageId, garageType, jobName, coords)
    if garageType == 'society' then
        -- Vehiculos de sociedad: usar sistema interno
        TriggerServerEvent('nb-jobmanager:server:garage:open', garageId, garageType, jobName, coords)
    else
        -- Vehiculos personales: delegar a qs-advancedgarages
        exports['qs-advancedgarages']:OpenGarageMenu(garageId)
    end
end

function Bridge.StoreVehicle(garageId, garageType, jobName)
    if garageType == 'society' then
        -- Guardar vehiculo de sociedad internamente
        local ped = PlayerPedId()
        local vehicle = GetVehiclePedIsIn(ped, false)
        if vehicle == 0 then
            vehicle = GetClosestVehicle(GetEntityCoords(ped), 8.0, 0, 71)
        end
        if vehicle == 0 then
            Bridge.ShowNotification(Locale('garage_no_vehicle'), 'error')
            return
        end
        local plate = Bridge.NormalizePlate(GetVehicleNumberPlateText(vehicle))
        local model = GetEntityModel(vehicle)
        local props = Bridge.GetVehicleProperties(vehicle)
        local netId = NetworkGetNetworkIdFromEntity(vehicle)
        TriggerServerEvent('nb-jobmanager:server:garage:store', garageId, garageType, jobName, netId, plate, model, props)
    else
        -- Delegar a qs-advancedgarages
        exports['qs-advancedgarages']:ImpoundVehicle()
    end
end
```

Este patron hibrido usa el garaje externo para vehiculos personales y el sistema interno para vehiculos de sociedad.

### Ejemplo: delegar todo al externo

Si quieres que el script externo maneje **todo** (incluyendo sociedad):

```lua
function Bridge.OpenGarage(garageId, garageType, jobName, coords)
    exports['qs-advancedgarages']:OpenGarageMenu(garageId)
end

function Bridge.StoreVehicle(garageId, garageType, jobName)
    exports['qs-advancedgarages']:ImpoundVehicle()
end
```

### Ejemplo: vms_garagesv2

```lua
function Bridge.OpenGarage(garageId, garageType, jobName, coords)
    exports['vms_garagesv2']:openTowMenu()
end

function Bridge.StoreVehicle(garageId, garageType, jobName)
    local ped = PlayerPedId()
    local vehicle = GetVehiclePedIsIn(ped, false)
    if vehicle == 0 then return end
    local plate = GetVehicleNumberPlateText(vehicle)
    local netId = NetworkGetNetworkIdFromEntity(vehicle)
    TriggerServerEvent('nb-jobmanager:server:garage:store:vms', netId, plate, garageId)
end
```

En el servidor, agrega el handler al final de `server/garage.lua`:

```lua
RegisterNetEvent('nb-jobmanager:server:garage:store:vms', function(netId, plate, garageId)
    local src = source
    exports['vms_garagesv2']:towVehicle(netId, plate, 'Impound1', nil, nil, GetPlayerName(src))
end)
```

### Ejemplo: script de garaje personalizado

```lua
function Bridge.OpenGarage(garageId, garageType, jobName, coords)
    exports['mygarage']:open(garageId, {
        job = jobName,
        type = garageType,
        coords = coords,
    })
end

function Bridge.StoreVehicle(garageId, garageType, jobName)
    local ped = PlayerPedId()
    local vehicle = GetVehiclePedIsIn(ped, false)
    if vehicle == 0 then
        Bridge.ShowNotification(Locale('garage_no_vehicle'), 'error')
        return
    end
    exports['mygarage']:store(vehicle, garageId)
end
```

---

## Facturacion externa

Si no quieres usar nb-billings ni el sistema interno de facturas, tienes dos opciones.

### Opcion 1: UseExternal (config)

La forma mas simple. En `shared/config.lua`:

```lua
Config.Billing = {
    Enabled = true,
    UseExternal = true,
    -- El resto de opciones...
}
```

Con `UseExternal = true`, al crear una factura se usa el sistema de billing del framework (ESX billing / QBCore billing) en lugar del sistema interno.

### Opcion 2: Modificar bridge/invoices.lua

Para integrar un script de facturacion especifico (okokBilling, esx_billing custom, etc.), modifica las funciones en `bridge/invoices.lua`:

```lua
-- bridge/invoices.lua

-- Reemplazar la creacion de facturas
function Bridge.DB.CreateInvoice(data)
    -- En lugar de insertar en nb_invoices, delegar al script externo
    return exports['okokBilling']:CreateInvoice(
        data.target_identifier,  -- quien paga
        data.job_name,           -- sociedad que cobra
        data.amount,             -- monto
        data.description         -- concepto
    )
end

-- Reemplazar la consulta de facturas del jugador
function Bridge.DB.GetPlayerInvoices(identifier, status)
    return exports['okokBilling']:GetPlayerInvoices(identifier) or {}
end

-- Reemplazar el pago
function Bridge.DB.UpdateInvoiceStatus(invoiceId, status)
    if status == 'paid' then
        exports['okokBilling']:PayInvoice(invoiceId)
    elseif status == 'cancelled' then
        exports['okokBilling']:CancelInvoice(invoiceId)
    end
    return true
end
```

### Ejemplo: esx_billing

```lua
-- bridge/invoices.lua

function Bridge.DB.CreateInvoice(data)
    -- esx_billing usa su propia tabla y UI
    MySQL.insert.await(
        'INSERT INTO billing (identifier, sender, target_type, target, label, amount) VALUES (?, ?, ?, ?, ?, ?)',
        {
            data.emitter_identifier,
            data.job_name,
            'player',
            data.target_identifier,
            data.description or 'Factura',
            data.amount,
        }
    )
end
```

---

## Dinero de sociedad externo

Si quieres que el dinero de sociedad se gestione con otro script (esx_addonaccount, qb-banking, etc.) en lugar de la tabla `nb_society_money`, modifica las funciones en `bridge/database_custom.lua`:

### Ejemplo: esx_addonaccount

```lua
-- bridge/database_custom.lua

-- Reemplazar las funciones de dinero de sociedad

function Bridge.DB.GetSocietyMoney(jobName)
    local account = exports['esx_addonaccount']:GetSharedAccount('society_' .. jobName)
    return account and account.money or 0
end

function Bridge.DB.AddSocietyMoney(jobName, amount)
    local account = exports['esx_addonaccount']:GetSharedAccount('society_' .. jobName)
    if account then
        account.addMoney(amount)
    end
    return Bridge.DB.GetSocietyMoney(jobName)
end

function Bridge.DB.RemoveSocietyMoney(jobName, amount)
    local account = exports['esx_addonaccount']:GetSharedAccount('society_' .. jobName)
    if account then
        account.removeMoney(amount)
    end
    return Bridge.DB.GetSocietyMoney(jobName)
end

function Bridge.DB.SetSocietyMoney(jobName, amount)
    local account = exports['esx_addonaccount']:GetSharedAccount('society_' .. jobName)
    if account then
        account.setMoney(amount)
    end
    return true
end
```

Con esto, el boss menu, las facturas y los exports siguen funcionando igual — pero el dinero se lee y escribe en esx_addonaccount en vez de `nb_society_money`.

### Ejemplo: qb-banking

```lua
-- bridge/database_custom.lua

function Bridge.DB.GetSocietyMoney(jobName)
    return exports['qb-banking']:GetAccountBalance(jobName) or 0
end

function Bridge.DB.AddSocietyMoney(jobName, amount)
    exports['qb-banking']:AddMoney(jobName, amount, 'Society deposit')
    return Bridge.DB.GetSocietyMoney(jobName)
end

function Bridge.DB.RemoveSocietyMoney(jobName, amount)
    exports['qb-banking']:RemoveMoney(jobName, amount, 'Society withdrawal')
    return Bridge.DB.GetSocietyMoney(jobName)
end
```

---

## Resumen: que archivo modificar

| Quiero cambiar... | Archivo a modificar |
|--------------------|---------------------|
| Garaje (sacar/guardar vehiculos) | `bridge/garage.lua` |
| Facturacion (crear/pagar facturas) | `bridge/invoices.lua` |
| Dinero de sociedad (depositar/retirar) | `bridge/database_custom.lua` |
| Queries de jobs/grades del framework | `bridge/database.lua` |

Todos estos archivos estan en `escrow_ignore` — son **abiertos** y editables aunque uses la version encriptada del recurso.

---

## Notas importantes

- Al modificar un bridge, **no** necesitas tocar `server/`, `client/` ni la UI. Todo pasa por las funciones `Bridge.*`.
- Si el script externo usa un formato de placa diferente, usa `Bridge.NormalizePlate(plate)` o `TRIM(plate)` en tus queries SQL para evitar problemas con los espacios que GTA agrega a las placas.
- Activa `Config.Debug = true` para ver logs detallados de cada operacion y verificar que la integracion funciona correctamente.
- Si usas esx_addonaccount para el dinero, asegurate de que la cuenta `society_{jobName}` exista en la tabla `addon_account`. El recurso **no** la crea automaticamente.
