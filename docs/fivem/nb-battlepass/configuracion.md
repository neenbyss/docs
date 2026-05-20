# Configuracion

Todas las opciones estan en **`shared/config.lua`** (archivo publico, puedes editarlo libremente).

---

## General

```lua
Config.Debug   = false         -- Activa logs detallados con Debugger()
Config.Locale  = 'es'          -- 'en' o 'es'
Config.Command = 'battlepass'  -- Comando para abrir la UI del jugador
Config.AdminCommand = 'battlepassadmin'
Config.CommandHelp  = 'Open the Battlepass'
```

---

## Admin

```lua
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
```

Cualquier jugador con uno de estos grupos puede abrir `/battlepassadmin`.

---

## Curva de XP (defaults para nuevas seasons)

```lua
Config.DefaultMaxLevel   = 50
Config.DefaultXpPerLevel = 1000
```

!!! info "Estos valores son solo defaults"
    Cuando creas una season desde el panel admin, **estos valores se pueden sobrescribir por season**. La curva real se almacena en la tabla `nb_battlepass_seasons` (`max_level`, `xp_per_level`).

La formula de nivel es lineal: `level = floor(xp / xp_per_level)`, con cap en `max_level`.

---

## Premium Pass

Bloque mas importante porque define **como se compra/otorga** el pase premium.

```lua
Config.Premium = {
    -- Master switch. Si false, el boton "Buy Premium" no se muestra
    -- aunque la season tenga premium_purchasable = true.
    PurchaseButtonEnabled = true,

    -- Verifica si el jugador puede comprar. DEBE devolver true/false.
    -- Recibe (src, season). Aqui es donde integras con tu sistema
    -- externo de monedas (coins, donator points, etc.).
    CanPurchase = function(src, season)
        if not season or not season.premium_purchasable then return false end
        if (season.premium_price or 0) <= 0 then return false end
        local money = Bridge.Money.Get(src, 'bank') or 0
        if money < season.premium_price then
            return false, Locale('not_enough_money')
        end
        return true
    end,

    -- Se ejecuta DESPUES de que CanPurchase devuelva true.
    -- Cobra/descuenta lo que sea necesario.
    OnPurchase = function(src, season)
        Bridge.Money.Remove(src, 'bank', season.premium_price)
        return true
    end,
}
```

### Como funciona el boton "Comprar Premium"

El boton se muestra solo si **TODAS** estas condiciones se cumplen:

1. ✅ `Config.Premium.PurchaseButtonEnabled = true` (global)
2. ✅ La season activa tiene `premium_purchasable = 1` (configurable in-game)
3. ✅ El jugador **no tiene** premium todavia

Cuando el jugador clickea:

1. Cliente envia `purchasePremium` al server.
2. Server llama `Config.Premium.CanPurchase(src, season)`.
3. Si devuelve `true`, llama `Config.Premium.OnPurchase(src, season)` para cobrar.
4. Si devuelve `true`, otorga el pase premium.

### Ejemplos de personalizacion

#### Cobrar con dinero del banco (default)

Ya viene asi de fabrica, no necesitas hacer nada.

#### Cobrar con coins de otro recurso (ej. nb-vip)

```lua
Config.Premium = {
    PurchaseButtonEnabled = true,

    CanPurchase = function(src, season)
        local coins = exports['nb-vip']:GetPlayerCoins(src) or 0
        if coins < season.premium_price then
            return false, 'No tienes suficientes coins.'
        end
        return true
    end,

    OnPurchase = function(src, season)
        return exports['nb-vip']:RemovePlayerCoins(src, season.premium_price, 'battlepass_premium')
    end,
}
```

#### Solo permitir grant por admin (desactivar compra)

```lua
Config.Premium = {
    PurchaseButtonEnabled = false,  -- ningun jugador ve el boton
    CanPurchase = function() return false end,
    OnPurchase  = function() return false end,
}
```

#### Hibrido: gratis para VIPs, pago para el resto

```lua
Config.Premium = {
    PurchaseButtonEnabled = true,

    CanPurchase = function(src, season)
        if exports['nb-vip']:IsVIP(src) then
            return true  -- VIP no paga
        end
        local money = Bridge.Money.Get(src, 'bank') or 0
        if money < season.premium_price then
            return false, 'No tienes suficiente dinero.'
        end
        return true
    end,

    OnPurchase = function(src, season)
        if exports['nb-vip']:IsVIP(src) then return true end
        Bridge.Money.Remove(src, 'bank', season.premium_price)
        return true
    end,
}
```

---

## Notificaciones

```lua
Config.NotifyOnLevelUp = true   -- Avisar al subir de nivel
Config.NotifyOnReward  = true   -- Avisar al reclamar recompensa
```

Las notificaciones se envian con `Bridge.Notify(src, msg, type)` — usa el sistema configurado en nb-bridge.

---

## Lo que **no** se configura aqui

Estas opciones se configuran **in-game** desde `/battlepassadmin`, no en `Config`:

- Nombre y descripcion de cada season
- Max level y XP por nivel **especifico** de cada season
- Si esa season en particular permite compra de premium
- Precio del premium para esa season
- Recompensas de cada nivel (free + premium)

La idea es que **una vez instalado**, no necesites tocar mas archivos para crear nuevas seasons.
