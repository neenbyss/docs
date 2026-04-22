# Instalacion

Pasos para instalar y poner en marcha nb-jobmanagers en tu servidor FiveM.

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Servidor con artifacts recientes (5181+) |
| **oxmysql** | Recurso para MySQL/MariaDB |
| **nb-bridge** | Bridge de framework (ESX/QBCore) |
| **Framework** | ESX Legacy (1.9.0+) o QBCore |

---

## 1. Instalar el recurso

1. Coloca la carpeta **nb-jobmanagers** dentro de `resources` (o dentro de una carpeta tipo `[neenbyss]/nb-jobmanagers`).
2. Asegurate de que **oxmysql** y **nb-bridge** esten instalados y configurados.

---

## 2. Base de datos

Importa el esquema SQL que se encuentra en la carpeta `[sql]` del recurso:

```bash
mysql -u usuario -p nombre_base_datos < nb-jobmanagers/[sql]/nb_jobmanagers.sql
```

Esto crea las siguientes tablas:

| Tabla | Funcion |
|-------|---------|
| `nb_job_markers` | Markers de cada trabajo (bossmenu, garaje, stash, duty, vestuario) |
| `nb_society_money` | Saldo de dinero por sociedad/trabajo |
| `nb_society_transactions` | Historial de depositos, retiros y pagos |
| `nb_invoices` | Facturas emitidas entre trabajos y jugadores |
| `nb_job_garage_vehicles` | Vehiculos de sociedad por garaje |
| `nb_job_outfits` | Uniformes/outfits guardados por trabajo |
| `nb_player_jobs` | Trabajos asignados a cada jugador (Multi-Job) |
| `nb_job_autoselect` | Override de visibilidad en "Trabajos Disponibles" |

Ademas, se modifican las tablas del framework (`jobs` y `job_grades`) para agregar las columnas `type`, `whitelisted`, `actions` y `permissions` si no existen, y se convierten a `utf8mb4` antes de anyadir las FKs. **Importa siempre `[sql]/nb_jobmanagers.sql` despues del SQL nativo** de tu framework para evitar `errno 150` al crear las claves foraneas.

---

## 3. Configuracion minima

Edita **shared/config.lua** en la raiz del recurso:

```lua
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
Config.Command = 'jobmanager'
Config.Locale = 'es'  -- o 'en'
```

El resto de opciones tienen valores por defecto. Ver [Configuracion](configuracion.md).

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended   # o qb-core
ensure nb-bridge
ensure nb-jobmanagers
```

El orden importa: **nb-bridge** debe cargar antes que **nb-jobmanagers**.

!!! warning "QBCore: contrato de arranque"
    En QBCore, nb-jobmanagers construye la tabla `QBCore.Shared.Jobs` desde la base de datos **antes** de marcarse como iniciado. Esto requiere que `qb-core`, `oxmysql` y `nb-bridge` esten listos primero. El orden exacto es `qb-core` → `oxmysql` → `nb-bridge` → `nb-jobmanagers`. Si los inviertes, resources que cachean `QBCore.Shared.Jobs` al arrancar pueden leer datos vacios u obsoletos.

---

## 5. Comprobar que funciona

1. Entra al servidor con un personaje que tenga un grupo de admin configurado en `Config.AdminGroups`.
2. Usa el comando `/jobmanager` para abrir el panel de administracion.
3. Crea un trabajo de prueba con al menos un grado.
4. Agrega un marker de tipo `bossmenu` y otro de tipo `garage` al trabajo.
5. Con otro personaje (o asignandote el trabajo), acercate al marker del boss menu y abrelo.
6. Verifica que puedes depositar/retirar dinero y gestionar empleados.

Si algo falla, revisa la consola del servidor y del cliente (F8). Activa `Config.Debug = true` para logs detallados.
