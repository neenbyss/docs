# Changelog

Historial de versiones de **nb-restaurants**. Descarga las actualizaciones desde tu area de cliente en [neenbyss.tebex.io](https://neenbyss.tebex.io/).

Semver:

- **MAJOR** — cambios incompatibles.
- **MINOR** — funcionalidad nueva, backwards-compatible.
- **PATCH** — bugfixes.

---

## v2.0.0 — 2026-04-19

Release grande de pulido que convierte el MVP en un sistema de negocio completo. Todo lo nuevo es **opt-in** via `Config.Features.*` — el admin decide que se activa.

### Cocinar se siente cocinar

- **Pulsar "Cocinar"** reproduce progressbar + animacion + prop pegado a la mano (espátula en el grill, coctelera en el bar, sartén en la fryer). El servidor espera `duration_ms` antes de entregar.
- Notificaciones con **el icono real del item**, no solo un check verde.
- **Texto flotante 3D** sobre cada marker cuando te acercas ("🍳 Grill Station").

### Blips totalmente configurables y opcionales

Editor dentro de cada marker:

- **Opt-in marker a marker** — el admin decide cuales salen en el mapa.
- Sprite, color, escala, nombre y short-range con preview en vivo.
- Kill-switch global `Config.Features.Blips = false`.

### Dashboard de business intelligence

Nueva pestaña **Stats** en el boss menu:

- **Ingresos rodantes 7 dias**, transacciones, clientes unicos, gasto a proveedor.
- **Grafico de barras diario** de ingresos.
- **Top 5 items** mas vendidos.
- **Leaderboard de staff por propinas**.
- **Ultimas reseñas** de clientes.

Cada venta se registra en `nb_restaurants_sales_log`. Filas viejas se purgan automaticamente (`Config.Stats.RetentionDays`).

### Propinas + Reseñas

Despues de cualquier compra, modal que ofrece:

- **Propina** con presets + campo custom. Va directa al staff que atendio si esta online, o a la sociedad si no.
- **Rating 1-5 estrellas** con comentario opcional.

Toggles independientes (`Config.Features.Tips`, `Config.Features.Ratings`).

### Promociones / happy hour

Desde el boss menu:

- **Target**: todos los items (`*`) o uno concreto.
- **Descuento** 1%-100%.
- **Ventana**: absoluta (`start_at` / `end_at`) o recurrente (dia de la semana + franja horaria).
- **Cap** de promos activas concurrentes para evitar "todo gratis" accidental.

Se aplican solas al calcular precio en el autoservicio.

### Tabla de toggles

```lua
Config.Features = {
    Blips             = true,
    CraftingAnimation = true,
    FloatingText      = true,
    ToastIcons        = true,
    StatsDashboard    = true,
    Ratings           = true,
    Tips              = true,
    Promotions        = true,
    FormalOrders      = false,
}
```

Desactivar una feature oculta su UI, corta sus hooks de server y deja las filas de BD intactas. Puedes re-activar despues sin perder datos.

### Nuevas tablas

Ejecuta `[sql]/002_stats_tips_ratings_promos.sql`:

- `nb_restaurants_sales_log`
- `nb_restaurants_tips`
- `nb_restaurants_ratings`
- `nb_restaurants_promotions`

### Compatibilidad

100% compatible con v1.1.0. No hay APIs eliminadas ni columnas modificadas. Todo gated por `Config.Features.*`.

---

## v1.1.0 — 2026-04-19

### Imagenes de items en cocina y autoservicio

Las UIs de crafting y self-service ahora muestran los **iconos reales** de cada item (los mismos que ves en tu inventario):

- **Autoservicio**: grid con artwork grande + precio destacado + hover con lift y acento rojo. Los slots agotados se ven atenuados en vez de desaparecer.
- **Mesa de crafteo**: tile grande con el icono del plato + badge `×N` si produce mas de uno. Ingredientes como iconitos con badge de cantidad y tooltip.
- **Pixel-art nitido** con `image-rendering: crisp-edges`.
- **Fallback** a icono de paquete si la imagen no carga.

### Mejor soporte de inventarios

El resolver de imagenes respeta las mismas convenciones que el resto del ecosistema:

- **origen_inventory**: lee **`client.image` Y `image`** — el URL que pones al crear items con `AddCustomItem` se muestra sin config extra.
- **ox_inventory**: respeta `client.image`.
- **qb-inventory**: usa `image` si es URL completa.
- **Soporte para las dos versiones de origen_inventory** — origen se distribuye en dos layouts (v1 usa `html/images/`, v2 usa `ui/images/`). El default apunta a v2 (a partir de nb-bridge v1.2.2). Si usas origen v1, override en tu `Config.InventoryImagePaths` — ver [configuracion de nb-bridge](../nb-bridge/configuracion.md#origen-inventory-v1-vs-v2).

### Compatibilidad

100% compatible con v1.0.0. El imageMap es additive — clientes antiguos siguen viendo placeholders.

---

## v1.0.0 — 2026-04-19

Release inicial publica. Suite completa de gestion de restaurantes con paneles in-game, crafting, autoservicio, facturacion y lavado de dinero opcional.

### Features principales

- Panel admin y panel staff Vue 3 con auto-deteccion del rol por job.
- 6 tipos de marker (boss menu, billing desk, crafting station, self-service, warehouse, supplier).
- Biblioteca de 15 plantillas de recetas + importacion a restaurantes concretos.
- Boss menu in-game con hire/fire, sociedad (deposit/withdraw), toggle de autoservicio y lavado.
- Crafting con ingredientes mixtos (warehouse + personal) y cooldown.
- Self-service con stock finito o infinito, multi-payment (cash/bank/both), can-carry check y checkout atomico.
- Billing desk con 3 modos de cobro (native / nb-billings / external) y picker de clientes cercanos.
- Lavado de dinero opcional por restaurante, con fuente configurable (item / cash / account), cap por operacion, cooldown y Discord webhook opcional.
- 4 integraciones opt-in con auto-deteccion: nb-jobmanagers, nb-billings, nb-shops, nb-consumibles.
- i18n estatico (EN/ES) + dinamico en BD (`db::` prefix) editable desde el panel.
- Hot reload de catalogo (marcadores + restaurantes) sin reiniciar el recurso.
- Escrow-ready: `shared/`, `bridge/`, `integrations/` y `ui/**` abiertos; core cifrado.
