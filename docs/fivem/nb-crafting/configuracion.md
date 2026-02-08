# Configuración

Todo se edita en **config.lua** en la raíz del recurso.

---

## Opciones principales

| Dónde | Qué |
|-------|-----|
| **Config.Framework** | `'ESX'` o `'QBCore'` |
| **Config.Inventory** | `'ox_inventory'`, `'qb-inventory'`, `'origen_inventory'`, `'ps-inventory'`, `'lj-inventory'`, `'qs-inventory'` |
| **Config.Permissions.adminGroup** | Grupo que puede usar el panel admin (ej. `'admin'`) |
| **Config.Interaction** | `key` (tecla, 38 = E), `distance`, `useMarker`, `markerType`, `markerSize`, `markerColor`, `markerBob` |
| **Config.Crafting** | `freezePlayer`, `useAnimation`, `animDict`, `animName`, `animFlag` |
| **Config.LevelSystem.enabled** | `true` = niveles/XP activos, `false` = sin niveles |
| **Config.UI** | `primaryColor`, `successColor`, `dangerColor` (hex) |
| **Config.Locale** | Idioma activo: `'es'`, `'en'`, etc. |
| **Config.Locales** | Tabla de idiomas; cada uno es una tabla de claves → texto. |

---

## Textos e idiomas

Los mensajes del juego están en **Config.Locales**. La clave activa es **Config.Locale**.

Para **añadir otro idioma** (ej. inglés): crea `Config.Locales['en']` con las mismas claves que tiene `['es']` y pon `Config.Locale = 'en'`. Las claves que usa el script son: `press_to_open`, `crafting`, `success`, `cancelled`, `error_materials`, `error_money`, `error_inventory`, `admin_menu`, `station_created`, `station_updated`, `item_added`, `no_perms`, `level_up`, `level_required`, `added_to_queue`, `items_collected`, `item_collected`, `no_items_to_collect`, `cannot_collect`, `queue_item_cancelled`, `config_saved`, `craft_complete`, `too_far`, `job_required`. Puedes usar `~g~`, `~r~`, `~w~` etc. para colores en los textos.
