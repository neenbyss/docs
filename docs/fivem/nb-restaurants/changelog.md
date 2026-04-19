# Changelog

Historial de versiones de **nb-restaurants**. Descarga las actualizaciones desde tu area de cliente en [neenbyss.tebex.io](https://neenbyss.tebex.io/).

Semver:

- **MAJOR** — cambios incompatibles.
- **MINOR** — funcionalidad nueva, backwards-compatible.
- **PATCH** — bugfixes.

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
