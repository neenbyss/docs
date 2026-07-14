# Configuracion

nb-bridge ofrece valores por defecto en `config.lua` y respeta los overrides del recurso consumidor.

---

## BridgeConfig (archivo `config.lua` de nb-bridge)

```lua
BridgeConfig = {
    -- Activar prints de debug de los modulos de nb-bridge
    Debug = false,

    -- Grupos de admin por defecto (usado por bridge.player.isAdmin cuando
    -- el consumidor no define su Config.AdminGroups)
    AdminGroups = { 'admin', 'superadmin', 'god' },

    -- Valores por defecto de stash (usado por bridge.inventory.registerStash
    -- cuando el consumidor no define su Config.Stash)
    Stash = {
        Slots = 50,
        MaxWeight = 100000,
    },

    -- Rutas NUI de iconos de inventario (usado por bridge.inventory.getImagePath)
    InventoryImagePaths = {
        ox_inventory      = 'nui://ox_inventory/web/images/%s.png',
        ['qb-inventory']  = 'nui://qb-inventory/html/images/%s.png',
        ['ps-inventory']  = 'nui://ps-inventory/html/images/%s.png',
        ['lj-inventory']  = 'nui://lj-inventory/html/images/%s.png',
        ['qs-inventory']  = 'nui://qs-inventory/html/images/%s.png',
        origen_inventory  = 'nui://origen_inventory/ui/images/%s.png',
    },

    -- Mapeo de grupos a nodos ACE, usado por bridge.player.getGroup en
    -- QBCore/QBX para resolver 'superadmin' y 'mod' ademas de 'admin'
    -- (ESX no lo necesita, usa xPlayer.getGroup() directamente)
    GroupMap = {
        god        = 'group.god',
        superadmin = 'group.superadmin',
        admin      = 'group.admin',
        mod        = 'group.mod',
    },

    -- Sinks de auditoria usados por bridge.log.createLog (v2.2.0).
    -- Orden de despacho: qb-log (si esta corriendo) -> webhook de Discord
    -- de abajo -> Debugger como ultimo recurso.
    -- SEGURIDAD: nunca commitees una URL de webhook real aca. Dejala vacia
    -- y configurala por servidor desde el Config.Logs del consumidor, o
    -- leela de una convar. Publicar un webhook real es una fuga de credenciales.
    Logs = {
        DefaultColor = 3447003, -- color del embed de Discord (decimal); usado en los webhooks
        QbLogColor   = 'default', -- nombre de color pasado a qb-log:server:CreateLog
        Webhooks = {
            default = '',        -- webhook de fallback para todas las categorias (vacio = deshabilitado)
            -- Overrides por categoria, ej.:
            -- banking = '',
            -- admin   = '',
        },
    },
}
```

---

### origen_inventory: v1 vs v2

origen_inventory se distribuye en **dos layouts NUI distintos** y no hay forma fiable de autodetectar cual esta corriendo un servidor:

| Version | Ruta de iconos |
|---------|----------------|
| **v2** (actual, default) | `nui://origen_inventory/ui/images/%s.png` |
| **v1** (legacy) | `nui://origen_inventory/html/images/%s.png` |

El default del bridge apunta a **v2**. Si tu servidor usa **v1** y ves iconos rotos, overridea la ruta desde tu script consumidor:

```lua
-- shared/config.lua del consumidor (nb-restaurants, nb-shops, etc.)
Config.InventoryImagePaths = {
    origen_inventory = 'nui://origen_inventory/html/images/%s.png',
}
```

El override se aplica tambien a `bridge.inventory.getImagePath()` por la cascada de configuracion.

---

## Cascada de configuracion

La mayoria de funciones de Bridge leen primero el `Config` del **consumidor** y, si no existe la clave, caen al `BridgeConfig`.

| Funcion | Lee primero | Cae a |
|---------|-------------|-------|
| `bridge.player.isAdmin()` | `Config.AdminGroups` | `BridgeConfig.AdminGroups` |
| `bridge.player.getGroup()` (QBCore/QBX) | `Config.GroupMap` | `BridgeConfig.GroupMap` |
| `bridge.inventory.registerStash()` | `Config.Stash` | `BridgeConfig.Stash` |
| `bridge.inventory.getImagePath()` | `Config.InventoryImagePaths` | `BridgeConfig.InventoryImagePaths` |
| `Debugger()` | `Config.Debug` | `BridgeConfig.Debug` |

Ejemplo desde un consumidor:

```lua
-- shared/config.lua del recurso consumidor
Config = {
    Debug = true,
    AdminGroups = { 'admin', 'superadmin', 'god', 'owner' },
}
```

Con ese `Config` activo, `bridge.player.isAdmin(src)` usara `{ 'admin', 'superadmin', 'god', 'owner' }` en lugar de la lista de `BridgeConfig`.

> **Nota:** `bridge.log.createLog()` NO participa de esta cascada de `Debug` — es un registro de auditoria de produccion y se ejecuta siempre, independientemente de `Config.Debug` / `BridgeConfig.Debug`. Su unica configuracion es `BridgeConfig.Logs`.

---

## Debug

Activar `Debug = true` imprime logs de todos los modulos:

```
[nb-bridge][SERVER][Framework] detected: QBX (qbx_core)
[nb-bridge][SERVER][Inventory] Detected: ox_inventory
[nb-bridge][SERVER][Inventory] RegisterStash: evidence_locker | label: Evidence | job: police
```

Puedes activarlo:

- En el `BridgeConfig.Debug` de nb-bridge (afecta a todo).
- En el `Config.Debug` del consumidor (afecta a tu script).

---

## Configuracion por modulo

No hay archivos `.cfg` por modulo — todo vive en `config.lua`. Si necesitas algo muy especifico, la via limpia es usar [overrides](overrides.md).
