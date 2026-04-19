# Configuracion

Toda la config vive en `shared/config.lua`. Lo referente a cada **garaje individual** (coords, tipo, restricciones) se gestiona desde el panel y vive en BD.

---

## General

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.AdminGroups` | Grupos con acceso a comandos admin | `{ 'admin', 'superadmin', 'god' }` |
| `Config.Debug` | Prints de debug | `false` |
| `Config.Locale` | Idioma (`'en'` / `'es'`) | `'es'` |

---

## Comandos

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Garage.CommandCreator` | Comando del panel admin | `'garagecreator'` |
| `Config.Commands.GiveCar` | Dar coche a un jugador | `'givecar'` |
| `Config.Commands.MyVehicles` | Listar mis coches | `'myvehicles'` |
| `Config.Commands.ManageVehicles` | Gestionar coches de otro | `'managevehicles'` |

---

## Minigames (para hotwire / lockpick / carjack)

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Minigame.Engine` | Motor global (`'native'`, `'ox_lib'`, `'qb-minigames'`, `'ps-ui'`, `'none'`) | `'native'` |
| `Config.Keys.Lockpick.Engine` | Override para lockpick (`nil` = global) | `nil` |
| `Config.Keys.Carjack.Engine` | Override para carjack | `nil` |
| `Config.Keys.Hotwire.Engine` | Override para hotwire | `nil` |

---

## Persistencia

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Persistence.Enabled` | Persistencia de vehiculos on/off | `true` |
| `Config.Persistence.ServerSideSpawn` | Usar `CreateVehicleServerSetter` (evita despawn) | `true` |
| `Config.Persistence.CullingRadius` | Distancia de culling entity | `5000.0` |
| `Config.Persistence.SaveOnLock` | Guardar posicion al cerrar con llave | `true` |
| `Config.Persistence.RecoverFee` | Coste para re-spawnear abandonado | `500` |
| `Config.Persistence.RespawnCheckInterval` | ms entre chequeos de vehiculos borrados | `10000` |
| `Config.Persistence.MovementTracking.Enabled` | Detectar movimiento para no impoundar a idle | `true` |
| `Config.Persistence.MovementTracking.Interval` | Segundos entre sweeps | `300` |
| `Config.Persistence.MovementTracking.Threshold` | Metros para considerar movimiento real | `15.0` |

Ver [Persistencia e impound](persistencia.md) para detalles.

---

## Garage global

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Garage.Types` | Tipos permitidos | `{ 'public', 'society', 'job', 'depot', 'house', 'custom' }` |
| `Config.Garage.Categories` | Grupos de clases GTA (`car`, `air`, `sea`, `rig`, `all`) | ver `config.lua` |
| `Config.Garage.StoreDistance` | Distancia al punto de guardado | `8.0` |
| `Config.Garage.DefaultHeading` | Heading por defecto | `0.0` |
| `Config.Garage.SocietyMaxOut` | Max vehiculos iguales fuera en society | `3` |
| `Config.Garage.UseBlips` | Mostrar blips | `true` |
| `Config.Garage.DefaultBlip` | `{ sprite, color, scale }` | `{ 357, 3, 0.7 }` |
| `Config.Garage.UseTarget` | Usar target framework en vez de marker | `false` |
| `Config.Garage.Marker.type` | Tipo de marker | `36` (cilindro) |
| `Config.Garage.Marker.color` | RGBA | indigo |
| `Config.Garage.Marker.scale` | Tamano | `vector3(0.8, 0.8, 0.5)` |
| `Config.Garage.Marker.drawDistance` | Distancia de render | `15.0` |
| `Config.Garage.Marker.interactDistance` | Distancia para pulsar E | `2.0` |

---

## Impound / Deposito

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Depot.Enabled` | Sistema de impound on/off | `true` |
| `Config.Depot.DefaultFee` | Fee base para recuperar | `500` |
| `Config.Depot.FeesByClass` | Override por clase (`[7] = 3000` para superdeportivos) | `{}` |
| `Config.Depot.AbandonAfter` | Segundos antes de ser elegible para impound | `7200` |
| `Config.Depot.CheckInterval` | Segundos entre sweeps | `600` |
| `Config.Depot.SkipWhenOwnerOnline` | No impoundar si el owner esta conectado | `true` |
| `Config.Depot.OrphanCleanupOnStart` | Enviar huerfanos a depot al arrancar | `true` |
| `Config.Depot.ShowOnMap` | Blip del depot en el mapa | `true` |

---

## Garajes de casa (house)

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.House.Enabled` | Integracion con sistemas de casas | `false` |
| `Config.House.KeyCheckResource` | Recurso con export de llaves (ej. `qb-houses`) | `nil` |
| `Config.House.KeyCheckExport` | Nombre de la funcion export que devuelve bool | `nil` |

---

## Keys system

Ver [Sistema de llaves](llaves.md) para la tabla completa. Seccion resumida:

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Keys.Enabled` | Activar todo el sistema | `true` |
| `Config.Keys.LockKeybind` | Tecla para lock/unlock | `'L'` |
| `Config.Keys.EngineKeybind` | Tecla para toggle engine | `'G'` |
| `Config.Keys.EngineRequiresKeys` | No arrancar sin llaves | `true` |
| `Config.Keys.LockDistance` | Distancia para cerrar | `5.0` |
| `Config.Keys.Hotwire.Enabled` | Hotwire on/off | `true` |
| `Config.Keys.Lockpick.*` | Tiers por clase, chances, break chance | ver `config.lua` |
| `Config.Keys.Carjack.*` | Chances por arma, multiplicadores por tipo de driver | ver `config.lua` |
| `Config.Keys.JobShared.Enabled` | Llaves compartidas por trabajo (policia, etc.) | `false` |
| `Config.Keys.JobShared.Jobs` | `{ police = { 'police', 'police2' } }` | `{}` |

---

## Idiomas

Los textos viven en `shared/locale.lua` bajo `Locale['en']` y `Locale['es']`. Extender es igual que en el resto de recursos `nb-*`: duplica el bloque, cambia la clave, pon `Config.Locale = 'xx'`.
