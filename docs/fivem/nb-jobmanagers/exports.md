# Exports

Otros recursos pueden usar **nb-jobmanagers** para gestionar trabajos, dinero de sociedad, empleados, facturas, vehiculos y mas.

---

## Server-side

Todos los exports de esta seccion se usan desde scripts del **servidor**.

---

### Trabajos

#### getJobs

Devuelve todos los trabajos con sus grados.

```lua
local jobs = exports['nb-jobmanagers']:getJobs()
-- jobs = { police = { name = 'police', label = 'Policia', grades = { ... } }, ... }
```

**Returns:** `table` — Tabla indexada por nombre de trabajo.

---

#### getJob

Devuelve un trabajo especifico.

```lua
local job = exports['nb-jobmanagers']:getJob('police')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |

**Returns:** `table` o `nil`

---

#### doesJobExist

Comprueba si un trabajo existe.

```lua
if exports['nb-jobmanagers']:doesJobExist('police') then
    print('Existe')
end
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |

**Returns:** `boolean`

---

#### getJobGrades

Devuelve los grados de un trabajo.

```lua
local grades = exports['nb-jobmanagers']:getJobGrades('police')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |

**Returns:** `table` — Tabla de grados.

---

#### createJob

Crea un trabajo programaticamente.

```lua
local success = exports['nb-jobmanagers']:createJob({
    name = 'miner',
    label = 'Minero',
    type = 'civ',
    actions = 0,
})
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `data` | table | `{ name, label, type, actions }` |

**Returns:** `boolean`

---

#### updateJob

Actualiza un trabajo existente.

```lua
local success = exports['nb-jobmanagers']:updateJob({
    name = 'miner',
    label = 'Minero Pro',
    type = 'civ',
    actions = 0,
})
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `data` | table | `{ name, label, type, actions }` |

**Returns:** `boolean`

---

#### deleteJob

Elimina un trabajo y todo lo asociado (grados, markers, dinero, vehiculos).

```lua
local success = exports['nb-jobmanagers']:deleteJob('miner')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |

**Returns:** `boolean`

---

### Markers

#### getMarkers

Devuelve todos los markers de todos los trabajos.

```lua
local markers = exports['nb-jobmanagers']:getMarkers()
```

**Returns:** `table` — Lista de markers.

---

#### getJobMarkers

Devuelve los markers de un trabajo especifico.

```lua
local markers = exports['nb-jobmanagers']:getJobMarkers('police')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |

**Returns:** `table` — Lista de markers del trabajo.

---

### Jugadores

#### getPlayerJob

Devuelve el trabajo actual de un jugador.

```lua
local job = exports['nb-jobmanagers']:getPlayerJob(source)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |

**Returns:** `table` — Datos del trabajo del jugador.

---

#### setPlayerJob

Asigna un trabajo y grado a un jugador.

```lua
local success = exports['nb-jobmanagers']:setPlayerJob(source, 'police', 2)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |
| `jobName` | string | Nombre del trabajo |
| `grade` | number | *(opcional)* Numero de grado, por defecto `0` |

**Returns:** `boolean`

---

#### giveVehicle

Spawnea un vehiculo para un jugador.

```lua
local vehicle = exports['nb-jobmanagers']:giveVehicle(source, 'police2', props)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |
| `model` | string | Modelo del vehiculo |
| `props` | table | *(opcional)* Propiedades del vehiculo |

**Returns:** `table` — Datos del vehiculo.

---

### Dinero de sociedad

#### getSocietyMoney

Devuelve el saldo de la cuenta de sociedad de un trabajo.

```lua
local balance = exports['nb-jobmanagers']:getSocietyMoney('police')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |

**Returns:** `number` — Saldo actual. Devuelve `0` si el trabajo no existe o no tiene cuenta.

---

#### addSocietyMoney

Agrega dinero a la cuenta de sociedad. Si se indica `reason`, la transaccion queda registrada en el historial con emisor "System".

```lua
local newBalance = exports['nb-jobmanagers']:addSocietyMoney('police', 5000, 'Pago por servicio')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |
| `amount` | number | Monto a agregar (debe ser > 0) |
| `reason` | string | *(opcional)* Motivo para el historial de transacciones |

**Returns:** `number` — Nuevo saldo. Devuelve `0` si los parametros son invalidos.

---

#### removeSocietyMoney

Retira dinero de la cuenta de sociedad. Si la sociedad no tiene saldo suficiente, no retira nada y devuelve el saldo actual.

```lua
local newBalance = exports['nb-jobmanagers']:removeSocietyMoney('police', 2000, 'Compra de equipo')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |
| `amount` | number | Monto a retirar (debe ser > 0) |
| `reason` | string | *(opcional)* Motivo para el historial de transacciones |

**Returns:** `number` — Nuevo saldo. Si no habia saldo suficiente, devuelve el saldo sin modificar.

---

### Empleados

#### getEmployees

Devuelve la lista de empleados de un trabajo.

```lua
local employees = exports['nb-jobmanagers']:getEmployees('police')
-- employees = { { identifier = '...', name = '...', grade = 2, ... }, ... }
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Nombre del trabajo |

**Returns:** `table` — Lista de empleados. Devuelve `{}` si el trabajo no existe.

---

### Multi-Job

Todos reciben `identifier` que en QBCore es el **citizenid** del personaje y en ESX es el **license**. Ver [Multi-Job](multijob.md) para el sistema completo.

!!! warning "QBCore + multichar"
    Nunca pases un `license:...` a estos exports en QBCore: la sesion lee `nb_player_jobs` por `citizenid`, asi que una fila bajo `license:...` no se puede asociar de vuelta al personaje. Usa `Bridge.GetIdentifier(source)` (citizenid del personaje activo) o un citizenid conocido.

#### getPlayerJobs

Lista todas las filas de `nb_player_jobs` de un jugador.

```lua
local jobs = exports['nb-jobmanagers']:getPlayerJobs(identifier)
-- jobs = { { job_name = 'police', grade = 2, active = true, ... }, ... }
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `identifier` | string | citizenid (QBCore) o license (ESX) |

**Returns:** `table[]` — Lista de trabajos del jugador. `{}` si no tiene ninguno.

---

#### getActiveJob

Devuelve la fila activa de `nb_player_jobs` del jugador (la que espeja el framework).

```lua
local active = exports['nb-jobmanagers']:getActiveJob(identifier)
```

**Returns:** `table|nil` — Fila activa o `nil` si no tiene ninguna.

---

#### addPlayerJob

Registra un trabajo en `nb_player_jobs` **sin activarlo** en el framework. Si la pareja (identifier, jobName) ya existe, actualiza el grado y `assigned_by`.

```lua
exports['nb-jobmanagers']:addPlayerJob(identifier, 'mechanic', 3, 'admin')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `identifier` | string | citizenid (QBCore) o license (ESX) |
| `jobName` | string | Nombre del trabajo |
| `grade` | number | Grado inicial (default 0) |
| `assignedBy` | string | `'admin'`, `'boss'`, `'self'`, `'system'` |

---

#### removePlayerJob

Quita un trabajo. Si era el activo, promueve automaticamente al siguiente de mayor grado o deja al jugador en `unemployed`.

```lua
exports['nb-jobmanagers']:removePlayerJob(identifier, 'mechanic')
```

---

#### setActiveJob

Cambia cual es el trabajo activo del jugador y lo sincroniza con el framework (via `Bridge.SetJob`).

```lua
exports['nb-jobmanagers']:setActiveJob(identifier, 'police')
```

---

#### setJobAutoSelect

Expone u oculta un trabajo en la pestaña "Trabajos Disponibles" de F7 (escribe una fila en `nb_job_autoselect`).

```lua
exports['nb-jobmanagers']:setJobAutoSelect('taxi', true)   -- expone
exports['nb-jobmanagers']:setJobAutoSelect('taxi', false)  -- oculta
```

---

### Facturacion

#### createInvoice

Crea una factura desde otro script.

```lua
local invoiceId = exports['nb-jobmanagers']:createInvoice(
    'police',      -- trabajo emisor
    targetSource,  -- jugador objetivo
    5000,          -- monto
    'Multa de transito',  -- descripcion
    emitterSource  -- jugador emisor
)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `jobName` | string | Trabajo que emite la factura |
| `targetSource` | number | ID del jugador que debe pagar |
| `amount` | number | Monto de la factura |
| `description` | string | Descripcion de la factura |
| `emitterSource` | number | ID del jugador que emite |

**Returns:** `number` — ID de la factura creada.

---

#### getPlayerInvoices

Obtiene las facturas de un jugador, opcionalmente filtradas por estado.

```lua
local invoices = exports['nb-jobmanagers']:getPlayerInvoices(identifier, 'pending')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `playerIdentifier` | string | Identificador del jugador |
| `status` | string | *(opcional)* Filtrar por estado: `pending`, `paid`, `rejected`, `cancelled` |

**Returns:** `table` — Lista de facturas.

---

#### payInvoice

Paga una factura programaticamente. Deduce el monto del jugador y lo agrega a la sociedad.

```lua
local success = exports['nb-jobmanagers']:payInvoice(invoiceId, playerSource)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `invoiceId` | number | ID de la factura |
| `playerSource` | number | ID del jugador que paga |

**Returns:** `boolean`

---

#### cancelInvoice

Cancela una factura pendiente.

```lua
local success = exports['nb-jobmanagers']:cancelInvoice(invoiceId)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `invoiceId` | number | ID de la factura |

**Returns:** `boolean`

---

### Garaje de sociedad

#### addSocietyVehicle

Agrega un vehiculo al garaje de sociedad.

```lua
local success = exports['nb-jobmanagers']:addSocietyVehicle(garageId, 'police', 'police2', 'Patrulla')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `garageId` | string | ID del garaje (formato: `job_police_garage_1`) |
| `jobName` | string | Nombre del trabajo |
| `model` | string | Modelo del vehiculo |
| `label` | string | Nombre visible del vehiculo |

**Returns:** `boolean`

---

#### removeSocietyVehicle

Elimina un vehiculo del garaje de sociedad.

```lua
local success = exports['nb-jobmanagers']:removeSocietyVehicle(vehicleId)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `vehicleId` | number | ID del vehiculo en la BD |

**Returns:** `boolean`

---

#### getGarageVehicles

Obtiene los vehiculos disponibles en un garaje de sociedad.

```lua
local vehicles = exports['nb-jobmanagers']:getGarageVehicles(source, garageId, garageType, 'police')
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `source` | number | ID del jugador |
| `garageId` | string | ID del garaje |
| `garageType` | string | Tipo de garaje |
| `jobName` | string | Nombre del trabajo |

**Returns:** `table` — Lista de vehiculos.

---

## Client-side

Exports disponibles desde scripts del **cliente**.

---

### Estado del jugador

#### IsPlayerHandcuffed

Verifica si el jugador local esta esposado.

```lua
local cuffed = exports['nb-jobmanagers']:IsPlayerHandcuffed()
```

**Returns:** `boolean`

---

#### IsPlayerDragged

Verifica si el jugador local esta siendo arrastrado.

```lua
local dragged = exports['nb-jobmanagers']:IsPlayerDragged()
```

**Returns:** `boolean`

---

## Ejemplo de uso: script de mineria con sociedad

```lua
-- En tu script de mineria (server)
RegisterNetEvent('miner:server:sellOre', function(amount)
    local src = source
    local price = amount * 50

    -- Pagar al jugador
    Bridge.AddMoney(src, 'cash', price)

    -- Agregar porcentaje a la sociedad
    local tax = math.floor(price * 0.1)
    exports['nb-jobmanagers']:addSocietyMoney('miner', tax, 'Impuesto de venta de mineral')
end)
```

Asegurate de que **nb-jobmanagers** este en `dependencies` de tu `fxmanifest.lua`:

```lua
dependencies {
    'nb-jobmanagers',
}
```
