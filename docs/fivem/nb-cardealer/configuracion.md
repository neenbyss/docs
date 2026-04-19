# Configuracion

Toda la config vive en `config.lua` (archivo abierto, no encriptado).

---

## Framework

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Framework` | Framework target | `'esx'` |
| `Config.SocietyProvider` | Proveedor de sociedades | `'esx_society'` |

Para QBCore, edita `framework.lua` (tambien abierto) — los wrappers estan centralizados.

---

## UI / Interaccion

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.UseOxLib` | Usar ox_lib en lugar de NUI pura | `false` |
| `Config.InteractionKey` | Keycode de interaccion (38 = E) | `38` |
| `Config.MaxActionDistance` | Radio para ver markers (m) | `4.0` |
| `Config.MaxBuyDistance` | Radio para comprar en showroom (m) | `3.0` |
| `Config.EnableServerDistanceChecks` | Validar distancia tambien en server | `true` |

---

## Showroom

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.ShowroomVehicleInvincible` | Vehiculos spawnados invencibles | `true` |
| `Config.ShowroomVehicleFrozen` | Congelados (no se pueden mover) | `true` |
| `Config.ShowroomVehicleLocked` | Cerrados con llave | `true` |

---

## Seguridad / Admin

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos admin | `{ 'admin', 'superadmin' }` |
| `Config.EnablePlateLocks` | Validar propietario del plate antes de vender | `true` |
| `Config.MaxPrice` | Precio max por sell request / counter | `100000000` (100M) |

**Rate limiting (hardcoded v1.6):**

| Accion | Cooldown |
|--------|----------|
| Crear sell request | 3s |
| Dealer action (accept/reject/counter) | 2s |
| Player accept counter | 3s |
| Comprar en showroom | 3s |
| Deposito/retiro sociedad | 2s |

No son configurables — evitan spam de duplicaciones y race conditions.

---

## Logs Discord

```lua
Config.Logs.DiscordWebhook = ''      -- vacio = deshabilitado

Config.Logs.Messages = {
    sellRequestCreated   = { title = 'Nueva peticion', description = '...' },
    sellRequestAccepted  = { title = 'Aceptada',       description = '...' },
    counterAccepted      = { title = 'Contra aceptada',description = '...' },
    showroomPurchase     = { title = 'Compra',         description = '...' },
}
```

Al configurar la URL, cada evento dispara un embed con:

- Jugador / dealer.
- Vehiculo (model + plate).
- Precios (asking / counter / final).
- Timestamp.

---

## Comandos

```lua
Config.Commands = {
    CreateDealer  = 'nb_createdealer',
    DeleteDealer  = 'nb_deletedealer',
    BossMenu      = 'nb_bossmenu',
    AdminPanel    = 'nb_admin',
}
```

Cada entrada cambia el nombre del comando correspondiente.

---

## Locales

**Un solo idioma**, editable inline en `config.lua`:

```lua
Config.Locale = {
    header = {
        dealerships = 'Concesionarios',
        ...
    },
    sell = {
        title = 'Vender tu vehiculo',
        askingPrice = 'Precio de venta',
        submit = 'Enviar peticion',
        ...
    },
    manage = {
        requests = 'Peticiones',
        stock    = 'Stock',
        money    = 'Dinero',
        settings = 'Ajustes',
        ...
    },
    buy = { ... },
    counter = { ... },
    showroom = { ... },
    admin = { ... },
    confirm = { ... },
    createDealer = { ... },
    common = { ... },
}
```

Por defecto viene en **Espanol**. Para traducir a otro idioma, edita los strings directamente — no hay archivo separado de locales.

---

## Hooks editables

- **`framework.lua`** — todos los wrappers a ESX (money, inventory, identifiers, plate ownership). Editar aqui para adaptar a QBCore.
- **`config.lua`** — locales + overrides.
- **`html/*`** — UI HTML/CSS/JS (no Vue, es vanilla + jQuery).
- **`sql/*`** — schema.

Todo lo demas (`server/`, `client/`) viene cifrado para Tebex.
