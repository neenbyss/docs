# Configuracion

Todas las opciones viven en `config.lua`.

---

## General

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.Language` | Idioma (`'en'`, `'es'`, `'fr'`) | `'es'` |
| `Config.Debug` | Prints de debug en consola | `false` |
| `Config.AdminCommand` | Comando que abre el panel | `'gangzones'` |

---

## Rotacion de zonas

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.ActiveZones` | Cantidad de zonas activas en paralelo | `4` |
| `Config.NextZoneDelay` | Segundos tras capturar antes de reaparecer una zona | `10800` (3h) |

---

## Mecanica de captura

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.PhaseDuration` | Segundos que dura cada fase | `40` |
| `Config.PhaseCooldown` | Segundos entre fase y fase | `60` |
| `Config.PhasesToCapture` | Fases consecutivas necesarias para capturar | `3` |
| `Config.CheckInterval` | ms entre chequeos client-side de posicion | `2000` |
| `Config.ServerTickInterval` | segundos entre validaciones server-side (authoritative) | `3` |

> El servidor es autoritativo. El cliente detecta entrar/salir en `CheckInterval`, pero el servidor re-valida cada `ServerTickInterval` para evitar spoofing.

---

## Blips

| Opcion | Descripcion | Por defecto |
|--------|-------------|-------------|
| `Config.DefaultBlipColor` | Color del blip (id GTA) | `0` (blanco) |
| `Config.BlipFlashSpeed` | Velocidad de parpadeo en ms | `500` |

---

## Funciones de integracion con tu sistema de gangs

Al final de `config.lua` hay tres funciones que **debes adaptar** a tu setup. Sin esto el script arranca pero la recompensa no se deposita y las gangs no se detectan.

```lua
-- Client: devuelve el nombre de la gang del jugador local (o nil si no tiene)
function GetPlayerGangClient()
    -- Ejemplo con nb-jobscreator:
    return exports['nb-jobscreator']:GetPlayerGang()
end

-- Server: devuelve la gang del jugador { name, label } (o nil)
function GetPlayerGangServer(src)
    -- Ejemplo con nb-jobscreator:
    return exports['nb-jobscreator']:GetPlayerGang(src)
end

-- Server: deposita la recompensa en la cuenta de la gang
function AddGangMoney(gangName, amount)
    -- Ejemplo ESX (society):
    TriggerEvent('esx_addonaccount:getSharedAccount', 'society_' .. gangName, function(acc)
        if acc then acc.addMoney(amount) end
    end)

    -- Ejemplo QBCore (qb-management):
    -- exports['qb-management']:AddMoney(gangName, amount)
end
```

Mas ejemplos concretos en [Integracion](integracion.md).

---

## Idiomas

Los strings viven en `locales.lua` bajo `Locales['en']`, `Locales['es']`, `Locales['fr']`. Para anadir un idioma nuevo:

1. Duplica uno de los bloques en `locales.lua` con la nueva clave (`Locales['de']`).
2. Pon `Config.Language = 'de'`.

Si una clave falta en el idioma activo, cae a `en` como fallback.
