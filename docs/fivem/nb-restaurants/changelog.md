# Changelog

Historial de versiones de **nb-restaurants**. Descarga las actualizaciones desde tu area de cliente en [neenbyss.tebex.io](https://neenbyss.tebex.io/).

Semver:

- **MAJOR** — cambios incompatibles.
- **MINOR** — funcionalidad nueva, backwards-compatible.
- **PATCH** — bugfixes.

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
