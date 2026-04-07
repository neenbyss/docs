# Configuracion

La configuracion se divide en varios archivos dentro de `shared/`.

---

## shared/config.lua

### Comandos y keybind

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Commands.Open` | Comando para abrir la tienda VIP | `'vip'` |
| `Config.Commands.Admin` | Comando para abrir el panel admin | `'vipadmin'` |
| `Config.OpenKey` | Tecla para abrir el menu VIP (`false` para desactivar) | `'F5'` |

### Grupos de admin

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Lista de grupos con acceso al panel admin | `{'admin', 'superadmin', 'god'}` |

### Garage

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Garage.garage` | Sistema de garage a usar | `'esx_garage'` |
| `Config.Garage.DefaultGarage` | Nombre del garage donde se guardan los vehiculos | `'VespucciBoulevard'` |

Garages soportados:

- `'esx_garage'` — Tabla `owned_vehicles`
- `'qb-garages'` — Tabla `player_vehicles`
- `'lunar_garage'` — Tabla `owned_vehicles` (formato Lunar)
- `'custom'` — Llama al export `GiveVehicle` de nb-vip (debes implementarlo)

### Discord Webhooks

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Webhooks.Enabled` | Activar webhooks de Discord | `true` |
| `Config.Webhooks.PurchaseURL` | URL del webhook para compras | `''` |
| `Config.Webhooks.CodeRedeemURL` | URL del webhook para canjes de codigos | `''` |
| `Config.Webhooks.AdminActionURL` | URL del webhook para acciones de admin | `''` |
| `Config.Webhooks.Color` | Color del embed (decimal) | `16753920` |
| `Config.Webhooks.ServerName` | Nombre del servidor en los embeds | `'My Server'` |

### Tiers VIP

Cada tier se define en `Config.Tiers` con:

| Campo | Descripcion |
|-------|-------------|
| `label` | Nombre visible |
| `color` | Color hex |
| `order` | Orden de aparicion |
| `icon` | Clase de FontAwesome |

Tiers por defecto: `bronze`, `silver`, `gold`, `diamond`.

### Categorias de tienda

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Store.EnablePackages` | Mostrar pestana de paquetes | `true` |
| `Config.Store.EnableItems` | Mostrar pestana de items individuales | `true` |
| `Config.Store.EnableVehicles` | Mostrar pestana de vehiculos individuales | `true` |

### Paquetes VIP (`Config.Packages`)

Cada paquete es una tabla con:

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| `id` | string | Identificador unico |
| `label` | string | Nombre visible |
| `description` | string | Descripcion en la UI |
| `tier` | string | Tier asociado (bronze, silver, etc.) |
| `price` | number | Precio en coins |
| `icon` | string | Clase de FontAwesome |
| `rewards.items` | table | `{ {name = 'item', count = n}, ... }` |
| `rewards.vehicles` | table | `{ 'modelo1', 'modelo2' }` |
| `rewards.money` | table | `{ cash = n, bank = n }` |
| `rewards.permissions` | table | `{ 'perm1', 'perm2' }` |
| `rewards.custom_actions` | table | `{ 'action_id' }` |

### Items individuales (`Config.StoreItems`)

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| `id` | string | Identificador unico |
| `name` | string | Nombre del item (spawn name) |
| `label` | string | Nombre visible |
| `description` | string | Descripcion en la UI |
| `price` | number | Precio en coins |
| `count` | number | Cantidad que se entrega |
| `icon` | string | Clase de FontAwesome |
| `category` | string | Categoria para filtrado |

### Vehiculos individuales (`Config.StoreVehicles`)

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| `id` | string | Identificador unico |
| `model` | string | Spawn name del vehiculo |
| `label` | string | Nombre visible |
| `description` | string | Descripcion en la UI |
| `price` | number | Precio en coins |
| `icon` | string | Clase de FontAwesome |
| `category` | string | Categoria para filtrado |

### Acciones custom (`Config.CustomActions`)

Cada accion se define como clave en la tabla:

```lua
Config.CustomActions = {
    ['mi_accion'] = {
        label = 'Mi Accion',
        description = 'Descripcion visible en la UI',
    },
}
```

La logica de ejecucion esta en `server/packages.lua`.

---

## shared/config_coins.lua

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Coins.Currency` | Nombre visible de la moneda | `'Coins'` |
| `Config.Coins.StartingBalance` | Balance inicial para nuevos jugadores | `0` |
| `Config.Coins.MaxBalance` | Balance maximo permitido | `999999` |
| `Config.Coins.AllowTransfers` | Permitir transferencias entre jugadores | `false` |

---

## shared/config_codes.lua

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.CodeFormat` | Formato del codigo (X = caracter aleatorio) | `'XXXX-XXXX-XXXX'` |
| `Config.CodeCharacterSet` | Caracteres usados para generar codigos | `'ABCDEFGHJKLMNPQRSTUVWXYZ23456789'` |
| `Config.DefaultMaxUses` | Usos maximos por defecto | `1` |
| `Config.DefaultExpiry` | Horas de expiracion (0 = nunca) | `0` |
| `Config.AllowMultipleRedeems` | Permitir que un jugador canjee el mismo codigo varias veces | `false` |

---

## shared/locale.lua

Los textos de la UI y notificaciones se definen en `Locale['es']` y `Locale['en']`. Cambia el idioma con:

```lua
Config.Locale = 'es'  -- o 'en'
```

Para agregar un idioma nuevo, crea una tabla `Locale['xx']` con las mismas claves que `Locale['es']`.
