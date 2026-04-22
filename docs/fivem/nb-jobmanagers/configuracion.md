# Configuracion

Todo se edita en **shared/config.lua** en la raiz del recurso.

---

## Opciones generales

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos que pueden abrir el panel admin | `{ 'admin', 'superadmin', 'god' }` |
| `Config.Command` | Comando para abrir el panel admin | `'jobmanager'` |
| `Config.Debug` | Activa logs detallados en consola | `false` |
| `Config.Locale` | Idioma activo: `'es'` o `'en'` | `'en'` |
| `Config.WebhookURL` | URL de webhook de Discord para logs | `''` |

---

## Tipos de trabajo

```lua
Config.JobTypes = { 'civ', 'leo', 'ems', 'fire', 'mechanic', 'custom' }
```

Cada trabajo tiene un tipo que determina que acciones rapidas estan disponibles. Se asigna al crear o editar el trabajo desde el panel admin.

---

## Grado por defecto

```lua
Config.DefaultGrade = {
    grade = 0,
    name = 'recruit',
    label = 'Recruit',
    salary = 0,
}
```

Se usa al crear un trabajo nuevo si no se especifica otro grado inicial.

---

## Markers

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Markers.Types` | Tipos de marker disponibles | `bossmenu`, `garage`, `stash`, `duty`, `clothing` |
| `Config.Markers.DefaultRadius` | Radio de interaccion | `2.0` |
| `Config.Markers.DrawDistance` | Distancia de renderizado del marker 3D | `15.0` |
| `Config.Markers.InteractionKey` | Tecla de interaccion (38 = E) | `38` |

### Blips

Cada tipo de marker tiene su propio blip en el mapa:

| Tipo | Sprite | Color | Escala |
|------|--------|-------|--------|
| `bossmenu` | 280 | 5 | 0.8 |
| `garage` | 357 | 3 | 0.8 |
| `stash` | 478 | 2 | 0.8 |
| `duty` | 480 | 2 | 0.8 |
| `clothing` | 366 | 4 | 0.8 |

### Marker 3D

```lua
Config.Markers.Marker3D = {
    type = 27,
    scale = vector3(1.0, 1.0, 0.5),
    color = { r = 99, g = 102, b = 241, a = 120 },
    bobUpAndDown = true,
    rotate = true,
}
```

---

## Stash

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Stash.Slots` | Slots del inventario del stash | `50` |
| `Config.Stash.MaxWeight` | Peso maximo del stash | `100000` |

---

## Boss Menu

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.BossMenu.MoneyType` | Tipo de dinero para depositos/retiros: `'cash'` o `'bank'` | `'cash'` |
| `Config.BossMenu.InviteRadius` | Radio para buscar jugadores cercanos al invitar | `10.0` |
| `Config.BossMenu.MaxTransactionHistory` | Cantidad de transacciones a mostrar en el historial | `50` |
| `Config.BossMenu.Command` | Comando alternativo para abrir el boss menu | `'bossmenu'` |

---

## Facturacion

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Billing.Enabled` | Activa el sistema de facturas | `true` |
| `Config.Billing.MaxAmount` | Monto maximo por factura | `1000000` |
| `Config.Billing.MoneyType` | De donde paga el jugador: `'cash'` o `'bank'` | `'bank'` |
| `Config.Billing.Command` | Comando para crear factura rapida | `'invoice'` |
| `Config.Billing.MyInvoicesCommand` | Comando para ver facturas pendientes | `'myinvoices'` |
| `Config.Billing.UseExternal` | Delegar al sistema de facturacion del framework | `false` |
| `Config.Billing.UseNbBillings` | Usar nb-billings si esta disponible: `'auto'`, `true`, `false` | `'auto'` |

Con `UseNbBillings = 'auto'`, el sistema detecta automaticamente si nb-billings esta instalado y lo usa. Si no, usa el sistema interno con la tabla `nb_invoices`.

---

## Garaje

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Garage.SpawnDistance` | Distancia de spawn del vehiculo | `5.0` |
| `Config.Garage.StoreDistance` | Distancia maxima para guardar vehiculo | `8.0` |
| `Config.Garage.DefaultHeading` | Heading por defecto al spawnear | `0.0` |
| `Config.Garage.SpawnOffset` | Offset de posicion del spawn | `vector3(0.0, 3.0, 0.0)` |
| `Config.Garage.SocietyMaxOut` | Max vehiculos de sociedad fuera por modelo (0 = sin limite) | `0` |
| `Config.Garage.UseNbGarages` | Usar nb-garages si esta disponible: `'auto'`, `true`, `false` | `'auto'` |

---

## Multi-Job

Sistema que permite a un jugador tener hasta N trabajos a la vez y cambiar entre ellos con `/myjobs` (F7). Ver [Multi-Job](multijob.md) para el detalle.

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.MultiJob.Enabled` | Activa el sistema (desactivar oculta F7, `/myjobs` y la NUI) | `true` |
| `Config.MultiJob.MaxJobsPerPlayer` | Limite de trabajos por jugador (0 = sin limite) | `3` |
| `Config.MultiJob.ChangeCooldown` | Segundos entre cambios (0 = sin cooldown) | `300` |
| `Config.MultiJob.AllowSelfAssign` | Toggle maestro de "Trabajos Disponibles" (false oculta toda la pestaña) | `true` |
| `Config.MultiJob.SelfAssignDefault` | Comportamiento por defecto para trabajos sin fila en `nb_job_autoselect`: `false` = opt-in (admin expone con `/publicjob`), `true` = legacy (cualquier `whitelisted = 0` es publico) | `false` |
| `Config.MultiJob.Command` | Comando para abrir el menu | `'myjobs'` |
| `Config.MultiJob.Keybind` | Tecla FiveM por defecto (rebindeable por el jugador) | `'F7'` |
| `Config.MultiJob.KeybindDescription` | Etiqueta mostrada en el Key Map de FiveM | `'My Jobs'` |
| `Config.MultiJob.SyncOnPlayerLoad` | Al conectar, restaura el trabajo activo en el framework desde `nb_player_jobs` | `true` |

!!! tip "Modo recomendado"
    `SelfAssignDefault = false` (opt-in). Los trabajos solo aparecen en F7 cuando un admin los expone con `/publicjob <job> on`. Evita que un jugador vea un trabajo que no puede tomar.

---

## Acciones rapidas

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Actions.Keybind` | Tecla para abrir el menu de acciones | `'F6'` |
| `Config.Actions.MaxPlayerDistance` | Distancia maxima al jugador objetivo | `3.0` |
| `Config.Actions.MaxVehicleDistance` | Distancia maxima al vehiculo objetivo | `5.0` |
| `Config.Actions.Handcuffs.RequireItem` | Requiere item de esposas en inventario | `true` |
| `Config.Actions.Handcuffs.ItemName` | Nombre del item de esposas | `'handcuffs'` |
