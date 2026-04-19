# Configuracion

La mayor parte de la config de **tiendas** vive en la BD (editable desde el panel). `shared/config.lua` solo define defaults globales: permisos, comandos, markers, blips y peds.

---

## General

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos con acceso al panel | `{ 'admin', 'superadmin', 'god' }` |
| `Config.Debug` | Prints de debug | `false` |
| `Config.Locale` | Idioma (`'en'` / `'es'`) | `'en'` |
| `Config.Command` | Comando para abrir el panel | `'shops'` |

---

## Markers

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Marker.DrawDistance` | Distancia a la que se dibuja | `15.0` |
| `Config.Marker.InteractionDistance` | Distancia para poder pulsar E | `2.0` |
| `Config.Marker.InteractionKey` | Keycode de interaccion (38 = E) | `38` |
| `Config.Marker.Type` | Tipo de marker GTA | `27` |
| `Config.Marker.Scale` | Escala XYZ | `vector3(1.0, 1.0, 0.5)` |
| `Config.Marker.BobUpAndDown` | Animacion de subida/bajada | `true` |
| `Config.Marker.Rotate` | Giro continuo | `true` |
| `Config.Marker.Colors.civil` | Color RGBA para tiendas civil | Azul |
| `Config.Marker.Colors.business` | Color RGBA para business | Verde |
| `Config.Marker.Colors.illegal` | Color RGBA para illegal | Rojo |

> El motor usa dos hilos: uno de scan (cada 500-1000ms) para detectar tiendas cercanas, y uno de draw (0ms) solo cuando hay tiendas visibles. Asi se evita gastar CPU en mapas sin tiendas alrededor.

---

## Blips

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Blip.Default.sprite` | Sprite por defecto (id GTA) | `52` |
| `Config.Blip.Default.scale` | Escala | `0.8` |
| `Config.Blip.Colors.civil` / `.business` / `.illegal` | Color por tipo | `0 / 2 / 1` |

Cada tienda puede sobrescribir sprite/color/scale desde el wizard.

---

## Peds

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Ped.DefaultModel` | Modelo cuando la tienda no define uno | `'s_m_m_shopkeep_01'` |
| `Config.Ped.DefaultScenario` | Escenario (animacion) | `'WORLD_HUMAN_STAND_IMPATIENT'` |

---

## Iconos de items

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.ItemImages` | Patron de URL NUI para iconos | `'nui://ox_inventory/web/images/%s.png'` |

Si tu inventario no es ox, ajustalo:

```lua
Config.ItemImages = 'nui://qb-inventory/html/images/%s.png'
Config.ItemImages = 'nui://qs-inventory/html/images/%s.png'
Config.ItemImages = 'nui://origen_inventory/ui/images/%s.png'
```

Ademas, cada item puede sobrescribir su icono desde `itemData.client.image` en tu definicion de inventario — nb-shops lo respetara.

---

## Idiomas

Los textos viven en `shared/locale.lua`. Para anadir un idioma:

1. Duplica el bloque `Locale['es']` a `Locale['xx']` y traduce.
2. `Config.Locale = 'xx'`.

---

## Metodos de pago

`cash`, `bank` o el nombre de **cualquier item** del inventario (`gold_coin`, `black_money`, etc.). Se configura por tienda desde el wizard:

```
payment_methods: ['cash', 'bank', 'gold_coin']
```

Cuando un item usa otro item como moneda se definen dos campos por item:

- `price_type` — el nombre del item que hace de moneda.
- `item_price_amount` — la cantidad requerida por unidad comprada.

---

## Acceso por job / gang

- **Civil** — todo el mundo puede comprar.
- **Business** — array de jobs (`['mechanic']`) permitidos. Validado en cliente (ocultar el marker) y en servidor (bloquear compra).
- **Illegal** — array de gangs (`['vagos', 'ballas']`). Validacion igual que business.

> Los checks **client-side** son solo para UX (no mostrar lo que no puedes usar). La autoridad esta siempre en el servidor.
