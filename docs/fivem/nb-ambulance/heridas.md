# Sistema de heridas

Tracking de daño **per-hueso x tipo** con efectos de movimiento, bleed tick pasivo, NUI de tratamiento para paramedicos y hooks de items para cualquier inventario.

---

## Modelo de datos

Cada jugador acumula daño en una tabla:

```
injuries[boneId] = {
    fracture    = 0,
    bleeding    = 0,
    burn        = 0,
    suffocating = 0,
}
```

Los valores se acumulan con cada hit (no se resetean automaticamente). Se limpian al recibir tratamiento, al ser revivido o al respawnear.

### Huesos (12 por defecto)

`head`, `neck`, `torso_upper`, `torso_lower`, `larm`, `rarm`, `lhand`, `rhand`, `lleg`, `rleg`, `lfoot`, `rfoot`.

Cada hueso lleva su hash y su label en `Config.Injuries.Bones`.

### Tipos de daño

| Tipo | Causas tipicas | Item de tratamiento |
|------|----------------|---------------------|
| `fracture` | impactos, atropellos, explosiones, puños | splint |
| `bleeding` | balas, cortes, pistola, rifle, escopeta | bandage / tourniquet |
| `burn` | molotov, fuego, gasolina, bengala | burn_dressing |
| `suffocating` | ahogamiento, asfixia | oxygen_mask |

El mapeo arma → tipo vive en `Config.Injuries.WeaponDamageTypes`. Las armas no listadas usan `DefaultDamageType` (por defecto `bleeding`).

### Severidad

```lua
Config.Injuries.Severity = {
    minor   = 1,   -- visible en NUI de tratamiento
    major   = 35,  -- desencadena movement penalty + warnings
    extreme = 70,  -- desencadena bleed tick acelerado
}
```

---

## Tracking

El cliente escucha `gameEventTriggered` y por cada evento `CEventNetworkEntityDamage` con la victima siendo el propio ped:

1. `GetPedLastDamageBone(ped)` → obtiene el hueso golpeado (mas confiable que `data[10]`).
2. `Config.Injuries.WeaponDamageTypes[weapon]` → tipo de daño.
3. `injuries[boneId][type] += damageAmount`.
4. Sync debounced (500ms) al server via `:server:syncInjuries`.
5. El server replica al statebag `Player(src).state.nb_injuries` para lectura cross-script.

---

## Efectos de movimiento

Mientras **cualquier hueso de pierna o pie** tenga `fracture` o `bleeding` >= severity `major`:

- `SetPedMovementClipset(ped, 'move_m@injured')` — animacion de cojera.
- `SetPedMoveRateOverride(ped, 0.7)` — velocidad reducida.

Al curar las heridas relevantes (o respawnear), se restablecen a la normalidad.

---

## Bleed tick

Cada `Config.Injuries.Bleed.IntervalMs` (8s default), si **algun hueso** tiene `bleeding > minor`:

- `damage = Config.Injuries.Bleed.Damage` (4 default).
- Si severidad `extreme`, daño x2.
- `SetEntityHealth(ped, hp - damage)`.

Si la hp llega a 0, el flujo de muerte normal toma el control (entra en laststand). El tick **no se aplica** mientras el jugador ya esta en laststand/dead (no tiene sentido matarlo dos veces).

---

## Tratamiento

### Auto-aplicacion (paciente con item)

El paciente puede usar items de su propio inventario para curarse. Conexion con ox_inventory mediante `data/items.lua`:

```lua
['bandage'] = {
    label  = 'Bandage',
    weight = 50,
    server = { export = 'nb-ambulance.use_bandage' }
},
```

Otros inventarios disparan el evento generico:

```lua
TriggerServerEvent('nb-ambulance:server:useHealItem', 'bandage')
```

El server consume el item y aplica `:client:applyTreatment` en el paciente con auto-targeting (cura el hueso con peor daño del tipo correspondiente).

### Aplicacion del paramedico (NUI)

```
/treat [id]
```

Sin id, el script captura al jugador mas cercano dentro de 4m. La NUI muestra:

- Una **card por hueso dañado**, con cada tipo de daño activo.
- Severidad coloreada (`minor` → warning, `major` → warning, `extreme` → danger).
- **Botones de items** por tipo de daño — solo aparecen los items que cura ese tipo.
- Boton refresh para volver a consultar el estado del paciente.

Al hacer click en un item:

1. Cliente: optimistic update de la NUI (resta el `amount` localmente).
2. Cliente: `NUI.post('treat', { target, boneId, item })` → server.
3. Server: valida item + permisos + Bridge.HasItem → `Bridge.RemoveItem`.
4. Server: `:client:applyTreatment` al paciente con bone + heals + amount.
5. Paciente client: decrementa el contador del hueso. Si todos los tipos llegan a 0, el hueso desaparece de la lista.
6. NUI re-fetch (250ms despues) para converger con el server.

### Item especial: medikit

`heals = 'all'` limpia TODAS las heridas del paciente y restaura `restoreHp = 100` hp. Util para emergencias pero pesado (500g por defecto).

---

## Items de heridas (Config.Injuries.Items)

| Item | Cura | Cantidad | Notas |
|------|------|----------|-------|
| `splint` | fracture | 100 | Cura fractura completamente en 1 uso. |
| `bandage` | bleeding | 50 | Sangrado leve. 2 usos = bleeding mayor curado. |
| `tourniquet` | bleeding | 100 | Sangrado mayor en 1 uso. |
| `burn_dressing` | burn | 100 | Quemaduras. |
| `oxygen_mask` | suffocating | 100 | Asfixia. |
| `medikit` | all | 100 | Limpia todo + 100 hp. |

Renombra los items en `Config.Injuries.Items` si tu economia tiene otros nombres.

---

## Exports relacionados

```lua
-- cliente
exports['nb-ambulance']:getInjuries()       -- tu propia tabla
exports['nb-ambulance']:hasInjuries()       -- bool
exports['nb-ambulance']:clearInjuries()     -- limpiar

-- server
exports['nb-ambulance']:getInjuries(src)    -- snapshot del cliente
exports['nb-ambulance']:clearInjuries(src)  -- forzar limpieza
exports['nb-ambulance']:useHealItem(src, itemName)  -- aplicar item programaticamente
```
