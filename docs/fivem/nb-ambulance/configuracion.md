# Configuracion

Todo en `shared/config.lua`. Esta pagina cubre cada bloque por orden.

---

## Admin / general

```lua
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
Config.Debug       = false
Config.Locale      = 'es'  -- 'en' / 'es'
Config.MedicalJobs = { 'ambulance', 'doctor', 'paramedic' }
```

`MedicalJobs` decide quien puede tomar llamadas, abrir el dispatch, recibir 911 inbound, tratar a otros y reanimar. Es la lista global; por hospital puedes restringir mas via la whitelist del creator.

---

## Comandos y keybindings

```lua
Config.Commands = {
    Distress  = '911',
    Dispatch  = 'dispatch',
    Treat     = 'treat',
    Revive    = 'revive',
    Creator   = 'creator',
    Hospitals = 'hospitals',
    Reload    = 'nbamb-reload',
    KeyBindings = {
        Dispatch = 'F6',
        Distress = '',
    },
}
```

Cualquier valor `false` o `''` desactiva ese alias (la version umbrella `/nb-ambulance <sub>` siempre queda disponible). Las teclas son **defaults** — el jugador puede rebindarlas en el menu de FiveM.

Ver **[Comandos](comandos.md)** para la tabla completa.

---

## Death system

```lua
Config.Death = {
    RespawnDelay      = 30,    -- s antes de poder respawnear con [E]
    AutoRespawnTimer  = 300,   -- s antes de respawn automatico
    RespawnControl    = 38,    -- E
    Animation         = { dict = 'dead', name = 'dead_a', flag = 1 },
    FadeMs            = 800,
    RespawnCost       = 0,     -- tarifa global (cada hospital puede sobrescribir)
    WipeInventoryOnRespawn = false,
    RemoveWeaponsOnRespawn = false,
    RestoreArmor      = false,
    RestoreHealth     = 200,
}
```

---

## Last Stand (estado intermedio)

```lua
Config.LastStand = {
    Enabled              = true,    -- false vuelve al flow Phase 1 (sin laststand)
    Duration             = 300,     -- s antes de progresar a death
    MinElapsedToGiveUp   = 30,      -- s antes de poder rendirse con [E]
    GiveUpControl        = 38,      -- E
    DistressControl      = 47,      -- G - llama 911
    Animation            = { dict = 'combat@damage@writhe', name = 'writhe_loop', flag = 1 },
}
```

Durante laststand el jugador es invencible, no recibe daño, y no puede salir hasta que el timer expire o un medico lo reanime.

---

## Fallback respawn

```lua
Config.FallbackRespawn = vec4(307.7, -1433.4, 29.9, 230.0)
```

Coordenadas usadas cuando NO hay hospitales registrados. En instalaciones limpias siempre se usa hasta que el admin cree el primer hospital con el creator.

---

## PVP-safe death

```lua
Config.Pvp = {
    Zones = {
        -- { coords = vec3(2440.0, 4970.0, 47.0), radius = 60.0, label = 'sandy_arena' },
    },
    ZoneCheckInterval     = 750,
    RestoreHealth         = 200,
    NotifyZoneTransitions = true,
}
```

Cada zona es una esfera; entrar activa el flag `nb_disableDeath` en el statebag del jugador, salir lo desactiva. Ver **[PVP-safe death](pvp.md)** para la integracion completa con scripts externos.

---

## Dispatch / 911

```lua
Config.Dispatch = {
    ExpireMs      = 900000,  -- 15 min sin actividad y la llamada se autoborra
    DistanceUnits = 'm',
}
```

---

## Revive (RCP / desfibrilador)

```lua
Config.Revive = {
    Enabled       = true,
    Range         = 2.5,         -- m, distancia maxima medico-paciente
    InteractKey   = 38,          -- E

    Cpr = {
        DurationMs    = 8000,
        SuccessChance = 0.45,
        Anim = { dict = 'mini@cpr@char_a@cpr_str', name = 'cpr_pumpchest', flag = 1 },
    },
    Defib = {
        Item          = 'defibrillator',
        DurationMs    = 5000,
        SuccessChance = 0.9,
        Anim = { dict = 'mini@cpr@char_a@cpr_def', name = 'cpr_pumpchest', flag = 1 },
    },

    Reward         = 250,        -- $ al banco del medico que reanima
    RestoreHealth  = 150,        -- hp post-revive (max 200)
}
```

El roll del exito es server-side (no se puede tampear desde cliente). El defib consume el item al iniciar el intento, success o fail.

---

## Sistema de heridas

```lua
Config.Injuries = {
    Enabled = true,
    Types   = { 'fracture', 'bleeding', 'burn', 'suffocating' },
    Severity = { minor = 1, major = 35, extreme = 70 },
    Bones = { ... },             -- 12 huesos por defecto
    WeaponDamageTypes = { ... }, -- weapon hash -> tipo de daño
    DefaultDamageType = 'bleeding',
    Movement = {
        LimpClipset   = 'move_m@injured',
        WalkSpeedMult = 0.7,
        RunDisable    = false,
    },
    Bleed = {
        IntervalMs = 8000,
        Damage     = 4,
    },
    Items = {
        { name = 'splint',         heals = 'fracture',    amount = 100 },
        { name = 'bandage',        heals = 'bleeding',    amount = 50  },
        { name = 'tourniquet',     heals = 'bleeding',    amount = 100 },
        { name = 'burn_dressing',  heals = 'burn',        amount = 100 },
        { name = 'oxygen_mask',    heals = 'suffocating', amount = 100 },
        { name = 'medikit',        heals = 'all',         amount = 100, restoreHp = 100 },
    },
}
```

Ver **[Sistema de heridas](heridas.md)** para detalles del tracking, severidad y flujo de tratamiento.
