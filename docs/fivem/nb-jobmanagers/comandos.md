# Comandos

Lista de comandos y teclas disponibles en nb-jobmanagers.

---

## Comandos de chat

| Comando | Descripcion | Quien puede usarlo |
|---------|-------------|--------------------|
| `/jobmanager` | Abre el panel de administracion de trabajos | Solo admins (`Config.AdminGroups`) |
| `/bossmenu` | Abre el boss menu del trabajo actual | Empleados con permiso `BOSS_MENU` |
| `/invoice [id] [monto] [desc]` | Crea una factura rapida a un jugador | Empleados con permiso `INVOICE_CREATE` |
| `/myinvoices` | Muestra las facturas pendientes del jugador | Todos los jugadores |
| `/myjobs` | Abre el menu de Multi-Job (lista trabajos + auto-asignacion) | Todos los jugadores |

Los nombres de los comandos se pueden cambiar en [Configuracion](configuracion.md).

---

## Comandos de admin (Multi-Job)

| Comando | Uso | Descripcion |
|---------|-----|-------------|
| `/assignjob` | `/assignjob [id\|citizenid] [job] [grade]` | Asigna un trabajo a un jugador online u offline y lo activa en el framework. Grado opcional (default 0). |
| `/removejob` | `/removejob [id\|citizenid] [job]` | Quita un trabajo. Si era el activo, promueve el siguiente de mayor grado. |
| `/publicjob` | `/publicjob [job] [on\|off]` | Expone u oculta un trabajo en la pestaña "Trabajos Disponibles" de F7. |
| `/whichjobs` | `/whichjobs [id\|citizenid]` | Diagnostico: imprime en la consola del servidor cada fila de `nb_player_jobs` para el objetivo. |

Requieren que el admin este en `Config.AdminGroups` (o que se ejecuten desde la consola del servidor / RCON). Ver [Multi-Job](multijob.md) para el detalle del sistema.

!!! warning "QBCore + multichar: nunca pases `license:...`"
    En QBCore cada personaje tiene un `citizenid` distinto y es ese el que usa la sesion para leer `nb_player_jobs`. Si pasas un `license:...` a `/assignjob`, la fila queda bajo una clave que el F7 nunca consulta. Usa **server ID** (online) o el **citizenid** del personaje (offline). Desde 1.2.1 el comando rechaza `license:...` automaticamente.

---

## Teclas

| Tecla | Descripcion | Quien puede usarlo |
|-------|-------------|--------------------|
| **E** | Interactuar con markers (boss menu, garaje, stash, duty, vestuario) | Empleados del trabajo del marker |
| **F6** | Abrir menu de acciones rapidas | Empleados con permisos de accion |
| **F7** | Abrir menu de Multi-Job | Todos los jugadores |

La tecla de interaccion se configura en `Config.Markers.InteractionKey`, la de acciones en `Config.Actions.Keybind` y la de Multi-Job en `Config.MultiJob.Keybind`.

---

## Acciones rapidas (F6)

Las acciones disponibles dependen del tipo de trabajo y los permisos del grado. Ver [Permisos](permisos.md) para la tabla completa.

### Acciones sobre jugadores

| Accion | Permiso de grado | Accion de trabajo |
|--------|-------------------|-------------------|
| Esposar / Quitar esposas | `HANDCUFF` | `HANDCUFF` |
| Arrastrar jugador | `HANDCUFF` | `HANDCUFF` |
| Meter en vehiculo | `HANDCUFF` | `HANDCUFF` |
| Sacar de vehiculo | `HANDCUFF` | `HANDCUFF` |
| Verificar identidad | `HANDCUFF` | `HANDCUFF` |
| Verificar licencia de conducir | `HANDCUFF` | `HANDCUFF` |
| Verificar licencia de armas | `HANDCUFF` | `HANDCUFF` |

### Acciones de registro

| Accion | Permiso de grado | Accion de trabajo |
|--------|-------------------|-------------------|
| Registrar jugador (vivo) | `SEARCH_CUFFED` | `SEARCH_CUFFED` |
| Registrar cuerpo | `SEARCH_DEAD` | `SEARCH_DEAD` |

### Acciones sobre vehiculos

| Accion | Permiso de grado | Accion de trabajo |
|--------|-------------------|-------------------|
| Verificar propietario | `VEHICLE_CHECK` | `VEHICLE_CHECK` |

Para que una accion aparezca en el menu, **tanto** el trabajo como el grado deben tener el flag correspondiente habilitado.
