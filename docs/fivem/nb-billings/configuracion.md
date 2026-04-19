# Configuracion

Todo en `shared/config.lua`.

---

## General

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos con permiso elevado (cancelar ajenas) | `{ 'admin', 'superadmin', 'god' }` |
| `Config.Debug` | Prints de debug | `false` |
| `Config.Locale` | Idioma (`'en'` / `'es'`) | `'en'` |

---

## Comandos

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Commands.Invoice` | Crear factura rapida | `'invoice'` |
| `Config.Commands.MyInvoices` | Abrir UI | `'myinvoices'` |

Sintaxis: `/invoice <playerId> <amount> <title>`. El resto de parametros (categoria, tax custom, metadata) se hacen desde la UI o via export.

---

## Billing

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Billing.MoneyType` | Cuenta por defecto para cobros | `'bank'` |
| `Config.Billing.MaxAmount` | Importe maximo por factura | `1000000` |
| `Config.Billing.MinAmount` | Importe minimo | `1` |
| `Config.Billing.AllowPartialPayments` | Pagos parciales | `true` |
| `Config.Billing.MinPartialAmount` | Minimo por pago parcial | `1` |
| `Config.Billing.AllowPlayerToPlayer` | Jugadores sin trabajo pueden facturar a otros | `true` |
| `Config.Billing.AllowReject` | Destinatario puede rechazar | `true` |
| `Config.Billing.AutoExpireDays` | Dias antes de auto-cancelar (`0` = nunca) | `30` |
| `Config.Billing.OverdueEnabled` | Marcar `overdue` automaticamente tras `due_date` | `true` |

---

## Impuestos

```lua
Config.Tax.Enabled      = true
Config.Tax.DefaultRate  = 7.5
Config.Tax.CategoryRates = {
    general  = 7.5,
    medical  = 0,
    repair   = 10,
    fine     = 0,
    service  = 7.5,
    rent     = 5,
}
```

- Se aplica al crear la factura: `tax_amount = floor(subtotal * rate / 100)`.
- La UI muestra el desglose al destinatario (subtotal + tax = total).
- Ver [Impuestos](impuestos.md) para detalles.

---

## Categorias

```lua
Config.Categories = {
    { id = 'general',  label = 'General',  icon = 'receipt' },
    { id = 'medical',  label = 'Medico',   icon = 'heart' },
    { id = 'repair',   label = 'Taller',   icon = 'wrench' },
    { id = 'fine',     label = 'Multa',    icon = 'alert-triangle' },
    { id = 'service',  label = 'Servicio', icon = 'briefcase' },
    { id = 'rent',     label = 'Alquiler', icon = 'home' },
}
```

Anade la que quieras — solo exige `id`, `label` y (opcional) `icon` (lucide icon name).

---

## UI

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.UI.PageSize` | Facturas por pagina | `20` |
| `Config.UI.NearbyRadius` | Radio de deteccion de cercanos al crear | `10.0` (m) |

---

## Idiomas

Los textos estan en `shared/locale.lua` bajo `Locale['en']` y `Locale['es']`. Duplica el bloque y pon `Config.Locale = 'xx'` para un idioma nuevo.

---

## Hooks opcionales

nb-billings detecta automaticamente [nb-jobscreator](../nb-jobmanagers/index.md) para depositar los cobros de facturas `asJob` en la sociedad. Si tienes otro sistema de sociedades, edita `bridge/society.lua` (archivo open):

```lua
function Bridge.Society.Deposit(jobName, amount, reason)
    exports['mi-script-societies']:AddMoney(jobName, amount, reason)
end
```
