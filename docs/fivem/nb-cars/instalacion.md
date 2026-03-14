# Instalación

## Requisitos

- FiveM Server
- [oxmysql](https://github.com/overextended/oxmysql)
- [ox_lib](https://github.com/overextended/ox_lib)
- Framework: ESX, QBCore o QBox

## Instalación

1. Descarga el recurso desde el panel de Tebex
2. Extrae el archivo y renombra la carpeta a `nb-cars`
3. Añade la carpeta a tu directorio de recursos
4. Añade `ensure nb-cars` a tu `server.cfg`

## Dependencias

Asegúrate de tener estas líneas en tu server.cfg:

```cfg
ensure oxmysql
ensure ox_lib
ensure nb-cars
```

## Base de datos

El script crea las tablas automáticamente:
- `nb_welcome_cars` - Registro de vehículos de bienvenida canjeados

No se requiere configuración adicional de base de datos.
