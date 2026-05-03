# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Artifacts 5181+ |
| **oxmysql** | Persistencia de hospitales |
| **nb-bridge** | Bridge centralizado (loader.lua) |
| **Framework** | ESX Legacy o QBCore |
| **ox_inventory (opcional)** | Auto-detectado para items medicos |

---

## 1. Retirar sistemas previos

nb-ambulance reemplaza los sistemas antiguos. Retira del `server.cfg` cualquier ambulancejob anterior:

```cfg
# ensure esx_ambulancejob   # quitar
# ensure qb-ambulancejob    # quitar
```

Captura el evento de muerte mediante `gameEventTriggered` — coexistir con otros scripts de muerte produce dobles death screens.

---

## 2. Instalar el recurso

1. Coloca **nb-ambulance** en `resources/[neenbyss]/nb-ambulance/`.
2. Verifica que **nb-bridge** este instalado y arranque antes.

---

## 3. Base de datos

Importa `[sql]/nb-ambulance.sql`:

```bash
mysql -u usuario -p nombre_db < nb-ambulance/\[sql\]/nb-ambulance.sql
```

Tablas creadas (todas con FK CASCADE a `nb_ambulance_hospitals`):

| Tabla | Descripcion |
|-------|-------------|
| `nb_ambulance_hospitals` | Hospital base: nombre, label, blip, tarifa, coordenadas centrales. |
| `nb_ambulance_beds` | Camas (respawn + spawn point opcional separado). |
| `nb_ambulance_npcs` | NPCs del lobby (rol, modelo, scenario, posicion). |
| `nb_ambulance_jobs` | Whitelist de jobs por hospital con grado minimo. |

---

## 4. Orden en `server.cfg`

```cfg
ensure oxmysql
ensure es_extended           # o qb-core
ensure ox_inventory          # opcional
ensure nb-bridge             # OBLIGATORIO antes de nb-ambulance
ensure nb-ambulance
```

---

## 5. Configuracion minima

Edita `shared/config.lua`:

```lua
Config.AdminGroups   = { 'admin', 'superadmin', 'god' }
Config.Locale        = 'es'
Config.MedicalJobs   = { 'ambulance', 'doctor', 'paramedic' }
```

---

## 6. Hospitales

nb-ambulance arranca **sin hospitales** registrados. Tienes dos caminos:

### a) Creator in-game (recomendado)

Como admin, ejecuta:

```
/creator
```

Crea hospital → captura coordenadas in-game → añade camas/NPCs/jobs en las pestañas. Persiste a SQL automaticamente. Ver **[Creator](creator.md)**.

### b) Seed por SQL

Si quieres arrancar con Pillbox precargado, ejecuta:

```sql
INSERT INTO `nb_ambulance_hospitals`
    (name, label, enabled, blip_enabled, blip_sprite, blip_color, blip_scale, respawn_cost, center_x, center_y, center_z)
VALUES
    ('pillbox', 'Pillbox Hill Medical', 1, 1, 61, 2, 0.8, 500, 307.70, -1433.40, 29.90);
SET @h = LAST_INSERT_ID();

INSERT INTO `nb_ambulance_beds` (hospital_id, label, bed_x, bed_y, bed_z, bed_h, spawn_x, spawn_y, spawn_z, spawn_h) VALUES
    (@h, 'ER Bed 1', 337.27, -1397.72, 32.51, 48.0,  338.40, -1396.90, 32.51, 230.0),
    (@h, 'ER Bed 2', 332.93, -1395.95, 32.51, 230.0, 333.80, -1396.90, 32.51, 230.0),
    (@h, 'ER Bed 3', 329.00, -1395.56, 32.51, 48.0,  330.10, -1394.70, 32.51, 230.0);

INSERT INTO `nb_ambulance_npcs` (hospital_id, role, model, scenario, pos_x, pos_y, pos_z, pos_h) VALUES
    (@h, 'doctor', 's_m_m_doctor_01', 'WORLD_HUMAN_CLIPBOARD', 309.05, -1433.93, 29.90, 230.0);

INSERT INTO `nb_ambulance_jobs` (hospital_id, job_name, min_grade) VALUES
    (@h, 'ambulance', 0), (@h, 'doctor', 0), (@h, 'paramedic', 0);
```

Despues ejecuta `/nbamb-reload` (o reinicia el recurso) para refrescar el cache.

---

## 7. Items en el inventario

nb-ambulance registra los items via **nb-bridge**, asi que en cualquier inventario soportado (ox / qs / qb / origen / ESX/QBCore default) basta con que los items existan en el catalogo del inventario y nb-ambulance se encarga del "use" automaticamente.

### Items minimos a definir

```
bandage, splint, tourniquet, burn_dressing, oxygen_mask, medikit,
defibrillator, medical_bag
```

Los nombres son configurables en `Config.Injuries.Items` y `Config.MedicalBag.ItemName` si tu economia usa otros.

### ox_inventory (opcional, da la mejor UX)

`data/items.lua` con declaraciones simples — **NO necesitas el `server.export`**, nb-ambulance ya engancha el `usingItem` hook automaticamente:

```lua
['bandage']        = { label = 'Bandage',       weight = 50  },
['splint']         = { label = 'Splint',        weight = 100 },
['tourniquet']     = { label = 'Tourniquet',    weight = 80  },
['burn_dressing']  = { label = 'Burn dressing', weight = 60  },
['oxygen_mask']    = { label = 'Oxygen mask',   weight = 200 },
['medikit']        = { label = 'Medikit',       weight = 500 },
['defibrillator']  = { label = 'Defibrillator', weight = 800 },
['medical_bag']    = { label = 'Bolsa medica',  weight = 1000, stack = false, close = false },
```

### qb-inventory / esx-inventory_hud / qs-inventory / origen / framework default

Solo añade los items al catalogo de tu inventario. nb-ambulance los registra automaticamente via `Bridge.RegisterUsableItem` y el callback se dispara cuando el jugador hace "use".

---

## 8. Verificacion

1. Reinicia el server. La consola debe mostrar:
   ```
   [nb-ambulance] Server loaded (v0.1.0)
   [nb-ambulance][SERVER][Hospitals] loaded N hospitals from DB
   ```
2. Como admin: `/creator` abre el editor.
3. Como medico (job en `Config.MedicalJobs`): `/dispatch` abre el panel (vacio si no hay llamadas).
4. Mata tu personaje: debe entrar en writhe loop con timer de bleed-out.
