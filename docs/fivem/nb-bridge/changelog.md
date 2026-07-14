# Changelog

Historial de cambios de nb-bridge. Las releases viven en **[github.com/neenbyss/nb-bridge/releases](https://github.com/neenbyss/nb-bridge/releases)**.

Este proyecto sigue [semver](https://semver.org/):

- **MAJOR** — cambios incompatibles (ej. renombrar un export).
- **MINOR** — funcionalidad nueva, backwards-compatible.
- **PATCH** — bugfixes, backwards-compatible.

---

## v2.2.0 — 2026-07-13

### Anadido

- **`bridge.log.*`** — auditoria server-side (`bridge.log.createLog(category, title, message, data, mention)`). Despacha al pipeline existente del servidor: **qb-log** (QBCore/QBX) si esta corriendo → un **webhook de Discord** configurado en `BridgeConfig.Logs.Webhooks` (funciona tambien en ESX, que no tiene un recurso de logs nativo) → fallback a `Debugger`. Es un registro de auditoria de produccion y **no** esta gateado por `Debug`.
- **`bridge.player.onMoneyChanged(cb)`** — hook server que dispara cuando el dinero de un jugador cambia, venga del bridge o de cualquier script de terceros que llame directamente las funciones de dinero del framework (eventos nativos: `esx:addAccountMoney` / `removeAccountMoney` / `setAccountMoney`, `QBCore:Server:OnMoneyChange`). `cb(source, moneyType, amount, newBalance, changeSource)` — `newBalance` es el saldo posterior al cambio, autoritativo.
- **`bridge.ui.*`** — hooks de ciclo de vida de UI en cliente (`beforeOpening`, `afterClosing`, `beforeAction`, `afterAction`) para que los recursos con menus nunca tengan que ramificar por framework en su codigo de apertura/cierre. Bloquea el inventario via statebag y cierra ox_inventory al abrir; libera al cerrar. Redefinible desde la carpeta `overrides/`.
- **`BridgeConfig.Logs`** — configuracion de los sinks de logging (`DefaultColor`, `QbLogColor`, `Webhooks`). Se distribuye con webhook vacio por defecto — nunca commitees una URL real.

### Notas

- Todos los añadidos son compatibles hacia atras con v2.1.0 — sin renombrados ni cambios de firma.

---

## v2.1.0 — 2026-06-21

### Anadido

- **`bridge.event.*`** — namespace unificado de hooks de ciclo de vida para los tres frameworks:
    - **Server:** `onPlayerLoaded`, `onPlayerUnloaded`, `onResourceStart`, `onResourceStop`, `onSelfStart`, `onSelfStop`.
    - **Client:** `onPlayerLoaded`, `onPlayerUnloaded`, `onPlayerSpawned`, `onResourceStart`, `onResourceStop`.
- **`bridge.diagnostics()`** — snapshot de runtime en el server (framework, sistema de inventario, feature flags, dependencias faltantes, uptime).
- **Comando `/nbdiag`** — admin y consola; imprime el diagnostico en la consola del servidor y en el chat de admin.
- **`exports['nb-bridge']:diagnostics()`** — export directo para llamar diagnostics desde otros recursos sin pasar por `get()`.
- **Type stubs LuaLS/EmmyLua** (`types/nb-bridge.lua` + `.luarc.json`) — autocompletado y chequeo de tipos para los namespaces del bridge.
- **CI con GitHub Actions** — lint con luacheck en cada push/PR; validacion de version de fxmanifest en tag pushes.

### Arreglado

- CI: agregadas natives de FiveM faltantes en los globals de luacheck.
- CI: fijado `actions/checkout@v4.2.2` para silenciar el warning de deprecacion de Node.js 20.

---

## v2.0.0 — 2026-06-21

### Cambios incompatibles (BREAKING)

- **API namespaced** — se eliminan todos los metodos planos `Bridge.Fn`. Nueva API:
    - `Bridge.player.*` — gestion de jugadores, dinero, trabajos, gangs, permisos.
    - `Bridge.inventory.*` — items, stashes, items usables.
    - `Bridge.vehicle.*` — spawn de vehiculos, propiedades, matriculas.
    - `Bridge.notify.*` — notificaciones (server + client).
    - `Bridge.callback.*` — callbacks de servidor.
    - `Bridge.license.*` — licencias de conducir/armas.
    - `Bridge.progress.*` — barras de progreso.
- **Cambia la API de consumo** — los scripts ahora deben usar `local bridge = exports['nb-bridge']:get()` en lugar de `shared_scripts { '@nb-bridge/loader.lua' }`.
- **Sin alias de compatibilidad hacia atras** — los nombres de metodos de v1.x desaparecen por completo.
- **Se eliminan todos los exports por metodo** — ya no existe `exports['nb-bridge']:GetJob(...)` ni similares. El unico export es `get()`.
- **Los nombres de metodo ahora son camelCase** — `addMoney`, `getJob`, `spawnVehicle`, etc.

### Anadido

- **Soporte para QBX (qbx_core)** — compatibilidad completa como tercer framework. Orden de deteccion: QBX → ESX → QBCore (evita falsos positivos por el compat-shim `qb-core` que expone QBX).
- `Bridge.player.setGang(source, gangName, grade)` — asigna gang al jugador (QBCore/QBX).
- `Bridge.player.onPlayerUnloaded(cb)` — hook de evento server + client para los tres frameworks.
- `Bridge.player.getAllPlayers()` — retorna array de source IDs online.
- `Bridge.player.registerCommand(name, group, cb, suggestion)` — registro unificado de comandos con gating ACE en ESX / QBCore / QBX.
- `Bridge.inventory.getItemMetadata(source, itemName)` — metadata por item (solo ox_inventory).
- `Bridge.player.getGroup` ahora retorna tambien `'superadmin'` y `'mod'` en QBCore/QBX via chequeo de permisos ACE. Configurable via `BridgeConfig.GroupMap`.

### Arreglado

- `Bridge.player.removeMoney` (ESX) — pre-chequea el balance del jugador; ahora retorna `false` real cuando faltan fondos en lugar de retornar siempre `true`.
- `Bridge.inventory.registerUsableItem` — en `origen_inventory`, registra SOLO via origen, eliminando el doble disparo que ocurria cuando origen y el framework notificaban a la vez.
- `Bridge.vehicle.spawnVehicle` — usa un deadline de wall-clock con `GetGameTimer()` en lugar de un contador de frames, evitando timeouts prematuros bajo carga del servidor.
- `Bridge.inventory.*` — se agrego el fallback sincrono de `ResolveInventorySystem()`; los llamados tempranos (antes del hilo de deteccion de 500ms) ya no fallan silenciosamente.
- `Bridge.inventory.forceOpenPlayerInventory` (qs-inventory) — usa el export de servidor cuando esta disponible; cae al evento de cliente correcto sin el `{}` inicial espurio.
- `Bridge.callback.trigger` — timeout de limpieza de 15 segundos; los callbacks pendientes ahora se liberan con `cb(nil)` si el servidor nunca responde, evitando memory leaks.

---

## v1.2.2 — 2026-04-19

### Cambiado

- **Ruta por defecto de `origen_inventory` vuelve a `ui/images/`** (rama **v2** de origen_inventory, la actual).

origen_inventory se distribuye en dos layouts NUI:

- **v2** (actual) — `ui/images/`
- **v1** (legacy) — `html/images/`

No hay forma fiable de autodetectar que rama corre un servidor, asi que tomamos **v2 como default**. Si usas la v1, override desde tu script consumidor:

```lua
Config.InventoryImagePaths = {
    origen_inventory = 'nui://origen_inventory/html/images/%s.png',
}
```

v1.2.1 asumio una unica ruta canonica; v1.2.2 corrige esa asuncion.

---

## v1.2.1 — 2026-04-19

### Arreglado

- ⚠️ **Obsoleto por v1.2.2**. Cambio `ui/images/` → `html/images/` para origen_inventory pensando que era la ruta canonica; resulto estar ligado a la rama v1 solamente. Si tu servidor usa origen v1, esta version funciona; para v2 (la mas comun) sube a v1.2.2+.

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
