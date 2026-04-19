# Configuracion

Casi todo se configura **desde el panel** (ver [Panel in-game](panel.md)). En este documento se listan solo las opciones globales que viven en `shared/config.lua`.

---

## shared/config.lua

### Permisos

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos con acceso al panel y al CRUD | `{ 'admin', 'superadmin', 'god' }` |

Los grupos se validan via `Bridge.IsAdmin(src)` de nb-bridge.

### General

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Debug` | Activa `Debugger(...)` | `false` |
| `Config.Locale` | Idioma por defecto del servidor (`'es'`, `'en'` o el que anadas) | `'es'` |
| `Config.PanelCommand` | Comando para abrir el panel | `'consumibles'` |
| `Config.PanelKeybind` | `RegisterKeyMapping` opcional (`nil` = sin keybind) | `nil` |

### Needs (hambre / sed)

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Needs.AutoScale` | Escala automaticamente los valores 0..100 al rango del framework | `true` |
| `Config.Needs.QBMax` | Escala QBCore | `100` |
| `Config.Needs.ESXMax` | Escala ESX (x10000) | `1000000` |

Con `AutoScale = true` puedes definir `hunger: { amount: 25 }` en el panel y nb-consumibles se encarga de convertirlo a `250000` en ESX y `25` en QBCore.

### Inventario

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Inventory.ShowRemoveHud` | Muestra `qb-inventory:client:ItemBox` al consumir | `true` |

### Cooldown global

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Cooldown.GlobalMs` | Tiempo minimo entre dos consumos de cualquier item (anti-spam) | `1500` |

Cada item ademas puede tener su propio `cooldown_ms` configurado en el panel — se aplica el **mayor** de los dos.

### Hot reload

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.HotReload` | Propaga los cambios del panel a todos los clientes sin reinicio | `true` |

### Defaults de animacion / progreso

Usados cuando un item no tiene preset asignado:

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.DefaultEatAnim` | `{ dict, anim, flags }` fallback para comer | mp_player_inteat@burger |
| `Config.DefaultDrinkAnim` | `{ dict, anim, flags }` fallback para beber | mp_player_intdrink |
| `Config.DefaultProgressMs` | Duracion en ms de la progressbar si el item no define una | `5000` |

### Idiomas de la interfaz

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.PanelLanguages` | Idiomas que el selector del panel ofrece | `{ 'es', 'en' }` |

Para anadir un idioma nuevo:

1. Anade la clave en `Config.PanelLanguages`.
2. Anade el bloque `Locales['xx']` en `shared/locale.lua` con las mismas claves que `es` y `en`.
3. Anade las traducciones de UI en `ui/js/i18n.js` bajo `I18N.xx`.

---

## shared/locale.lua

Los textos de notificaciones/progressbars estan en `Locales['es']` y `Locales['en']`. El **panel** sobrescribe estos en runtime con las filas de la tabla `nb_consumibles_locales`.

Desde un item del panel puedes referenciar una clave dinamica asi:

- `progress_label_key = 'db::mi_item_progreso'` — leera la fila `{ locale: 'es', key: 'mi_item_progreso' }` de la BD.
- `progress_label_key = 'eat_progress'` — leera la clave estatica de `shared/locale.lua`.

---

## Cascada de configuracion con nb-bridge

nb-consumibles define `Config.AdminGroups`, por lo que tiene prioridad sobre `BridgeConfig.AdminGroups`. Lo mismo aplica a cualquier otra opcion soportada por nb-bridge.
