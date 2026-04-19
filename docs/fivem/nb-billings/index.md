# nb-billings

Sistema de **facturacion** standalone para FiveM. Cualquier jugador o trabajo puede emitir facturas; el destinatario puede pagarlas completas o a plazos, rechazarlas o ignorarlas. Impuestos por categoria, deposito automatico a sociedades de trabajo, historial de pagos auditables y panel Vue 3.

---

## Caracteristicas

- 🧾 **Emisores multiples** — `player` (persona), `job` (un jugador actua en nombre de su trabajo), `system` (desde otro script).
- 💳 **Dos metodos de pago** — `cash` o `bank`, configurables por defecto.
- 💸 **Pagos parciales** — el destinatario puede pagar en varios tramos; el sistema lleva `amount_paid` vs `amount`.
- 🧮 **Impuestos por categoria** — tasa global + overrides por tipo de factura (medical = 0%, multas = 0%, servicio = 7.5%).
- 🏢 **Integracion con sociedades** — cuando la factura se emite `asJob`, los cobros entran automaticamente a la cuenta de sociedad (compatible con nb-jobscreator).
- ⏰ **Overdue tracking** — facturas con `due_date` vencida pasan a estado `overdue` automaticamente cada 5 minutos.
- 🗑️ **Auto-expiry** — facturas pendientes viejas (>N dias) se cancelan solas.
- 📊 **Dashboard** — stats por jugador (pending, paid, overdue, total adeudado, total pagado).
- 📜 **Historial** — tabla separada (`nb_billing_payments`) con cada pago (monto, metodo, pagador).
- 👥 **Selector de cercanos** — al crear una factura puedes elegir al destinatario desde un listado de jugadores en tu radio (default 10m).
- 🌐 **Multi-framework** — ESX + QBCore via nb-bridge.
- 🌎 **i18n** — EN / ES.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy / QBCore |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge |
| **Sociedades (opcional)** | nb-jobscreator (auto-detectado) o adaptador custom |

> **No es compatible con `esx_billing` / `qb-billing`** — es un sistema separado. Si los tienes instalados, retiralos primero.

---

## Secciones

- **[Instalacion](instalacion.md)** — Requisitos, SQL, arranque.
- **[Configuracion](configuracion.md)** — Todas las opciones.
- **[Uso](uso.md)** — Crear, pagar, rechazar, cancelar. Flujo completo.
- **[Impuestos](impuestos.md)** — Tasas globales + por categoria.
- **[Exports](exports.md)** — API para facturar desde otros recursos.
