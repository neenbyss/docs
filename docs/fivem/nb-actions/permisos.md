# Permisos

nb-actions usa un modelo de **tres capas paralelas**. Un jugador tiene permiso para una accion si **alguna** de las tres le da acceso.

---

## 1. Admin bypass

```lua
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
Config.AdminBypass = true
```

Si `AdminBypass = true`, los grupos de `AdminGroups` tienen acceso automatico a **todas** las acciones sin mas validaciones. `Bridge.IsAdmin(src)` es el chequeo.

Desactiva `AdminBypass` si tu server tiene rol-play estricto y quieres que los admins tambien pasen por la validacion de job/gang.

---

## 2. Grupos permitidos (`Config.AllowedGroups`)

La forma normal de dar acceso por rol.

```lua
Config.AllowedGroups = {
    police = {
        grades  = { 0, 1, 2, 3, 4 },
        actions = { 'handcuff', 'drag', ... },
    },
    ambulance = {
        grades  = { 0, 1, 2 },
        actions = { 'checkIdentity' },
    },
    vagos = {              -- tambien acepta gangs (QBCore + nb-bridge)
        grades  = { 3 },
        actions = { 'searchPlayer' },
    },
}
```

### Matching

- La clave (`police`, `ambulance`, ...) se compara con `Bridge.GetJob(src).name` **y** con `Bridge.GetGang(src).name`.
- `grades` es un array de numeros. Para "todos los grados", usa:
    ```lua
    grades = true
    ```
- `actions` es un array con los ids de acciones permitidas. Pon `'*'` para todas:
    ```lua
    actions = { '*' }
    ```

### Grado minimo efectivo

El listado de `grades` es explicito — si omites el 0, los novatos no tienen acceso. Para "grado 2 o superior":

```lua
grades = { 2, 3, 4, 5, 6, 7, 8, 9, 10 }
```

---

## 3. Identificadores especificos (`Config.AllowedIdentifiers`)

Atajo para dar permiso a jugadores concretos que no estan en un rol policial. Util para devs, staff sin rol admin o personajes especiales.

```lua
Config.AllowedIdentifiers = {
    'license:abcdef1234...',     -- ESX license (primaria)
    'steam:11000010a1b2c3d',     -- tambien steam
    'ABCD1234',                  -- QBCore citizenid
}
```

Los jugadores de la lista tienen **todas** las acciones disponibles, igual que admins con bypass.

---

## Callback que calcula el permiso

En runtime, el cliente llama:

```lua
Bridge.TriggerServerCallback('nb-actions:getAllowedActions', function(actions)
    -- actions = { handcuff = true, drag = true, checkIdentity = false, ... }
end)
```

El server devuelve un mapa `action_id -> bool`. El UI filtra las acciones con este resultado antes de dibujar.

Se recalcula automaticamente cuando:

- El jugador entra al servidor (`OnPlayerLoaded`).
- Cambia de trabajo (`OnJobUpdate`).
- Cambia de gang (`OnGangUpdate`).

Asi no hace falta reabrir el panel tras un cambio de turno.

---

## Seguridad del servidor

Ademas del filtro client-side, **todos los eventos de server** re-validan el permiso antes de ejecutar. Un cliente modificado que dispare `nb-actions:handcuffPlayer` sin pasar el panel sera rechazado con silencio (el servidor no responde).

```lua
-- server/main.lua (pseudo)
RegisterNetEvent('nb-actions:handcuffPlayer', function(targetId)
    local src = source
    if not PlayerCanDoAction(src, 'handcuff') then return end
    -- ... logica
end)
```

---

## Ejemplo: server minimo solo para policia

```lua
Config.AdminBypass = false       -- admins tambien pasan por rol
Config.AllowedGroups = {
    police = {
        grades = true,
        actions = { '*' },
    },
}
Config.AllowedIdentifiers = {}   -- vacio
```

Con este setup, solo los personajes con trabajo `police` tienen acceso al panel, independientemente del ACE del jugador.
