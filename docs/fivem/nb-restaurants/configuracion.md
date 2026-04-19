# Configuracion

Toda la config global vive en `shared/config.lua`. Lo relacionado con cada **restaurante** (nombre, logo, marcadores, recetas) se configura desde el panel in-game.

---

## General

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos con acceso al panel admin | `{ 'admin', 'superadmin', 'god' }` |
| `Config.Debug` | Prints de debug | `false` |
| `Config.Locale` | Idioma por defecto | `'es'` |
| `Config.PanelCommand` | Comando para abrir el panel | `'restaurants'` |
| `Config.PanelKeybind` | RegisterKeyMapping opcional | `nil` |
| `Config.DefaultLogo` | URL del logo cuando un restaurante no tiene uno | `'https://i.imgur.com/5gJp0Gk.png'` |
| `Config.PanelLanguages` | Idiomas del selector del panel | `{ 'es', 'en' }` |

---

## Integraciones

| Opcion | Valores | Descripcion |
|--------|---------|-------------|
| `Config.Integrations.JobManagers` | `'auto'` / `'off'` / `'force'` | Delegar grades + sociedad a nb-jobmanagers. |
| `Config.Integrations.Billings` | `'auto'` / `'off'` / `'force'` | Usar nb-billings para `billing_mode = nb-billings`. |
| `Config.Integrations.Shops` | `'auto'` / `'off'` / `'force'` | Permitir supplier markers apuntando a shops de nb-shops. |
| `Config.Integrations.Consumibles` | `'auto'` / `'off'` / `'force'` | Registrar items cocinados con `effects` en nb-consumibles. |

- **`auto`** — si el recurso esta arrancado, se usa; si no, fallback.
- **`off`** — nunca se usa aunque este arrancado.
- **`force`** — error en consola si no esta arrancado.

---

## Markers

| Opcion | Default |
|--------|---------|
| `Config.Marker.DrawDistance` | `15.0` |
| `Config.Marker.InteractionDistance` | `2.0` |
| `Config.Marker.InteractionKey` | `38` (E) |
| `Config.Marker.Type` | `27` |
| `Config.Marker.Scale` | `vector3(1.0, 1.0, 0.5)` |
| `Config.Marker.Colors` | Por kind — `boss_menu` ambar, `crafting_station` indigo, `self_service` azul, etc. |

---

## Almacen (warehouse)

| Opcion | Default | Descripcion |
|--------|--------:|-------------|
| `Config.Warehouse.Slots` | `50` | Slots cuando el marker no define un override. |
| `Config.Warehouse.MaxWeight` | `200000` | Peso maximo (gramos). |

Cada marker `warehouse` puede sobrescribir `slots` y `max_weight` en su `params`.

---

## Crafting

| Opcion | Default | Descripcion |
|--------|--------:|-------------|
| `Config.Crafting.GlobalCooldownMs` | `1000` | Entre dos crafts del mismo jugador. |
| `Config.Crafting.Engine` | `'native'` | Motor de skillcheck para recetas que lo tienen (`'native'`, `'ox_lib'`, `'none'`). |

---

## Self-service

| Opcion | Default | Descripcion |
|--------|--------:|-------------|
| `Config.SelfService.GlobalCooldownMs` | `1500` | Anti-spam por cliente. |
| `Config.SelfService.ShowEmptySlots` | `true` | Slots agotados visibles como "Sin stock". |

---

## Billing

| Opcion | Default | Descripcion |
|--------|--------:|-------------|
| `Config.Billing.MaxAmount` | `500000` | Importe maximo por emision. |
| `Config.Billing.NearbyRadius` | `10.0` | Radio para el picker de clientes cercanos. |

---

## Lavado de dinero

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Config.Laundering.DirtyMoneySource` | De donde sale el dinero sucio | `'item'` (`'cash'` / `'account'`) |
| `Config.Laundering.DirtyMoneyItem` | Nombre del item a consumir | `'dirty_money'` |
| `Config.Laundering.DirtyMoneyAccount` | Cuenta alterna | `'black_money'` |
| `Config.Laundering.MinIntervalSeconds` | Cooldown entre lavados | `60` |
| `Config.Laundering.MaxPerOperation` | Tope por operacion | `100000` |
| `Config.Laundering.DiscordWebhook` | Webhook opcional para auditoria | `''` |

El admin habilita el lavado **por restaurante** desde el panel (campo `allow_laundering`). El tax (%) tambien se configura por restaurante.

---

## Tipos de estacion por defecto

`Config.DefaultStationTypes` — sugerencias mostradas en la UI. Puedes poner cualquier string, no esta cerrado a la lista:

```lua
Config.DefaultStationTypes = { 'grill', 'fryer', 'mixer', 'oven', 'cold', 'bar' }
```

---

## Hot reload

`Config.HotReload = true` — los cambios del panel se propagan a todos los clientes al instante sin reiniciar el recurso.
