# Comandos

---

## Admin

| Comando | Quién | Qué hace |
|---------|-------|----------|
| `/warehouseadmin` | admin | Abre el creador in-game: crear, editar, eliminar almacenes de cualquier tipo. |
| `/whadmin` | admin | Alias del anterior. |

Gate server-side con `Bridge.player.isAdmin(source)` — nunca se confía en un
flag del cliente. Sin permiso, el jugador recibe un toast de error y la NUI
ni siquiera se abre.

---

## Dueño / gestión

| Comando | Quién | Qué hace |
|---------|-------|----------|
| `/warehousemanage` | dueño primario | Abre el panel de gestión: poner/cambiar/quitar contraseña, renovar renta, subir de nivel. |
| `/whmanage` | dueño primario | Alias del anterior. |

Solo funciona si hay un almacén habilitado dentro del radio de interacción
**y** el jugador es el dueño primario (`owner_type` = `player`/`job`/`gang`
con coincidencia exacta de identifier, o job/gang + grado ≥ `min_grade`) —
**nunca** alguien que solo tiene acceso vía la whitelist de
`nb_warehouse_access`. El servidor revalida esto en cada acción
(`Warehouse.IsManager`).

Todos los nombres son configurables en `shared/config.lua`
(`Config.AdminCommand`/`Config.ManageCommand` y sus alias).

---

## Interacción in-game (sin comando)

| Tecla | Contexto | Acción |
|-------|----------|--------|
| `[E]` | Cerca de un almacén habilitado | Abrir / comprar / rentar / ingresar contraseña / intentar robo, según el estado del almacén y el acceso del jugador. |
| `[E]` | Durante captura de ubicación (creador admin) | Captura coords + heading en el punto actual. |
| `[BACKSPACE]` | Durante captura de ubicación | Cancela la captura y vuelve a la NUI. |

El servidor decide qué opción(es) mostrar en cada caso — el cliente nunca
asume el resultado por su cuenta.
