# Impuestos

nb-billings soporta un sistema de impuestos con **tasa global + overrides por categoria**. El impuesto se calcula en el momento de crear la factura y se guarda en la tabla junto al subtotal.

---

## Habilitar

```lua
Config.Tax.Enabled = true
```

Si esta en `false`, el calculo salta y las facturas llevan `tax_rate = 0`, `tax_amount = 0`.

---

## Tasa por defecto

```lua
Config.Tax.DefaultRate = 7.5   -- porcentaje
```

Se aplica a cualquier factura cuya `category` no este en `Config.Tax.CategoryRates`.

---

## Tasas por categoria

```lua
Config.Tax.CategoryRates = {
    general  = 7.5,
    medical  = 0,         -- exento
    repair   = 10,
    fine     = 0,         -- multas sin IVA
    service  = 7.5,
    rent     = 5,
}
```

Al crear una factura con `category = 'medical'`, la tasa aplicada sera `0`.

---

## Override por factura

Tanto desde la UI (campo opcional) como desde el export, puedes forzar una tasa concreta:

```lua
exports['nb-billings']:createInvoice({
    targetSource = src,
    amount       = 500,
    title        = 'Evento especial',
    category     = 'service',
    taxRate      = 0,         -- override: sin IVA
})
```

Si se pasa `taxRate`, se usa ese valor en lugar de la tabla.

---

## Calculo

```
subtotal   = amount          -- lo que introduce el emisor
tax_amount = floor(subtotal * tax_rate / 100)
total      = subtotal + tax_amount
```

Se guardan en BD los tres campos (`subtotal`, `tax_rate`, `tax_amount`) + `amount` que es el total.

La UI muestra:

```
Subtotal           $500
IVA (10%)           $50
Total              $550
```

---

## Deposito a sociedades

Cuando una factura `asJob = true` se paga, el **importe total** (subtotal + tax) entra en la sociedad, no solo el subtotal. Si quieres que los impuestos vayan a una cuenta diferente (ej. "gobierno"), edita `bridge/society.lua`:

```lua
function Bridge.Society.Deposit(jobName, amount, reason, context)
    -- context = { subtotal, tax_amount, category, ... }
    if context and context.tax_amount > 0 then
        exports['mi-script']:AddMoney('government', context.tax_amount, 'tax_' .. context.category)
        amount = context.subtotal    -- solo subtotal a la sociedad
    end
    exports['nb-jobscreator']:AddGangMoney(jobName, amount)
end
```

> El parametro `context` se pasa al hook pero el default de nb-billings deposita el total completo.

---

## Mostrar al destinatario

El destinatario ve en la UI:

- Subtotal (lo que se factura).
- % de IVA aplicado.
- Importe de IVA en euros.
- Total a pagar.

Esto aparece tanto en el listado (resumido) como en el detalle de la factura.

---

## Deshabilitar solo para ciertas categorias

Si quieres impuestos solo para la `rent` y nada mas:

```lua
Config.Tax.Enabled      = true
Config.Tax.DefaultRate  = 0
Config.Tax.CategoryRates = {
    rent = 5,
}
```

Asi la tasa por defecto es 0 y solo `rent` lleva 5% de IVA.
