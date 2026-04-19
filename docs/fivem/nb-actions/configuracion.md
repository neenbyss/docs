# Configuracion

Todo en `shared/config.lua`.

---

## General

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos que cuentan como admin | `{ 'admin', 'superadmin', 'god' }` |
| `Config.AdminBypass` | Los admins tienen todas las acciones | `true` |
| `Config.Debug` | Prints de debug en consola + F8 | `false` |
| `Config.Locale` | Idioma (`'en'` / `'es'`) | `'es'` |

---

## Input

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Keybind` | Tecla para abrir el panel | `'F6'` |
| `Config.KeybindDescription` | Texto en `Settings -> Keybinds` | `'Abrir Menu de Acciones'` |
| `Config.MaxPlayerDistance` | Radio de deteccion de jugadores (m) | `3.0` |
| `Config.MaxVehicleDistance` | Radio de deteccion de vehiculos (m) | `5.0` |

---

## Permisos

Tres mecanismos en paralelo. Un jugador pasa si **alguna** de las tres lo permite:

1. **`Config.AdminBypass`** + pertenecer a `Config.AdminGroups`.
2. **`Config.AllowedGroups`** — por job/gang + grado (ver abajo).
3. **`Config.AllowedIdentifiers`** — array de licenses/citizenIds que saltan toda validacion.

```lua
Config.AllowedGroups = {
    police = {
        grades  = { 0, 1, 2, 3, 4 },   -- true = todos los grados
        actions = { 'handcuff', 'drag', 'putInVehicle', 'takeFromVehicle',
                    'checkIdentity', 'checkDriverLicense', 'checkWeaponLicense',
                    'checkVehicleOwner', 'searchPlayer', 'searchDeadPlayer' },
    },
    ambulance = {
        grades  = { 0, 1, 2 },
        actions = { 'checkIdentity' },        -- solo comprobar identidad
    },
    -- Gangs tambien valen con nb-bridge (usa GetGang)
    marabunta = {
        grades  = { 2, 3 },
        actions = { 'searchPlayer' },
    },
}

Config.AllowedIdentifiers = {
    'license:abc123...',           -- ESX
    'ABCD1234',                    -- QBCore citizenid
}
```

Ver [Permisos](permisos.md) para detalles de como se resuelve cada accion.

---

## Esposas

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Handcuffs.RequireItem` | Exigir el item de esposas en el inventario | `true` |
| `Config.Handcuffs.ItemName` | Nombre interno del item | `'handcuffs'` |

Si `RequireItem = false`, cualquier policia puede esposar sin coste.

---

## Bridges editables

Los archivos en `bridge/` son **open source** (no encriptados). Edita sin miedo para adaptar a tu servidor:

- `bridge/licenses.lua` — deteccion de identidad, carnet, armas. Usa nb-bridge por defecto (`Bridge.GetIdentity`, `Bridge.GetDriverLicense`, `Bridge.GetWeaponLicense`) pero puedes sobrescribir para tu propio script de licencias.
- `bridge/vehicle.lua` — lookup de dueno por matricula. Adaptar aqui si tu servidor usa una columna diferente o un sistema de matriculas custom.
- `bridge/inventory.lua` — forzar apertura del inventario del target. Usa `Bridge.ForceOpenPlayerInventory` por defecto.

Ejemplo — integrar con un script de registros propietario:

```lua
-- bridge/licenses.lua
function Bridge.Licenses.GetIdentity(targetId)
    return exports['mi-registro']:GetPlayerIdentity(targetId)
end
```

---

## Idiomas

Los textos viven en `shared/locale.lua` bajo `Locale['en']` y `Locale['es']`. Para anadir:

1. Duplica el bloque `Locale['es']` a `Locale['xx']`.
2. `Config.Locale = 'xx'`.
