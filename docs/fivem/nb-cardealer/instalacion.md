# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Artifacts 5181+ |
| **oxmysql** | Queries |
| **es_extended** | Framework (ESX Legacy 1.x+) |
| **esx_society** | Panel boss y sociedades |
| **esx_addonaccount** | Cuenta compartida del concesionario (`society_<job>`) |

---

## 1. Instalar el recurso

1. Coloca **nb-cardealer** en `resources/`.
2. Asegurate de que los 4 recursos anteriores estan operativos.

---

## 2. Base de datos

Importa `sql/install.sql`:

```bash
mysql -u usuario -p nombre_db < nb-cardealer/sql/install.sql
```

Tablas creadas:

| Tabla | Descripcion |
|-------|-------------|
| `nb_dealerships` | Concesionarios (nombre, job, society_account, coords de sell_point y manage_point). |
| `nb_dealer_sell_requests` | Peticiones de venta de jugadores con estado (`PENDING`, `COUNTERED`, `ACCEPTED`, `REJECTED`, `CANCELLED`). |
| `nb_dealer_stock` | Vehiculos que el concesionario ha comprado (flota para revender). |
| `nb_dealer_showroom` | Vehiculos fisicamente colocados en el mundo con precio y slot. |
| `nb_dealer_payouts` | Registro de pagos a vendedores individuales. |

---

## 3. Configuracion minima

Edita `config.lua`:

```lua
Config.Framework       = 'esx'
Config.SocietyProvider = 'esx_society'
Config.AdminGroups     = { 'admin', 'superadmin' }
Config.MaxPrice        = 100000000              -- tope anti-troll
Config.MaxActionDistance = 4.0                  -- radio de markers
Config.MaxBuyDistance    = 3.0                  -- radio para comprar en showroom

Config.Logs.DiscordWebhook = ''                 -- opcional
```

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended
ensure esx_society
ensure esx_addonaccount
ensure nb-cardealer
```

---

## 5. Crear un concesionario

1. Entra al servidor como admin (grupo en `Config.AdminGroups`).
2. Ejecuta `/nb_createdealer` — se abre un modal.
3. Rellena:
    - **Nombre** — "Premium Deluxe Motorsport".
    - **Job** — `cardealer` (el trabajo debe existir previamente, con grades `boss` y empleados).
4. Se crea el concesionario + la cuenta `society_cardealer` via `esx_addonaccount`.
5. Entra como boss del job y ejecuta `/nb_bossmenu`:
    - Ve a **Settings** → **Sell point**: define donde los jugadores llegan a vender.
    - Ve a **Settings** → **Manage point**: define donde el staff gestiona requests/stock.

---

## 6. Comprobaciones

- Consola server: la cuenta `society_cardealer` aparece en la lista de `esx_addonaccount`.
- Con otro jugador: acercate al **sell point** en un coche y pulsa E → formulario de venta.
- Con el boss: `/nb_bossmenu` → ver el request pendiente.
- Con el cliente: ir al punto donde el staff ponga vehiculos del showroom y comprar.
