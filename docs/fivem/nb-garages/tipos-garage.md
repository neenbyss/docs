# Tipos de garaje

nb-garages soporta 6 tipos distintos. Eliges el tipo al crear el garaje desde el panel (`/garagecreator`). Cada tipo tiene sus reglas de acceso, su comportamiento de spawn y su tratamiento de vehiculos.

---

## public

Garaje publico sin restriccion.

- Accesible por cualquier jugador.
- Los coches se guardan en la tabla del framework (`owned_vehicles` / `player_vehicles`) como vehiculos personales.
- Sirve para garajes de ciudad (downtown, sandy shores, etc.).

Campo `restriction_type = 'none'`.

---

## society

Flota compartida entre miembros de un trabajo / gang.

- Los vehiculos viven en `nb_garage_vehicles` (flota con `min_grade` por vehiculo).
- El jugador **no es owner** — saca uno de la flota, lo usa y lo devuelve.
- `Config.Garage.SocietyMaxOut` limita cuantos vehiculos iguales pueden estar fuera a la vez.
- Ideal para garaje de comisaria, mecanica, ambulancias, gang.

Campos:
- `restriction_type = 'job'` o `'gang'`.
- `restriction_value = 'police'` (nombre del job/gang).

---

## job

Vehiculos personales pero **solo visibles cuando estas en el trabajo correspondiente**.

- Reutiliza la tabla del framework, pero anade la columna `nb_job` para saber de que trabajo es el coche.
- Cuando el jugador cambia de trabajo, sus coches `job = ...` desaparecen del listado hasta que vuelva al trabajo.
- Util para premiar personajes con vehiculo propio ligado a un rol sin compartirlo.

Campos:
- `restriction_type = 'job'`.
- `restriction_value = 'police'`.

---

## depot

Deposito / impound.

- Los vehiculos se mueven aqui cuando estan abandonados (`Config.Depot.AbandonAfter`).
- Sacar un coche cuesta la fee: `Config.Depot.FeesByClass[vehClass]` o `Config.Depot.DefaultFee`.
- Visible en el mapa si `Config.Depot.ShowOnMap = true`.
- Cuando el owner paga la fee, el coche sale en el spawn del depot y se limpia la flag `pound`.

Campo `restriction_type = 'none'` (todo el mundo puede consultar si tiene coches aqui, pero solo el owner puede retirar).

Detalle completo en [Persistencia e impound](persistencia.md).

---

## house

Garaje ligado a una casa concreta.

- **Dinamico** — no vive estatico en el mapa; aparece cuando entras en el radio de tu casa.
- Requiere una integracion con un sistema de casas. Configura:

    ```lua
    Config.House.Enabled        = true
    Config.House.KeyCheckResource = 'qb-houses'
    Config.House.KeyCheckExport   = 'HasKeys'
    ```

- Puedes registrar garajes de casa dinamicamente desde otro recurso:

    ```lua
    exports['nb-garages']:RegisterHouseGarage(source, {
        garage_id = 'house_42',
        label     = 'Casa de Tony',
        coords    = vector3(x, y, z),
        spawn_coords = vector3(sx, sy, sz),
        house_id  = 42,
    })
    exports['nb-garages']:UnregisterHouseGarage(source, 'house_42')
    exports['nb-garages']:RefreshHouseGarages(source)
    ```

Campo `restriction_type = 'house'`, `house_id` apuntando al id de la casa.

---

## custom

Escape hatch — garaje con logica custom de acceso (gang, VIP, mision, clan).

- `restriction_type = 'gang'` / `'vip'` / `'custom'`.
- `restriction_value` es el parametro (nombre de gang, tier VIP).
- Se valida via funciones en `bridge/`:
    - `Bridge.Garage.HasGang(source, gangName)`.
    - `Bridge.Garage.HasVIP(source, tier)`.
    - `Bridge.Garage.CustomAccess(source, garage)` — escape hatch para cualquier otra regla.

Estas tres funciones viven en la carpeta `bridge/` del recurso, que es **open** (editable sin escrow).

---

## Tabla comparativa

| Tipo | Owner | Ubicacion | Acceso | Restock / flota |
|------|-------|-----------|--------|-----------------|
| `public` | Jugador | Ciudad | Todos | — |
| `society` | Trabajo/gang | Comisaria/bunker | Job + grado | Admin desde el panel |
| `job` | Jugador | Ciudad | Solo on-duty | — |
| `depot` | Jugador | Impound | Todos (con fee) | — |
| `house` | Jugador | Dinamico | House keys | — |
| `custom` | Jugador/society | Depende | Funcion custom | — |

---

## Ejemplo: crear garaje de comisaria (society)

Desde el panel:

| Campo | Valor |
|-------|-------|
| ID | `police_mission_row` |
| Label | `Comisaria Mission Row` |
| Tipo | `society` |
| Restriction type | `job` |
| Restriction value | `police` |
| Coords marker | posicion del parking |
| Coords spawn | donde spawnean los coches |
| Categoria | `car` |
| Blip sprite | `60`, color `29` |

Despues anades los vehiculos de la flota desde el panel en la pestana **Flota**: model, label, min_grade.
