# Configuración

Archivo: `shared/config.lua`.

---

## General

| Parámetro | Descripción | Por defecto |
|-----------|-------------|-------------|
| `Config.AdminCommand` / `Config.AdminCommandAlias` | Comandos que abren el creador admin | `warehouseadmin` / `whadmin` |
| `Config.ManageCommand` / `Config.ManageCommandAlias` | Comandos que abren el panel de gestión del dueño (contraseña, renta, nivel) | `warehousemanage` / `whmanage` |
| `Config.Types` | Lista extensible de tipos de almacén (`{name, label}`) | `business`, `illegal` |
| `Config.DefaultRadius` | Radio de interacción por defecto (metros) | `3.0` |
| `Config.Stash` | Slots/peso globales usados como fallback cuando no se puede aplicar capacidad por-almacén | `50` slots / `100000` peso |
| `Config.PoliceJobs` | Nombres de job contados como "policía" para `min_police_online` | `{'police', 'sheriff'}` |
| `Config.PurchaseMoneyType` | Cuenta debitada en compras (almacenes públicos con `price > 0` y compra/renta de almacenes de jugador) | `bank` |
| `Config.Logs.Webhooks` | Webhooks de Discord para `Bridge.log.createLog` (dejar vacío y setear por servidor) | `''` |

---

## Riesgo de robo (alarma clásica, solo `type='illegal'`)

`Config.IllegalDefaults` — `alarmChance`, `policeNotifyChance`,
`robberyCooldown`, `minPoliceOnline`. Cada almacén puede sobreescribir
cualquiera de esas claves individualmente desde la pestaña **Risk** del
creador.

---

## Renta

`Config.Rent` — `DefaultPeriodHours`, `SweepIntervalMinutes`,
`GracePeriodHours`, `WarnBeforeHours`.

---

## Niveles

`Config.Levels[2..MaxWarehouseLevel]` — `cost`, `slotsBonus`, `weightBonus`,
`alarmReduction` por nivel.

---

## Robo server-validado (cualquier tipo, `robbable=true`)

`Config.RobberyDefaults` — `cooldownMinutes`, `minPoliceOnline`,
`alarmChance`, `policeNotifyChance`, `skillCheckByLevel`, `minSkillCheckMs`,
`successChanceByDifficulty`, `levelSuccessPenalty`.

Cada almacén puede sobreescribir el subconjunto que expone la pestaña
**Risk** del creador (`cooldownMinutes`, `minPoliceOnline`, `alarmChance`,
`policeNotifyChance`). `minSkillCheckMs` y `successChanceByDifficulty` son
globales por dificultad (`easy`/`medium`/`hard`) — ver
**[Exports](exports.md)** y el detalle del mecanismo en el README del
repositorio para la fórmula completa.
