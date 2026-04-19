# Changelog

Historial de cambios de nb-bridge. Las releases viven en **[github.com/neenbyss/nb-bridge/releases](https://github.com/neenbyss/nb-bridge/releases)**.

Este proyecto sigue [semver](https://semver.org/):

- **MAJOR** — cambios incompatibles (ej. renombrar un export).
- **MINOR** — funcionalidad nueva, backwards-compatible.
- **PATCH** — bugfixes, backwards-compatible.

---

## v1.2.1 — 2026-04-19

### Arreglado

- Ruta por defecto de `origen_inventory` en `BridgeConfig.InventoryImagePaths` corregida de `ui/images/` a `html/images/` — los iconos de items ahora se resuelven correctamente en servidores que usan origen_inventory sin necesidad de override manual.

Fix cosmetico. Los consumidores (nb-consumibles, nb-shops, nb-restaurants) se benefician inmediatamente.

---

## v1.2.0 — 2026-04-19

### Anadido

- **Soporte para `origen_inventory`** en todo el modulo inventory. Deteccion automatica (`Bridge.InventorySystem == 'origen_inventory'`).
- `Bridge.AddItem` / `RemoveItem` / `HasItem` / `CanCarry` → routean a `exports.origen_inventory:addItem` / `removeItem` / `getItemCount` / `canCarryItem`.
- `Bridge.RegisterStash` → usa `exports.origen_inventory:registerStash(id, { label, slots, weight })`.
- `Bridge.ForceOpenStash` y `Bridge.ForceOpenPlayerInventory` → origen abre desde el cliente, el bridge hace el relay via el evento `nb-bridge:client:origenOpenInventory`.
- `Bridge.GetAllItems` → lee items en runtime con `exports.origen_inventory:Items()`.
- Client: `Bridge.OpenStash`, `Bridge.OpenPlayerInventory` y `Bridge.GetItemCount` (via `Search('count', item)`).
- `Bridge.RegisterUsableItem` → ademas del framework, cuando origen esta activo tambien se inscribe via `exports.origen_inventory:CreateUseableItem` para que la accion "usar" dispare pase lo que pase.

### Compatibilidad

- 100% compatible con v1.1.0 — sin renombrados ni firmas cambiadas.
- Opt-in: la integracion solo se activa si `origen_inventory` esta arrancado.

---

## v1.1.0 — 2026-04-19

### Anadido

- **`Bridge.RegisterUsableItem(itemName, handler)`** — abstraccion unificada de `ESX.RegisterUsableItem` y `QBCore.Functions.CreateUseableItem`. El callback recibe `(source, { name, slot })` en cualquier framework.
- **`Bridge.IsUsableItemRegistered(itemName)`** — consulta si nb-bridge ya inscribio un item usable.
- **Soporte para `qs-inventory`** — se suma a ox_inventory, qb-inventory y los defaults de framework en toda la suite del modulo `inventory` (`AddItem`, `RemoveItem`, `HasItem`, `CanCarry`, `RegisterStash`, `ForceOpenStash`).

### Compatibilidad

- 100% compatible con consumidores de v1.0.0 — no hay renombrados ni firmas modificadas.
- `Bridge.InventorySystem` ahora puede valer `'qs-inventory'` ademas de los anteriores.

### Recursos que se benefician

- [nb-consumibles](../nb-consumibles/index.md) — requiere `Bridge.RegisterUsableItem`, por lo que depende directamente de v1.1.0+.
- Cualquier recurso que detecte el inventario: ahora soportan servidores con qs-inventory sin custom overrides.

---

## v1.0.0 — 2026-04-03

Release inicial publica.

### Anadido

- **Deteccion automatica de framework** — ESX Legacy y QBCore detectados al arranque, expuestos en `Bridge.Framework` y `Bridge.FrameworkObject`.
- **Modulo framework** — API unificada para jugadores, permisos, dinero, trabajo, gang, metadata, playtime, teleport, billing.
- **Modulo inventory** — abstraccion de ox_inventory, qb-inventory y defaults del framework para items y stashes.
- **Modulo notify** — `Bridge.Notify` / `Bridge.ShowNotification` con auto-deteccion de ox_lib, ESX, QBCore y native.
- **Modulo vehicle** — matriculas, spawn, propiedades y alta en BD.
- **Modulo callbacks** — `CreateCallback` / `TriggerServerCallback` sin exports manuales.
- **Modulo licenses** — identidad + licencias (driver, weapon) con auto-deteccion de bcs, okok, esx_license y metadata de QBCore.
- **Modulo progress** — barra de progreso con fallback nativo o ox_lib si esta presente.
- **Cascada de configuracion** — `Config` del consumidor manda; si falta una clave, cae a `BridgeConfig`.
- **Carpeta `overrides/`** — customiza cualquier funcion de `Bridge` sin editar archivos base.
- **Exports FiveM** — toda la API publicada tambien como `exports['nb-bridge']:FuncName`.

### Notas

- Todos los recursos `nb-*` pasan a depender de este paquete, eliminando el `bridge/` duplicado en cada uno.

---

## Como actualizar

1. Parar el servidor o solo nb-bridge.
2. Reemplazar la carpeta `nb-bridge` por la nueva version (descargada desde [releases](https://github.com/neenbyss/nb-bridge/releases)).
3. Leer las notas de la release por si hay breaking changes (solo en versiones **MAJOR**).
4. Arrancar de nuevo.

Una actualizacion minor/patch es transparente para todos los consumidores.
