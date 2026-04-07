# Compatibilidad

nb-jobmanagers se integra con otros scripts del ecosistema Neenbyss y puede ser usado desde cualquier script externo mediante [exports](exports.md).

---

## Scripts del ecosistema Neenbyss

### nb-bridge (requerido)

nb-jobmanagers usa **nb-bridge** para funcionar en ESX y QBCore sin cambiar codigo. Es una dependencia obligatoria. Solo necesitas agregarlo en `dependencies` de tu `fxmanifest.lua` — la tabla global `Bridge.*` se carga automaticamente sin importar archivos.

```cfg
ensure oxmysql
ensure es_extended   # o qb-core
ensure nb-bridge
ensure nb-jobmanagers
```

---

### nb-billings (opcional)

Si **nb-billings** esta instalado, nb-jobmanagers delega todo el sistema de facturacion a el: crear, pagar, rechazar, cancelar facturas y la UI de facturas del jugador.

| Config | Comportamiento |
|--------|---------------|
| `Config.Billing.UseNbBillings = 'auto'` | Detecta nb-billings automaticamente. Si esta, lo usa; si no, usa el sistema interno |
| `Config.Billing.UseNbBillings = true` | Requiere nb-billings. Si no esta, muestra error |
| `Config.Billing.UseNbBillings = false` | Siempre usa el sistema interno (tabla `nb_invoices`) |

Con `'auto'` no necesitas tocar nada: si nb-billings esta en el servidor, se usa; si lo quitas, vuelve al sistema interno.

---

### nb-garages (opcional)

Si **nb-garages** esta instalado, nb-jobmanagers delega la gestion de vehiculos de sociedad: sacar, guardar, agregar y eliminar vehiculos, ademas del sistema de llaves.

| Config | Comportamiento |
|--------|---------------|
| `Config.Garage.UseNbGarages = 'auto'` | Detecta nb-garages automaticamente |
| `Config.Garage.UseNbGarages = true` | Requiere nb-garages |
| `Config.Garage.UseNbGarages = false` | Usa el garaje interno (tabla `nb_job_garage_vehicles`) |

Cuando nb-garages esta activo, tambien gestiona las llaves de los vehiculos de sociedad al sacarlos y guardarlos.

---

## Usar nb-jobmanagers desde otros scripts

Cualquier script puede interactuar con nb-jobmanagers a traves de sus [exports](exports.md). Solo necesitas agregar la dependencia en tu `fxmanifest.lua`:

```lua
dependencies {
    'nb-jobmanagers',
}
```

---

### Ejemplo: script de mineria

Un script de trabajo "miner" donde el jugador vende mineral, cobra su parte y un porcentaje va a la sociedad.

**fxmanifest.lua**
```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

dependencies {
    'oxmysql',
    'nb-bridge',
    'nb-jobmanagers',
}

shared_scripts {
    'shared/config.lua',
}

server_scripts {
    'server/main.lua',
}

client_scripts {
    'client/main.lua',
}
```

No necesitas importar ningun archivo de nb-bridge — la tabla `Bridge.*` esta disponible globalmente en cuanto el recurso carga.

**server/main.lua**
```lua
local TAX_RATE = 0.10  -- 10% para la sociedad

RegisterNetEvent('miner:server:sellOre', function(amount)
    local src = source
    local job = exports['nb-jobmanagers']:getPlayerJob(src)

    -- Verificar que el jugador sea minero
    if not job or job.name ~= 'miner' then return end

    local price = amount * 50
    local tax = math.floor(price * TAX_RATE)
    local playerPay = price - tax

    -- Pagar al jugador
    Bridge.AddMoney(src, 'cash', playerPay, 'Venta de mineral')

    -- Agregar impuesto a la sociedad
    exports['nb-jobmanagers']:addSocietyMoney('miner', tax, 'Impuesto de venta')
end)
```

---

### Ejemplo: multa automatica desde un radar

Un script de radar que crea una factura al jugador cuando excede el limite de velocidad.

**server/radar.lua**
```lua
RegisterNetEvent('radar:server:speedTicket', function(targetSource, speed, limit)
    local src = source
    local fine = (speed - limit) * 100  -- $100 por km/h de exceso

    -- Crear factura de la policia al jugador
    exports['nb-jobmanagers']:createInvoice(
        'police',          -- trabajo emisor
        targetSource,      -- jugador multado
        fine,              -- monto
        ('Exceso de velocidad: %d/%d km/h'):format(speed, limit),
        src                -- oficial que emite
    )
end)
```

---

### Ejemplo: verificar si un jugador esta esposado

Desde el **cliente**, otros scripts pueden verificar el estado del jugador para bloquear acciones.

**client/main.lua**
```lua
-- Antes de abrir un menu o inventario, verificar que no este esposado
RegisterCommand('abrir_tienda', function()
    if exports['nb-jobmanagers']:IsPlayerHandcuffed() then
        -- No puede abrir la tienda esposado
        return
    end

    if exports['nb-jobmanagers']:IsPlayerDragged() then
        -- No puede abrir la tienda siendo arrastrado
        return
    end

    -- Abrir tienda normalmente
    abrirTienda()
end)
```

---

### Ejemplo: sistema de bonus por sociedad

Un script que reparte bonus a todos los empleados conectados cuando la sociedad acumula cierta cantidad.

**server/bonus.lua**
```lua
local BONUS_THRESHOLD = 50000  -- Repartir bonus cuando hay $50,000+
local BONUS_PER_PLAYER = 500

CreateThread(function()
    while true do
        Wait(60000 * 30)  -- Revisar cada 30 minutos

        local jobs = exports['nb-jobmanagers']:getJobs()

        for jobName, _ in pairs(jobs) do
            local balance = exports['nb-jobmanagers']:getSocietyMoney(jobName)

            if balance >= BONUS_THRESHOLD then
                local employees = exports['nb-jobmanagers']:getEmployees(jobName)
                local onlineCount = 0

                for _, emp in ipairs(employees) do
                    local src = Bridge.GetSourceFromIdentifier(emp.identifier)
                    if src then
                        Bridge.AddMoney(src, 'bank', BONUS_PER_PLAYER, 'Bonus de sociedad')
                        onlineCount = onlineCount + 1
                    end
                end

                if onlineCount > 0 then
                    local totalPaid = onlineCount * BONUS_PER_PLAYER
                    exports['nb-jobmanagers']:removeSocietyMoney(
                        jobName, totalPaid, ('Bonus a %d empleados'):format(onlineCount)
                    )
                end
            end
        end
    end
end)
```

---

## Frameworks soportados

nb-jobmanagers funciona en ambos frameworks gracias a nb-bridge:

| Framework | Version minima | Notas |
|-----------|---------------|-------|
| ESX Legacy | 1.9.0+ | Usa tablas `jobs` / `job_grades` de ESX |
| QBCore | Cualquiera | Usa tablas `jobs` / `job_grades` de QBCore |

No necesitas cambiar nada en el config segun el framework. nb-bridge lo detecta automaticamente.
