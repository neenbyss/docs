# Permisos

nb-jobmanagers usa un sistema de permisos **bitwise** (por bits). Cada grado de un trabajo tiene un valor entero que codifica todos sus permisos. Esto permite control granular sin tablas extra.

---

## Como funciona

Cada permiso es una potencia de 2. Se combinan sumandolos. Por ejemplo, un grado con permisos de **boss menu** (1) + **crear facturas** (2) + **invitar empleados** (4) tiene un valor de `7`.

Los permisos se almacenan en la columna `permissions` de la tabla `job_grades`.

---

## Permisos de grado

| Permiso | Valor | Descripcion |
|---------|-------|-------------|
| `BOSS_MENU` | 1 | Puede abrir el boss menu, depositar y retirar dinero |
| `INVOICE_CREATE` | 2 | Puede crear facturas a jugadores |
| `EMPLOYEE_INVITE` | 4 | Puede invitar jugadores al trabajo |
| `EMPLOYEE_MANAGE` | 8 | Puede promover, degradar y despedir empleados |
| `HANDCUFF` | 16 | Puede esposar jugadores |
| `SEARCH_CUFFED` | 32 | Puede registrar jugadores esposados |
| `SEARCH_DEAD` | 64 | Puede registrar cuerpos |
| `SEARCH_GESTUAL` | 128 | Puede registrar por gesto |
| `SEARCH_ALL` | 256 | Puede registrar a cualquier jugador |
| `VEHICLE_CHECK` | 512 | Puede verificar propietario de vehiculo |
| `VEHICLE_REPAIR` | 1024 | Puede reparar vehiculos |

---

## Acciones de trabajo (job actions)

Ademas de los permisos por grado, cada **trabajo** tiene un campo `actions` que define que capacidades tiene el trabajo en general. Un grado solo puede usar un permiso si el trabajo tambien tiene la accion correspondiente habilitada.

| Accion | Valor | Descripcion |
|--------|-------|-------------|
| `HANDCUFF` | 1 | El trabajo puede esposar |
| `SEARCH_CUFFED` | 2 | El trabajo puede registrar esposados |
| `SEARCH_DEAD` | 4 | El trabajo puede registrar cuerpos |
| `SEARCH_GESTUAL` | 8 | El trabajo puede registrar por gesto |
| `SEARCH_ALL` | 16 | El trabajo puede registrar a cualquiera |
| `VEHICLE_CHECK` | 32 | El trabajo puede verificar vehiculos |
| `VEHICLE_REPAIR` | 64 | El trabajo puede reparar vehiculos |

---

## Ejemplo: configurar un trabajo de policia

1. **Trabajo** (`jobs.actions`): Habilitar esposar + registrar esposados + verificar vehiculos = `1 + 2 + 32 = 35`
2. **Grado Recluta** (`job_grades.permissions`): Solo esposar = `16`
3. **Grado Jefe** (`job_grades.permissions`): Boss menu + facturas + invitar + gestionar + esposar + registrar + verificar vehiculo = `1 + 2 + 4 + 8 + 16 + 32 + 512 = 575`

Todo esto se configura visualmente desde el panel admin NUI al crear o editar grados.

---

## Funciones de ayuda

Disponibles en `shared/permissions.lua` para uso en scripts:

```lua
-- Verificar si un permiso esta activo
HasPermission(permissions, Permissions.BOSS_MENU)  -- true/false

-- Agregar un permiso
local nuevo = AddPermission(permisos_actuales, Permissions.INVOICE_CREATE)

-- Quitar un permiso
local nuevo = RemovePermission(permisos_actuales, Permissions.HANDCUFF)

-- Verificar accion de trabajo
HasAction(actions, Actions.HANDCUFF)  -- true/false
```
