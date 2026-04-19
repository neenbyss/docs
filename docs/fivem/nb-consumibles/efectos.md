# Efectos disponibles

Catalogo completo de efectos que se pueden encadenar en cualquier item desde el panel. Cada efecto se declara en `shared/effects_catalog.lua` con su schema; la UI genera los inputs automaticamente.

- **Side** — donde corre el handler:
    - `server` — mutacion autoritativa (dinero, inventario, metadata).
    - `client` — afecta al jugador que consume (anims, HUD, filtros).
    - `both` — escape hatch para eventos custom.

---

## Needs / Salud

### hunger

**side:** `server`

Ajusta el metadata `hunger` del jugador. Respeta el `AutoScale` de config.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `amount` | number | `25` | Cantidad a sumar/establecer (rango 0..100). |
| `operation` | enum | `add` | `add` suma, `set` establece el valor absoluto. |

### thirst

**side:** `server`. Mismos params que `hunger`, aplicado a la sed.

### stress_relieve

**side:** `server`. Dispara `hud:server:RelieveStress` (compatibilidad QB HUD) y `hud:client:RelieveStress`.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `min` | number | `2` | Minimo a aliviar. |
| `max` | number | `4` | Maximo a aliviar (aleatorio entre min y max). |

### health

**side:** `client`.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `amount` | number | `25` | Total a aplicar. |
| `operation` | enum | `add` | `add` suma, `set` fija, `regen` reparte el total en ticks. |
| `regen_seconds` | number | `0` | Duracion del regen en segundos (solo si `operation = regen`). |

### armor

**side:** `client`.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `amount` | number | `75` | Valor a aplicar. |
| `operation` | enum | `set` | `set` fija, `add` suma al valor actual. |
| `max` | number | `100` | Tope maximo. |

### oxygen_restore

**side:** `client`. Rellena el timer de aire al bucear (`SetPlayerUnderwaterTimeRemaining`).

### clear_blood

**side:** `client`. Ejecuta `ClearPedBloodDamage(PlayerPedId())`.

---

## Movimiento / Stamina

### stamina_restore

**side:** `client`. `RestorePlayerStamina(PlayerId(), 1.0)`.

### sprint_multiplier

**side:** `client`.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `value` | number | `1.3` | Multiplicador (1.0 = normal, 1.49 = meth, 1.1 = coca). |
| `duration` | number | `30` | Segundos antes de volver a 1.0. |

### super_jump

**side:** `client`. Llama `SetSuperJumpThisFrame` en un loop durante N segundos.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `duration` | number | `20` | Segundos de super salto. |

### slow_motion

**side:** `client`. Manipula `SetTimeScale`.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `duration` | number | `8` | Segundos. |
| `scale` | number | `0.5` | Factor de escala (0.1–1.0). |

### movement_clipset

**side:** `client`. Sobrescribe la animacion de caminar.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `clipset` | string | `move_m@drunk@slightlydrunk` | Clipset a cargar. |
| `duration` | number | `30` | Segundos. |

### drunk_level

**side:** `client`. Combina blur + clipset + `SetPedIsDrunk`. Encapsula el comportamiento de ESX/optionalneeds.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `level` | number | `0` | 0 (ligero) / 1 (moderado) / 2 (muy borracho). |
| `duration` | number | `60` | Segundos. |

---

## Combate / Buffs

### damage_resistance

**side:** `client`. Aumenta `SetPlayerWeaponDefenseModifier` + `SetPlayerMeleeWeaponDefenseModifier`.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `duration` | number | `20` | Segundos. |
| `reduction` | number | `0.5` | Reduccion (0..0.95). |

### invincibility

**side:** `client`. `SetPlayerInvincible(true)` durante N segundos. Usar con cuidado.

### melee_damage_mult

**side:** `client`. `SetPlayerMeleeWeaponDamageModifier`. Params: `mult` (default `1.5`), `duration`.

### weapon_damage_mult

**side:** `client`. `SetPlayerWeaponDamageModifier`. Params: `mult` (default `1.5`), `duration`.

### give_weapon

**side:** `client`. `GiveWeaponToPed` (sin persistir). Util para paracaidas y gadgets.

| Param | Tipo | Default |
|-------|------|---------|
| `weapon` | string | `GADGET_PARACHUTE` |
| `ammo` | number | `1` |

---

## Visual / Sensorial

### screen_effect

**side:** `client`. `StartScreenEffect` nativo (efectos tipo `DrugsTrevorClownsFight`).

| Param | Tipo | Default |
|-------|------|---------|
| `effect` | string | `DrugsTrevorClownsFight` |
| `duration` | number | `5` (segundos) |
| `loop` | bool | `false` |

### flash_screen

**side:** `client`. `SetFlash`. Params: `duration` (ms), `fade_in` (ms), `fade_out` (ms).

### night_vision

**side:** `client`. Activa `SetNightvision(true)` durante `duration` segundos.

### thermal_vision

**side:** `client`. Activa `SetSeethrough(true)` durante `duration` segundos.

### timecycle_modifier

**side:** `client`. Combina `SetTimecycleModifier` y `SetTimecycleModifierStrength`.

| Param | Tipo | Default |
|-------|------|---------|
| `modifier` | string | `spectator5` |
| `strength` | number | `1.0` |
| `duration` | number | `20` (segundos) |

### ragdoll_chance

**side:** `client`. Durante `duration`, cada ~1.5s hay `chance`% de ragdollear al jugador si esta corriendo.

| Param | Tipo | Default |
|-------|------|---------|
| `chance` | number | `30` (%) |
| `duration` | number | `20` (segundos) |

---

## Economia / Inventario

### money_reward

**side:** `server`. Anade `math.random(min, max)` al jugador.

| Param | Tipo | Default |
|-------|------|---------|
| `account` | enum | `cash` (o `bank`) |
| `min` | number | `50` |
| `max` | number | `500` |

### give_item

**side:** `server`. `Bridge.AddItem(src, item, count)`.

### remove_item

**side:** `server`. `Bridge.RemoveItem(src, item, count)`. Util para pedir un contenedor adicional (p.ej. consumir agua + limpiar botella vacia).

---

## Roleplay / Misc

### evidence_status

**side:** `client`. `TriggerEvent('evidence:client:SetStatus', status, duration)` — compatible con `qb-policejob` y similares.

### random_teleport

**side:** `client`. Teletransporta al jugador a una coordenada aleatoria de una lista.

| Param | Tipo | Default | Descripcion |
|-------|------|---------|-------------|
| `coords_list` | list | `[]` | Lista JSON de `"x,y,z"` strings o `{x,y,z}` objects. |
| `fade` | bool | `true` | Fade out/in de pantalla durante el tp. |

Ejemplo (lista JSON en la UI):

```json
["-1037.6,-2737.6,20.2", "24.5,-1347.3,29.5", "125.8,-1081.9,29.2"]
```

### notification

**side:** `client`. `Bridge.ShowNotification(Locale(i18n_key), type)`.

| Param | Tipo | Default |
|-------|------|---------|
| `i18n_key` | string | `used_item` |
| `type` | enum | `success` / `error` / `info` / `warning` |

### heart_rate_hud

**side:** `client`. Activa el overlay rojo pulsante del panel (`SendNUIMessage action: heartRate`). Ideal para stimulantes.

| Param | Tipo | Default |
|-------|------|---------|
| `duration` | number | `20` (segundos) |
| `intensity` | number | `1.0` |

### custom_event

**side:** `both`. Escape hatch para disparar cualquier evento.

| Param | Tipo | Default |
|-------|------|---------|
| `target` | enum | `client` / `server` |
| `event` | string | `''` |
| `args` | list | `[]` |

Ejemplos:

- `{ target: 'client', event: 'MyResource:teleport', args: ['safehouse'] }`
- `{ target: 'server', event: 'banking:deposit', args: [500] }`

---

## Extender el catalogo

Desde otro recurso puedes registrar tus propios efectos sin tocar nb-consumibles:

```lua
-- client
exports['nb-consumibles']:AddEffectHandler('gravity_low', function(params)
    -- tu logica
end)

-- server
exports['nb-consumibles']:AddServerEffectHandler('mi_efecto', function(source, params)
    -- tu logica
end)
```

Despues anade la entrada correspondiente a `EffectsCatalog` en `shared/effects_catalog.lua` para que aparezca en la UI del panel.
