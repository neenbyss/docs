# Uso

Flujo completo desde el punto de vista del jugador: crear, cobrar, pagar, rechazar, cancelar.

---

## Abrir el panel

`/myinvoices` abre el panel con cuatro secciones (sidebar):

- **Recibidas** — facturas donde eres el target. Aqui pagas o rechazas.
- **Enviadas** — facturas donde eres el emisor. Aqui puedes cancelar las que no se han pagado.
- **Crear** — formulario para emitir una factura nueva.
- **Stats** — dashboard con KPIs (pending, paid, overdue, total adeudado, total pagado).

---

## Crear una factura

### Desde la UI

1. Abre **Crear**.
2. Selecciona el destinatario:
    - **Picker de cercanos** — lista de jugadores en tu radio (default 10m, configurable).
    - **Por ID** — escribir el server id a mano.
3. Rellena:
    - **Categoria** (general / medico / multa / ...).
    - **Titulo** y descripcion.
    - **Importe** (subtotal — el impuesto se anade automaticamente y se muestra antes de guardar).
    - **Como trabajo** — toggle si quieres emitirla en nombre de tu job (el cobro ira a la sociedad).
    - **Fecha de vencimiento** (opcional).
    - **Notas internas** — no visibles al destinatario.
4. Pulsa **Emitir**. El destinatario recibe un toast en pantalla.

### Via comando rapido

```
/invoice <playerId> <amount> <title>
```

Emite una factura con categoria `general`, sin descripcion, sin due_date, con el `Config.Billing.MoneyType` por defecto.

### Via export (desde otro script)

```lua
local result = exports['nb-billings']:createInvoice({
    targetSource = 12,
    amount       = 750,
    title        = 'Servicio medico',
    category     = 'medical',
    asJob        = true,        -- lo emite como sociedad (cobro va al society)
    dueDate      = '2026-05-01 12:00',
})

if result.success then
    print('Factura creada:', result.invoiceNumber)     -- 'INV-000123'
end
```

---

## Pagar una factura

En **Recibidas**:

- **Pagar total** — paga el `amount - amount_paid` pendiente.
- **Pago parcial** (si `AllowPartialPayments = true`) — introduce un importe minimo (`MinPartialAmount`) y elige metodo (`bank` o `cash`).

La UI valida que tienes fondos antes de enviar. El servidor re-valida para evitar dobles pagos simultaneos.

Al pagar:

1. Se resta del `bank` o `cash`.
2. Se anade a la sociedad si la factura es `asJob`.
3. Se crea una fila en `nb_billing_payments` con `method = 'direct'` o `'partial'`.
4. La factura pasa a `paid` (si todo) o `partial` (si aun queda).
5. Emisor recibe notificacion.

---

## Rechazar una factura

Si `AllowReject = true`, el destinatario puede rechazar una factura pendiente.

- Estado pasa a `rejected`.
- No se puede recuperar ni reabrir — el emisor tendria que crear una nueva.
- El emisor recibe notificacion.

> Usa rejection para disputas. Si la factura ya tiene pagos parciales, no se puede rechazar; solo cancelar (por admin).

---

## Cancelar una factura

El **emisor** puede cancelar su propia factura mientras no haya recibido ningun pago:

- Estado pasa a `cancelled`.
- La factura se guarda en BD (para auditoria) pero deja de aparecer como pendiente.

Los admins (`Config.AdminGroups`) pueden cancelar **cualquier** factura pendiente.

---

## Overdue y auto-expiry

- **Overdue** — las facturas con `due_date < NOW()` pasan a `overdue` automaticamente cada 5 minutos. Son pagables igual que pending, solo cambia el color / badge.
- **Auto-expiry** — si `Config.Billing.AutoExpireDays > 0`, las facturas `pending` con mas de N dias sin actividad se cancelan automaticamente.

Los pagos parciales **no** expiran — una vez que alguien paga algo, la factura queda abierta indefinidamente hasta completarla o cancelarla manualmente.

---

## Notificaciones al jugador

- **Login** — `You have N pending invoices` si el jugador entra con facturas pendientes.
- **Factura recibida** — toast con titulo + importe.
- **Pago recibido** — toast al emisor cuando el destinatario paga.
- **Rechazo** — toast al emisor.
- **Overdue** — se puede anadir un periodic ping (ver hook en `server/main.lua`).

Todas usan `Bridge.Notify` / `Bridge.ShowNotification`, compatibles con ox_lib / ESX / QBCore / native.
