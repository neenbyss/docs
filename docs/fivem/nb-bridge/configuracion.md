# Configuracion

nb-bridge ofrece valores por defecto en `config.lua` y respeta los overrides del recurso consumidor.

---

## BridgeConfig (archivo `config.lua` de nb-bridge)

```lua
BridgeConfig = {
    -- Activar prints de debug de los modulos de nb-bridge
    Debug = false,

    -- Grupos de admin por defecto (usado por Bridge.IsAdmin cuando el
    -- consumidor no define su Config.AdminGroups)
    AdminGroups = { 'admin', 'superadmin', 'god' },

    -- Valores por defecto de stash (usado por Bridge.RegisterStash
    -- cuando el consumidor no define su Config.Stash)
    Stash = {
        Slots = 50,
        MaxWeight = 100000,
    },

    -- Rutas NUI de iconos de inventario (usado por Bridge.GetImagePath)
    InventoryImagePaths = {
        ox_inventory      = 'nui://ox_inventory/web/images/%s.png',
        ['qb-inventory']  = 'nui://qb-inventory/html/images/%s.png',
        ['ps-inventory']  = 'nui://ps-inventory/html/images/%s.png',
        ['lj-inventory']  = 'nui://lj-inventory/html/images/%s.png',
        ['qs-inventory']  = 'nui://qs-inventory/html/images/%s.png',
        origen_inventory  = 'nui://origen_inventory/ui/images/%s.png',
    },
}
```

---

## Cascada de configuracion

La mayoria de funciones de Bridge leen primero el `Config` del **consumidor** y, si no existe la clave, caen al `BridgeConfig`.

| Funcion | Lee primero | Cae a |
|---------|-------------|-------|
| `Bridge.IsAdmin()` | `Config.AdminGroups` | `BridgeConfig.AdminGroups` |
| `Bridge.RegisterStash()` | `Config.Stash` | `BridgeConfig.Stash` |
| `Debugger()` | `Config.Debug` | `BridgeConfig.Debug` |
| `Bridge.GetImagePath()` | N/A | `BridgeConfig.InventoryImagePaths` |

Ejemplo desde un consumidor:

```lua
-- shared/config.lua del recurso consumidor
Config = {
    Debug = true,
    AdminGroups = { 'admin', 'superadmin', 'god', 'owner' },
}
```

Con ese `Config` activo, `Bridge.IsAdmin(src)` usara `{ 'admin', 'superadmin', 'god', 'owner' }` en lugar de la lista de `BridgeConfig`.

---

## Debug

Activar `Debug = true` imprime logs de todos los modulos:

```
[nb-bridge][Framework] detected: ESX
[nb-bridge][Inventory] Detected: ox_inventory
[nb-bridge][Inventory] RegisterStash: evidence_locker | label: Evidence | job: police
```

Puedes activarlo:

- En el `BridgeConfig.Debug` de nb-bridge (afecta a todo).
- En el `Config.Debug` del consumidor (afecta a tu script).

---

## Configuracion por modulo

No hay archivos `.cfg` por modulo — todo vive en `config.lua`. Si necesitas algo muy especifico, la via limpia es usar [overrides](overrides.md).
