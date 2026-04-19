# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Artifacts 5181+ |
| **oxmysql** | MariaDB 10.0+ recomendado |
| **nb-bridge** | Bridge centralizado de Neenbyss |
| **Framework** | ESX Legacy (1.9+) o QBCore |
| **ox_lib** (opcional) | Notificaciones + textUI |
| **Minigames** (opcional) | ox_lib, qb-minigames o ps-ui |

---

## 1. Retirar `qb-vehiclekeys` (si lo tenias)

nb-garages reemplaza `qb-vehiclekeys` declarando `provide 'qb-vehiclekeys'` en su manifest. Saca `qb-vehiclekeys` del `server.cfg`.

---

## 2. Instalar el recurso

1. Coloca **nb-garages** en `resources/`.
2. Comprueba que **nb-bridge** este presente.

---

## 3. Base de datos

Importa `[sql]/garages.sql`:

```bash
mysql -u usuario -p nombre_db < nb-garages/\[sql\]/garages.sql
```

Crea 3 tablas propias y amplia las tablas del framework:

| Tabla | Descripcion |
|-------|-------------|
| `nb_garages` | Definicion de cada garaje (tipo, coords, spawn, blip, restricciones). |
| `nb_garage_vehicles` | Flota de vehiculos de sociedad/trabajo compartida. |
| `nb_persistent_vehicles` | Posiciones de vehiculos para persistencia entre reinicios. |

**Columnas anadidas al framework:**

- ESX `owned_vehicles` — `parking`, `pound`, `depotprice`, `type`, `nb_job`.
- QBCore `player_vehicles` — `nb_job`.

El schema usa `ADD COLUMN IF NOT EXISTS` (requiere MariaDB 10.0+).

---

## 4. Configuracion minima

Edita `shared/config.lua`:

```lua
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
Config.Locale      = 'es'

Config.Minigame.Engine = 'native'   -- 'native' | 'ox_lib' | 'qb-minigames' | 'ps-ui' | 'none'

Config.Persistence.Enabled     = true
Config.Persistence.RecoverFee  = 500   -- coste para re-spawnear un coche abandonado
```

Resto con defaults razonables. Ver [Configuracion](configuracion.md).

---

## 5. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended       # o qb-core
ensure ox_lib            # opcional pero recomendado
ensure nb-bridge
ensure nb-garages
```

---

## 6. Crear tu primer garaje

1. Entra al servidor como admin.
2. `/garagecreator` — se abre el panel.
3. **Nuevo** -> rellena:
   - ID (`downtown_public`)
   - Label (`Garaje Downtown`)
   - Tipo `public`
   - Coords del marker
   - Coords de spawn
   - Categoria (`car`, `air`, `sea`, `rig`, `all`)
4. Guardar. Aparece el marker en el sitio.
5. Acercate, pulsa E y prueba sacar un coche.

---

## 7. Comprobaciones

- Consola server: `[nb-garages] Loaded (<framework> / <inventory>)`.
- `/myvehicles` como jugador — lista tus coches.
- Si tienes `ox_lib`, las notificaciones salen con estilo ox; si no, caen a nativas.
