# Exports

nb-billings expone una API server para integrar facturacion desde cualquier otro recurso.

---

## createInvoice

Crea una factura.

```lua
local result = exports['nb-billings']:createInvoice({
    targetSource = 12,                     -- o targetIdentifier + targetName
    amount       = 500,                    -- subtotal
    title        = 'Servicio medico',
    category     = 'medical',              -- clave en Config.Categories
    description  = 'Detalle visible al target',
    notes        = 'Notas internas',       -- no visibles
    asJob        = true,                   -- usa el job del emisor para firmar
    dueDate      = '2026-05-01 12:00',
    taxRate      = 0,                      -- override opcional
    metadata     = { misionId = 42 },      -- JSON libre
})

-- result = { success = true, invoiceId = 17, invoiceNumber = 'INV-000017' }
-- o
-- result = { success = false, error = 'You cannot invoice yourself' }
```

| Campo de `data` | Tipo | Descripcion |
|-----------------|------|-------------|
| `targetSource` | number | Server id del destinatario. Si no, usa `targetIdentifier` + `targetName`. |
| `amount` | number | Subtotal (sin impuestos). |
| `title` | string | Obligatorio. |
| `category` | string | Clave en `Config.Categories`. Default: `'general'`. |
| `description` | string | Texto visible al destinatario. |
| `notes` | string | Notas internas (invisibles). |
| `asJob` | bool | Si `true` y el emisor tiene trabajo, la firma es del job + los cobros van a la sociedad. |
| `dueDate` | string | `'YYYY-MM-DD HH:MM'`. Marca `overdue` si se pasa. |
| `taxRate` | number | Override del IVA. |
| `metadata` | table | JSON libre persistido en la fila. |

---

## payInvoice

Paga una factura. Si `amount` se omite, paga el total pendiente.

```lua
local ok = exports['nb-billings']:payInvoice(invoiceId, playerSource, amount, moneyType)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `invoiceId` | number | Id de la factura. |
| `playerSource` | number | Quien paga. |
| `amount` | number? | `nil` = total pendiente. |
| `moneyType` | string? | `'bank'` / `'cash'`. Default `Config.Billing.MoneyType`. |

---

## cancelInvoice

Cancela una factura pendiente sin pagos. Solo el emisor o un admin.

```lua
local ok = exports['nb-billings']:cancelInvoice(invoiceId)
```

---

## rejectInvoice

El destinatario rechaza la factura.

```lua
local ok = exports['nb-billings']:rejectInvoice(invoiceId, playerSource)
```

---

## getInvoice / getInvoiceByNumber

```lua
local invoice = exports['nb-billings']:getInvoice(invoiceId)
local invoice = exports['nb-billings']:getInvoiceByNumber('INV-000123')
```

Devuelve la fila con todos los campos.

---

## getPlayerInvoices

```lua
local invoices = exports['nb-billings']:getPlayerInvoices(identifier, {
    status   = 'pending',
    category = 'medical',
})
```

---

## getJobInvoices

```lua
local invoices = exports['nb-billings']:getJobInvoices('mechanic', { status = 'paid' })
```

---

## countPendingInvoices

```lua
local n = exports['nb-billings']:countPendingInvoices(identifier)
-- n = 3
```

Cuenta `pending + partial + overdue`.

---

## getPlayerStats

```lua
local stats = exports['nb-billings']:getPlayerStats(identifier)
-- { pending = 3, paid = 12, overdue = 1, totalOwed = 850, totalPaid = 4300 }
```

Usado por el dashboard del panel.

---

## Eventos

### Server -> Client

| Evento | Cuando |
|--------|--------|
| `nb-billings:client:openInvoices` | Abrir UI con lista + stats. |
| `nb-billings:client:updateInvoices` | Refrescar recibidas. |
| `nb-billings:client:updateSentInvoices` | Refrescar enviadas. |
| `nb-billings:client:invoiceDetail` | Mostrar detalle + historial de pagos. |
| `nb-billings:client:receiveInvoice` | Toast: nueva factura recibida. |

### Client -> Server

| Evento | Payload |
|--------|---------|
| `nb-billings:server:createInvoice` | data |
| `nb-billings:server:payInvoice` | invoiceId, amount?, moneyType? |
| `nb-billings:server:rejectInvoice` | invoiceId |
| `nb-billings:server:cancelInvoice` | invoiceId |
| `nb-billings:server:getMyInvoices` | filters? |
| `nb-billings:server:getSentInvoices` | filters? |
| `nb-billings:server:getInvoiceDetail` | invoiceId |

---

## Ejemplos

### Factura automatica al morir en el hospital

```lua
-- server
AddEventHandler('esx_ambulancejob:onRevive', function(targetId)
    exports['nb-billings']:createInvoice({
        targetSource = targetId,
        amount       = 1500,
        title        = 'Atencion medica de urgencia',
        category     = 'medical',
        asJob        = true,        -- cobra para la sociedad ambulancia
    })
end)
```

### Multa de trafico desde policia

```lua
-- server (llamado por policia)
RegisterNetEvent('police:issueFine', function(targetId, amount, reason)
    exports['nb-billings']:createInvoice({
        targetSource = targetId,
        amount       = amount,
        title        = 'Multa de trafico',
        description  = reason,
        category     = 'fine',
        asJob        = true,
        taxRate      = 0,
        dueDate      = os.date('%Y-%m-%d', os.time() + 7 * 86400),
    })
end)
```

### Recordar deuda al login

```lua
-- server
AddEventHandler('esx:playerLoaded', function(playerId, xPlayer)
    local n = exports['nb-billings']:countPendingInvoices(xPlayer.identifier)
    if n > 0 then
        TriggerClientEvent('chat:addMessage', playerId, {
            args = { 'BANK', ('Tienes %d facturas pendientes. Usa /myinvoices'):format(n) },
        })
    end
end)
```
