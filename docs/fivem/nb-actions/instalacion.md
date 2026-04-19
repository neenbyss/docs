# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Artifacts 5181+ |
| **oxmysql** | Queries al framework |
| **nb-bridge** | Bridge centralizado |
| **Framework** | ESX Legacy o QBCore |
| **Inventario** | cualquiera soportado por nb-bridge |

---

## 1. Instalar el recurso

1. Coloca **nb-actions** en `resources/`.
2. Comprueba que **nb-bridge** este instalado.

---

## 2. Base de datos

**No hay tablas custom.** nb-actions consulta directamente las tablas del framework (`owned_vehicles`, `users`, `player_vehicles`, `players`) para el lookup de vehiculos y licencias.

---

## 3. Configuracion minima

Edita `shared/config.lua`:

```lua
Config.AdminGroups   = { 'admin', 'superadmin', 'god' }
Config.Locale        = 'es'
Config.Keybind       = 'F6'
Config.MaxPlayerDistance  = 3.0
Config.MaxVehicleDistance = 5.0

-- Quien puede usar el panel (ademas de admins con AdminBypass).
Config.AllowedGroups = {
    police = {
        grades  = { 0, 1, 2, 3, 4 },           -- qualquier grado
        actions = { 'handcuff', 'drag', 'putInVehicle', 'takeFromVehicle',
                    'checkIdentity', 'checkDriverLicense', 'checkWeaponLicense',
                    'checkVehicleOwner', 'searchPlayer', 'searchDeadPlayer' },
    },
    ambulance = {
        grades  = { 0, 1, 2 },
        actions = { 'checkIdentity' },
    },
}
```

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended       # o qb-core
ensure nb-bridge
ensure nb-actions
```

---

## 5. Comprobar que funciona

1. Entra como un personaje con trabajo `police` (o como admin con `AdminBypass = true`).
2. Acercate a otro jugador a menos de `MaxPlayerDistance`.
3. Pulsa **F6** — aparece el panel con las acciones permitidas para tu rol.
