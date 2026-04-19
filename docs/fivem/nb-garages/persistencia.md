# Persistencia e impound

nb-garages mantiene los vehiculos en el mundo tras reinicios y envia los abandonados al deposito. Todo esto es opcional y configurable.

---

## Persistencia de vehiculos

Cuando un vehiculo sale del garaje, su posicion se registra en `nb_persistent_vehicles`. Cada cierto tiempo (o al cerrarse con llaves) se actualiza. Al reiniciar el servidor, los vehiculos tracked se vuelven a spawnear en la misma posicion.

### Opciones

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Config.Persistence.Enabled` | On/off global | `true` |
| `Config.Persistence.ServerSideSpawn` | Usa `CreateVehicleServerSetter` — el coche no depende del cliente cercano para existir | `true` |
| `Config.Persistence.CullingRadius` | Distancia maxima en la que el servidor propaga la entidad | `5000.0` |
| `Config.Persistence.SaveOnLock` | Guarda al cerrar con llaves | `true` |
| `Config.Persistence.RecoverFee` | Coste para re-spawnear un coche abandonado en el garaje | `500` |
| `Config.Persistence.RespawnCheckInterval` | ms entre chequeos de entidades borradas | `10000` |

### Que pasa al reiniciar

1. Al arrancar, el servidor lee `nb_persistent_vehicles`.
2. Para cada fila, comprueba si hay un owner online:
   - Si **si**, spawnea el coche en `coords` con `props` guardados.
   - Si **no** y `Config.Depot.OrphanCleanupOnStart = true`, manda el coche al depot (flag `state = 0`).

---

## Tabla `nb_persistent_vehicles`

```sql
CREATE TABLE `nb_persistent_vehicles` (
    `plate` VARCHAR(12) NOT NULL PRIMARY KEY,
    `coords` LONGTEXT NOT NULL,
    `heading` FLOAT DEFAULT 0.0,
    `props` LONGTEXT DEFAULT NULL,
    `last_garage` VARCHAR(50) NOT NULL,
    `owner` VARCHAR(60) DEFAULT NULL,
    `job_name` VARCHAR(50) DEFAULT NULL,
    `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## Deteccion de movimiento (anti-falso-abandono)

Un coche aparcado 2 horas porque el jugador fue a dormir no es "abandonado". nb-garages distingue entre **quieto porque se olvido** y **quieto porque el owner esta offline / lejos**:

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Config.Persistence.MovementTracking.Enabled` | Sistema activo | `true` |
| `Config.Persistence.MovementTracking.Interval` | Segundos entre sweeps | `300` |
| `Config.Persistence.MovementTracking.Threshold` | Metros para considerar "movido" | `15.0` |

El ciclo registra la posicion cada `Interval`. Si un coche no se ha movido mas de `Threshold` y su owner no esta cerca/online, empieza a contar hacia `Config.Depot.AbandonAfter`.

---

## Impound / deposito

Cuando un coche se considera abandonado se **mueve al deposito** (set de flags `pound = 1` y/o `state = 0` segun framework). El owner puede recuperarlo pagando la fee.

### Opciones

| Opcion | Descripcion | Default |
|--------|-------------|---------|
| `Config.Depot.Enabled` | On/off | `true` |
| `Config.Depot.DefaultFee` | Fee base | `500` |
| `Config.Depot.FeesByClass` | `{ [classId] = fee }` — override por clase GTA | `{}` |
| `Config.Depot.AbandonAfter` | Segundos antes de ser elegible | `7200` (2h) |
| `Config.Depot.CheckInterval` | Segundos entre sweeps | `600` (10min) |
| `Config.Depot.SkipWhenOwnerOnline` | No impoundar si el owner esta conectado | `true` |
| `Config.Depot.OrphanCleanupOnStart` | Huerfanos -> depot al arrancar | `true` |
| `Config.Depot.ShowOnMap` | Blip del depot en el mapa | `true` |

### FeesByClass ejemplo

```lua
Config.Depot.FeesByClass = {
    [7]  = 3000,   -- Super
    [12] = 2000,   -- Vans
    [16] = 10000,  -- Planes
    [17] = 5000,   -- Service
}
```

Clases GTA: ver native `GET_VEHICLE_CLASS` (0x29439776).

### Export programatico

```lua
-- server
exports['nb-garages']:SendToDepot(plate, fee, reason)
```

Util para recursos que quieran impoundar manualmente (ej. `/impound` de policia).

---

## Stats y debugging

```lua
local stats = exports['nb-garages']:GetPersistenceStats()
-- {
--   tracked     = 124,   -- vehiculos con entity tracked
--   db_rows     = 142,   -- filas en la tabla persistente
--   stale       = 18,    -- filas sin entity activa
--   impounded   = 37,    -- en el depot ahora
-- }
```

Con `Config.Debug = true` tambien se imprimen los sweeps:

```
[nb-garages][SERVER] persistence sweep: 124 tracked, 3 stale purged, 2 sent to depot
```

---

## Recovery fee

Cuando un vehiculo desaparece del mundo (crash, culling agresivo, bug) pero sigue en la BD, el owner puede **re-summonearlo** desde su garaje de origen pagando `Config.Persistence.RecoverFee`. Esto evita que se quede atrapado mientras no vuelvan a cargar el area.

UI: boton **"Recuperar"** en el panel de `/myvehicles` para vehiculos marcados como `stale`.
