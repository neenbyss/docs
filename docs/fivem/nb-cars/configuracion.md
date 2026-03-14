# Configuración

## Configuración General

```lua
Config.Framework = 'auto' -- 'auto', 'esx', 'qbcore', 'qbox'

-- Sistema de llaves
Config.KeySystem = 'brutal_keys' -- 'brutal_keys', 'qb-vehiclekeys', 'qs-vehiclekeys', 'cd_garage', 'wasabi_carlock', 'jaksam', 'custom', 'none'

-- Permisos
Config.UseAcePermissions = false
Config.PermissionGroup = 'group.admin'

-- Comandos
Config.CommandName = 'givecar'
Config.DeleteCommandName = 'delcar'
Config.ShowcarsCommandName = 'showcars'
```

## Sistema Welcome Car

```lua
Config.WelcomeCar = {
    enabled = false,
    SetupCommandName = 'setupwelcomecar',
    
    npc = {
        coords = vector3(-45.5, -1098.5, 26.5),
        heading = 320.0,
        model = 'a_m_m_business_01',
        blip = { sprite = 225, color = 3, label = 'Vehículos Welcome' }
    },
    
    defaultGarage = 'motelgarage', -- Garage donde se guardarán los vehículos
    
    vehicles = {
        { model = 'blista', name = 'Blista', image = 'https://...' },
        { model = 'emperor', name = 'Emperor', image = 'https://...' }
    }
}
```

## Textos (Internacionalización)

Todos los textos son configurables en `Config.Lang`:

```lua
Config.Lang = {
    received_car = 'Has recibido un vehículo: %s',
    given_car = 'Has entregado un vehículo (%s) al ID %s',
    no_permission = 'No tienes permisos para usar este comando.',
    -- ... más textos
}
```

## Sistema de Permisos

### Opción 1: Usar rangos del framework (Recomendado para principiantes)

Por defecto, el script usa los rangos de admin del framework:

```lua
Config.UseAcePermissions = false
```

- **ESX**: Necesitas el grupo `admin` o `superadmin`
- **QBCore/QBox**: Necesitas el permiso `admin` o `god`

### Opción 2: Usar ACE Permissions (Recomendado para servidores avanzados)

Esta opción ignora los rangos del framework y usa permisos ACE directamente.

#### Paso 1: Habilitar ACE en config.lua

```lua
Config.UseAcePermissions = true
Config.PermissionGroup = 'group.admin' -- o el nombre que prefieras
```

#### Paso 2: Dar permisos en server.cfg

Añade estas líneas en tu `server.cfg`:

```cfg
# Dar permiso a un usuario específico
add_ace user.steam:110000100000000 group.admin allow

# Dar permiso a un rol (más común)
add_principal group.moderator group.admin
add_principal group.admin group.admin

# O crear un grupo específico para el script
add_ace group.nbgivecars allow "command.givecar"
add_ace group.nbgivecars allow "command.delcar"
add_principal group.moderator group.nbgivecars
```

#### Ejemplos prácticos

```cfg
# Ejemplo 1: Solo un admin específico tiene acceso
add_ace user.steam:110000100000000 group.admin allow

# Ejemplo 2: Todos los moderadores y admins del framework tienen acceso
add_principal group.moderator group.admin
add_principal group.admin group.admin

# Ejemplo 3: Crear grupo específico para el script (recomendado)
add_ace group.nbgivecars allow "command.givecar"
add_ace group.nbgivecars allow "command.delcar"
add_principal group.admin group.nbgivecars

# Ejemplo 4: Permiso por comando específico (más restrictivo)
add_ace group.vehicles allow "command.givecar"
add_ace group.vehicles allow "command.delcar"
add_principal identifier.steam:110000100000000 group.vehicles
```

#### Verificar que los permisos funcionan

Reinicia el servidor después de modificar `server.cfg`. Para verificar:

1. Entra al juego como el usuario con permisos
2. Ejecuta el comando `/givecar` o el comando configurado
3. Si funciona, los permisos están correctos

#### Comandos útiles para debugging

```cfg
# Ver todos los permisos de un jugador en consola
player 1 | ace check group.admin

# Listar todos los ACEs cargados
ace list
```
