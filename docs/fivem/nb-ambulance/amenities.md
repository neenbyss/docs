# Amenities (Stash / Garage / Pharmacy / Medical Bag)

Phase 7 añade los 4 elementos clasicos del lobby de un hospital: stash medico, garage de vehiculos, farmacia y bolsa portatil. Todos persisten en SQL y se gestionan desde el creator in-game.

> **Requisito:** `ox_inventory` para stash + pharmacy shop + medical bag. Sin ox hay un fallback minimo de buy event para la pharmacy, pero el stash y la bolsa requieren ox.

---

## Stash

Caja de almacenamiento medica. Cada hospital puede tener N stashes. Tipos:

- **Compartido (`shared = true`)**: una sola caja para toda la organizacion. Lo que mete un medico lo ve otro.
- **Personal (`shared = false`)**: cada jugador tiene su propio inventario en ese punto.

### Crear

1. `/creator` → selecciona hospital → pestaña **Stash**.
2. Rellena **label**, **slots** (default 30), **peso** en gramos (default 50000), toggle **Compartido**.
3. Click **`Capturar posicion del stash`** → captura coords donde quieras que aparezca el marker.
4. Persistido. Aparece un marker rojo flotante; cualquier medico cerca con `[E]` abre la caja.

### Backend

- ID en ox_inventory: `nb_amb_stash_<id>` (autogenerado).
- Re-registrado con `exports.ox_inventory:RegisterStash(...)` cada vez que el hospital se reload.
- `owner = nil` para shared, `owner = true` para personal.

---

## Garage

Spawner de vehiculos para medicos.

### Crear

1. `/creator` → pestaña **Garage**.
2. Label + modelo del NPC vendedor (default `s_m_y_cop_01`).
3. **Captura del NPC** (donde habla el medico).
4. **Captura del spawn point** (donde aparece el vehiculo).
5. Click **`Crear garage`**.

### Añadir vehiculos al garage

Cada garage en la lista expone un sub-form:

- **Label**: el nombre que ve el medico (ej. `Ambulance`).
- **Model**: hash del modelo (ej. `ambulance`).
- **Grado minimo**: filtro contra el `job.grade` del medico.

Click **`+`** para añadir. La lista muestra todos los vehiculos disponibles en ese garage.

### Uso

El medico se acerca al NPC del garage → `[E]` → se abre una NUI con la lista filtrada por su grado → click → vehiculo aparece en el spawn point.

### Backend

- Validacion server-side (`:server:requestVehicle` callback) lookup `garage_id + vehicle_id` en el cache.
- `Bridge.SpawnVehicle` para spawn — usa la implementacion del framework (ESX `ESX.OneSync.SpawnVehicle` / QBCore `QBCore.Functions.SpawnVehicle`).
- Fallback `CreateVehicle` cuando el bridge no expone spawn.

---

## Pharmacy

Tienda de items medicos. Vende splint, bandage, tourniquet, etc. al medico on-duty con descuento opcional por grado.

### Crear

1. `/creator` → pestaña **Pharmacy**.
2. Label + modelo del NPC vendedor (default `s_m_m_doctor_01`).
3. Click **`Capturar posicion del NPC`**.

### Añadir items

Cada pharmacy en la lista tiene su sub-form:

- **`item_name`** — nombre exacto del item en tu inventario.
- **Label** — texto visible en la tienda.
- **Precio** ($).
- **Grado minimo**.

Click **`+`** añade el item. Se reflejan en `ox_inventory:RegisterShop(...)` automaticamente.

### Uso

Medico se acerca al NPC → `[E]` → ox_inventory abre la tienda con la lista de items vendidos. El precio se cobra del banco (configurable en ox via `groups`).

### Sin ox_inventory

Sin ox, la pharmacy cae al fallback `:server:pharmacyBuy` callback que valida + cobra del banco + entrega el item via `Bridge.AddItem`. La interaccion del lado cliente es minima (solo se compra el primer item al pulsar E) — recomendado actualizar a ox para experiencia completa.

---

## Medical Bag (item portatil)

Stash personal del medico que cabe en el inventario. Permite cargar consumibles sin volver al hospital.

### Configuracion

```lua
Config.MedicalBag = {
    Enabled  = true,
    ItemName = 'medical_bag',
    Slots    = 15,
    Weight   = 30000,  -- 30kg
}
```

### Registrar el item en ox_inventory

`data/items.lua`:

```lua
['medical_bag'] = {
    label = 'Bolsa Medica',
    weight = 1000,
    stack = false,
    close = false,
    consume = 0,
    server = {
        export = 'nb-ambulance.use_medical_bag'
    }
}
```

### Uso

El medico tiene el item en el inventario → click derecho → `Use` → se abre un stash unico ligado a su identifier (`nb_amb_medbag_<charid>`). Lo que guarde aqui es exclusivamente suyo.

### Otros inventarios

Sin ox_inventory, dispara desde el callback de uso del item:

```lua
TriggerServerEvent('nb-ambulance:server:useMedicalBag')
```

Pero ten en cuenta que las stashes personales con owner explicito son funcionalidad propia de ox — sin ox, el feature avisa con toast y no abre.

---

## Permisos

Las 4 amenities estan **gated por `Config.MedicalJobs`**:

- Solo jugadores con job en la lista pueden interactuar con stash, garage, pharmacy.
- Admins (`Bridge.IsAdmin`) bypass siempre.
- El grado minimo (`min_grade`) en vehiculos / items pharmacy filtra contra `job.grade`.

---

## Schema SQL

```
nb_ambulance_stashes          -> 1 fila por stash (slots, weight, shared, pos)
nb_ambulance_garages          -> 1 fila por garage (ped pos + spawn pos)
nb_ambulance_garage_vehicles  -> N vehiculos por garage (FK CASCADE)
nb_ambulance_pharmacies       -> 1 fila por pharmacy (ped pos)
nb_ambulance_pharmacy_items   -> N items por pharmacy (FK CASCADE)
```

Eliminar un hospital borra todo lo relacionado en cascada.
