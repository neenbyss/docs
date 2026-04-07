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

Los nombres de los comandos se pueden cambiar en [Configuracion](configuracion.md).

---

## Teclas

| Tecla | Descripcion | Quien puede usarlo |
|-------|-------------|--------------------|
| **E** | Interactuar con markers (boss menu, garaje, stash, duty, vestuario) | Empleados del trabajo del marker |
| **F6** | Abrir menu de acciones rapidas | Empleados con permisos de accion |

La tecla de interaccion se configura en `Config.Markers.InteractionKey` y la de acciones en `Config.Actions.Keybind`.

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
