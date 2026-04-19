# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Artifacts 5181+ |
| **oxmysql** | Persistencia de facturas y pagos |
| **nb-bridge** | Bridge centralizado |
| **Framework** | ESX Legacy o QBCore |
| **nb-jobscreator (opcional)** | Deposito automatico en sociedades |

---

## 1. Retirar sistemas previos

nb-billings es **standalone**. Retira `esx_billing`, `qb-billing`, `okokBilling` u otros del `server.cfg`. Los eventos antiguos no son compatibles.

---

## 2. Instalar el recurso

1. Coloca **nb-billings** en `resources/`.
2. Comprueba que **nb-bridge** este instalado.

---

## 3. Base de datos

Importa `[sql]/nb_billings.sql`:

```bash
mysql -u usuario -p nombre_db < nb-billings/\[sql\]/nb_billings.sql
```

Tablas creadas:

| Tabla | Descripcion |
|-------|-------------|
| `nb_billing_invoices` | Facturas (emisor, destinatario, importe, estado, categoria, impuestos). |
| `nb_billing_payments` | Historial de pagos (importe, metodo, pagador). Ligado por FK cascade. |

Estados posibles: `pending` → `paid` / `partial` → `paid` / `overdue` / `cancelled` / `rejected`.

---

## 4. Configuracion minima

Edita `shared/config.lua`:

```lua
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
Config.Locale      = 'es'

Config.Commands.Invoice     = 'invoice'       -- /invoice
Config.Commands.MyInvoices  = 'myinvoices'    -- /myinvoices

Config.Billing.MoneyType          = 'bank'    -- 'bank' o 'cash'
Config.Billing.MaxAmount          = 1000000
Config.Billing.AllowPartialPayments = true
Config.Billing.AllowPlayerToPlayer  = true    -- permitir facturas entre jugadores sin trabajo
Config.Billing.AllowReject          = true
Config.Billing.AutoExpireDays       = 30

Config.Tax.Enabled      = true
Config.Tax.DefaultRate  = 7.5
Config.Tax.CategoryRates = {
    medical = 0,
    fine    = 0,
}
```

---

## 5. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended       # o qb-core
ensure nb-bridge
ensure nb-billings
```

---

## 6. Comprobar que funciona

1. Entra al servidor.
2. Ejecuta `/myinvoices` — abre la UI (vacia la primera vez).
3. Ejecuta `/invoice <playerId> 100 Prueba` — crea una factura rapida via comando.
4. Con el jugador destinatario, `/myinvoices` y pulsa **Pagar**.
