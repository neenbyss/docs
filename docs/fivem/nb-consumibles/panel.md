# Panel in-game

El panel es el centro de control de nb-consumibles. Todo se configura desde aqui: no tienes que tocar ningun fichero `.lua` para crear o modificar items.

Abre el panel con `/consumibles` (requiere pertenecer a `Config.AdminGroups`).

---

## Barra superior

- **Badge framework / inventario** — muestra lo detectado por nb-bridge. Sirve para verificar que los datos de `source` y el registro de usable items estan activos.
- **Selector de idioma** — cambia el idioma **de la UI del panel**. Es independiente de `Config.Locale`, que aplica a las notificaciones in-game.
- **Boton recargar** (🔁) — fuerza una relectura de la BD y reenvia el catalogo a todos los clientes.
- **Boton cerrar** (✕) — cierra el panel sin cambios.

---

## Pestana: Items

Listado de todos los consumibles registrados.

### Acciones rapidas

- 🔍 **Buscador** — filtra por nombre interno, etiqueta o categoria.
- ▶ **Vista previa** — ejecuta la progressbar + animacion + prop del item **sin aplicar efectos**. Util para afinar posiciones de props.
- 🗑 **Eliminar** — pide confirmacion y borra el item + todos sus efectos.
- ➕ **Nuevo item** — abre el editor en blanco.

### Editor de item

Campos principales:

| Campo | Descripcion |
|-------|-------------|
| **Nombre** | Nombre interno del item. Debe coincidir con el registrado en tu inventario (`bread`, `water`...). |
| **Etiqueta** | Nombre bonito usado en notificaciones. |
| **Categoria** | `food`, `drink`, `alcohol`, `drug`, `armor` o `custom`. Solo afecta al icono y el agrupamiento visual. |
| **Duracion (ms)** | Tiempo de la progressbar. |
| **Clave de texto** | Clave del string de la progressbar (`eat_progress`, `generic_progress`, `db::tu_clave`...). |
| **Cooldown (ms)** | Tiempo minimo entre dos usos del mismo item por el mismo jugador. |
| **Animacion** | Preset de animacion a reproducir. |
| **Prop** | Preset de prop a adjuntar al ped. |
| **Grupo requerido** | Si se rellena, solo ese grupo (o admins) puede consumir el item. Deja vacio para publico. |

Toggles:

- **Activo** — inhabilita el item sin borrarlo. Los jugadores veran una notificacion de error al intentar usarlo.
- **Eliminar al usar** — si esta desactivado, el item no se consume del inventario (util para items reutilizables como chalecos temporales).
- **Cancelable** — permite al jugador cancelar la progressbar.
- **Bloquear combate** — desactiva el combate durante la progressbar.
- **Bloquear movimiento** — desactiva el movimiento durante la progressbar.

### Cadena de efectos

Cada item tiene una lista de efectos que se ejecutan en el orden definido. Para cada efecto:

- **Selector con agrupador** — los efectos estan agrupados por categoria (needs, movement, combat, visual, economy, rp).
- **Badge `client` / `server` / `both`** — indica donde corre el efecto.
- **Reordenar** (⬆ ⬇) — cambia el orden de ejecucion.
- **Toggle** (👁 / 🚫) — deshabilita el efecto sin borrarlo.
- **Inputs auto-generados** — la UI deduce el tipo de input (numero con min/max, dropdown, checkbox, lista JSON) desde el schema del catalogo de efectos.

Al guardar, el backend hace un **DELETE + INSERT** de toda la cadena (la validacion se hace en el servidor), reinscribe el item como usable via `Bridge.RegisterUsableItem`, y reenvia el catalogo a todos los clientes.

---

## Pestana: Efectos

Vista de **solo lectura** del catalogo. Util para:

- Saber que efectos hay disponibles.
- Ver en que side corren (`client`, `server`, `both`).
- Ver la lista de parametros de cada efecto y sus valores por defecto.

Los efectos se definen en `shared/effects_catalog.lua`. Si anades efectos custom via `exports:AddEffectHandler`, anade tambien su entrada en el catalogo para que aparezcan en la UI.

---

## Pestana: Presets

Dos bloques: **Animaciones** y **Props**.

### Animaciones

| Campo | Descripcion |
|-------|-------------|
| **Nombre** | Identificador bonito (`eat_burger`, `smoke_meth`...). |
| **Dict** | Dict de animacion del juego (ej. `mp_player_inteat@burger`). |
| **Anim** | Nombre de la animacion (ej. `mp_player_int_eat_burger`). |
| **Flags** | Flags de la animacion (normalmente `49`). |

### Props

| Campo | Descripcion |
|-------|-------------|
| **Nombre** | Identificador bonito (`burger`, `water_bottle`...). |
| **Modelo** | Spawn name del prop. |
| **Hueso** | Id del hueso al que se ata (`60309` = mano derecha). |
| **Posicion X/Y/Z** | Offset relativo al hueso. |
| **Rotacion X/Y/Z** | Rotacion de la prop en grados. |

Si borras un preset, los items que lo usaban pierden su asignacion automaticamente (`SET NULL`) — no se quedan apuntando a un id invalido.

---

## Pestana: Idiomas

Editor de la tabla `nb_consumibles_locales`. Cada fila: `locale + key + value`.

- **Nueva fila** — anade una traduccion al vuelo.
- **Guardado inline** — cuando editas el valor y sales del input, se guarda automaticamente.
- **Buscador** — filtra por idioma, clave o valor.

Las claves de esta tabla **no** colisionan con las estaticas de `shared/locale.lua`. Si quieres sobrescribir una clave estatica, crea una fila con el mismo nombre — la BD gana.

Para referenciar una clave dinamica desde un item usa el prefijo `db::`:

```
progress_label_key = db::beber_del_grifo
```

---

## Pestana: Ajustes

Tabla clave/valor de `nb_consumibles_settings`. Ajustes disponibles:

| Clave | Efecto |
|-------|--------|
| `default_locale` | Sobrescribe `Config.Locale` al cargar el servidor. |

Puedes anadir tus propias claves y leerlas con:

```lua
local settings = Bridge.DB.GetSettings()
local mio = settings.mi_ajuste
```

---

## Hot reload y red

Cada operacion del panel hace:

1. Mutacion en BD (parametrizada, bajo `Bridge.IsAdmin`).
2. Relectura completa de items, efectos y presets.
3. Re-inscripcion de usable items via `Bridge.RegisterUsableItem`.
4. `TriggerClientEvent(-1, ...)` para refrescar el catalogo en todos los clientes.
5. Respuesta al panel del admin con el dataset actualizado (toast + listas re-renderizadas).

No necesitas `refresh` del inventario en frameworks que usan la tabla nativa (ESX/QB); si usas `ox_inventory`, el item **base** debe existir en `items.lua` — nb-consumibles solo gestiona el efecto, no la definicion del item en el inventario.
