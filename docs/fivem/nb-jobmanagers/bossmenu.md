# Boss Menu

El boss menu es la interfaz principal para que los jefes gestionen su trabajo: dinero de sociedad, empleados, facturas e historial.

---

## Acceso

Para abrir el boss menu:

1. El jugador debe tener un trabajo asignado.
2. El grado del jugador debe tener el permiso `BOSS_MENU` (valor 1). Ver [Permisos](permisos.md).
3. Acercarse a un marker de tipo `bossmenu` del trabajo y presionar **E**.
4. O usar el comando `/bossmenu`.

---

## Dinero de sociedad

Cada trabajo tiene una cuenta de sociedad almacenada en la tabla `nb_society_money`. Desde el boss menu se puede:

- **Depositar** — Transfiere dinero del jugador a la sociedad.
- **Retirar** — Transfiere dinero de la sociedad al jugador.

El tipo de dinero (cash o bank) se configura en `Config.BossMenu.MoneyType`.

Todas las transacciones quedan registradas en `nb_society_transactions` con:

- Quien hizo la operacion
- Tipo (deposito, retiro, pago de factura)
- Monto
- Fecha y hora

El historial es visible desde el boss menu (ultimas 50 transacciones por defecto).

---

## Gestion de empleados

Desde el boss menu, segun los permisos del grado:

| Accion | Permiso requerido | Descripcion |
|--------|-------------------|-------------|
| Invitar | `EMPLOYEE_INVITE` | Busca jugadores cercanos y les envia invitacion |
| Promover/Degradar | `EMPLOYEE_MANAGE` | Cambia el grado de un empleado |
| Despedir | `EMPLOYEE_MANAGE` | Remueve al empleado del trabajo |

El radio de busqueda para invitaciones se configura en `Config.BossMenu.InviteRadius`.

Cuando un jugador recibe una invitacion, le aparece una notificacion para aceptar o rechazar.

---

## Facturacion

Si `Config.Billing.Enabled = true`, los empleados con permiso `INVOICE_CREATE` pueden crear facturas desde el boss menu.

### Flujo de una factura

1. El empleado crea la factura indicando jugador objetivo, monto y descripcion.
2. El jugador recibe una notificacion con la factura pendiente.
3. El jugador puede **pagar** o **rechazar** la factura desde `/myinvoices`.
4. Si paga, el monto se deduce del jugador y se agrega a la cuenta de sociedad.
5. Un jefe con `BOSS_MENU` puede cancelar facturas pendientes.

### Estados de factura

| Estado | Descripcion |
|--------|-------------|
| `pending` | Pendiente de pago |
| `paid` | Pagada por el jugador |
| `rejected` | Rechazada por el jugador |
| `cancelled` | Cancelada por un jefe |

### Integracion con nb-billings

Si `Config.Billing.UseNbBillings = 'auto'` y nb-billings esta instalado, el sistema de facturas se delega completamente a nb-billings. La UI y el flujo de pago los maneja nb-billings en lugar del sistema interno.

---

## Garaje de sociedad

Si el trabajo tiene markers de tipo `garage`, los empleados pueden acceder a vehiculos compartidos de la sociedad.

- Los vehiculos se gestionan desde el panel admin (agregar/eliminar modelos).
- Cualquier empleado del trabajo puede sacar o guardar vehiculos.
- Si `Config.Garage.SocietyMaxOut > 0`, se limita cuantos vehiculos del mismo modelo pueden estar fuera a la vez.
- Con `Config.Garage.UseNbGarages = 'auto'`, se integra con nb-garages si esta disponible.

---

## Vestuario

Los markers de tipo `clothing` permiten a los empleados cambiar de uniforme. Los outfits se guardan en la tabla `nb_job_outfits` y se gestionan desde la interfaz de vestuario.

---

## Stash

Los markers de tipo `stash` abren un inventario compartido del trabajo. La capacidad se configura en `Config.Stash.Slots` y `Config.Stash.MaxWeight`.

---

## Toggle duty

Los markers de tipo `duty` permiten al jugador entrar y salir de servicio.
