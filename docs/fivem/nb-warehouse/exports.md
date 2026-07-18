# Exports

API de solo lectura para que otro recurso (HUD, panel de facción, overlay de
dispatch, estadísticas externas) consulte almacenes sin acoplarse al esquema
de la tabla `nb_warehouses`.

Ninguno expone la contraseña ni permite escribir estado — comprar, rentar,
subir de nivel, robar o administrar contraseña siguen siendo flujos
exclusivamente server-authoritative dentro de `nb-warehouse`. Si necesitas
iniciar uno de esos flujos desde otro recurso, dispara el evento de red
correspondiente (ver más abajo) en vez de buscar un export de escritura —
no existe, a propósito.

---

## Server

```lua
exports['nb-warehouse']:getWarehouse(warehouseId)
-- table|nil — misma forma que el broadcast :client:sync (id, name, label,
-- type, coords, radius, enabled, price, owner_type, for_sale, for_rent,
-- rent_price, robbable, has_password, blip/marker). Nunca incluye la
-- identidad del dueño más allá de owner_type, ni la contraseña real.

exports['nb-warehouse']:getWarehouses()
-- table[] — snapshot de todos los almacenes cacheados.

exports['nb-warehouse']:getWarehousesByOwner(ownerType, ownerName)
-- table[] — ownerType: 'job' | 'gang' | 'player'. ownerName: nombre de
-- job/gang, o el identifier del jugador para owner_type='player'. Devuelve
-- {} si falta cualquiera de los dos argumentos.

exports['nb-warehouse']:hasAccess(source, warehouseId)
-- bool — mismo chequeo server-authoritative que usa requestOpen (owner/
-- job+grado/gang+grado/whitelist, incluye expiración de renta). false si
-- el almacén no existe.

exports['nb-warehouse']:isManager(source, warehouseId)
-- bool — SOLO el dueño primario (mismo gate que contraseña/renovar renta/
-- subir nivel). Más estricto que hasAccess, que también acepta whitelist.

exports['nb-warehouse']:getEffectiveCapacity(warehouseId)
-- slots, weight — capacidad con el bonus de nivel ya aplicado. nil, nil si
-- el almacén no existe.
```

---

## Cliente

```lua
exports['nb-warehouse']:getWarehouses()
-- table — id -> fila pública (misma forma que el export server
-- getWarehouse). Cache local sincronizado; vacío hasta el primer
-- :client:sync (unos segundos después del arranque del recurso).

exports['nb-warehouse']:getWarehouse(warehouseId)
-- table|nil
```

---

## Evento: alerta policial

`nb-warehouse` **no** implementa un sistema de dispatch propio. Dos
mecanismos independientes pueden disparar este evento local (server-side,
no de red):

```lua
AddEventHandler('nb-warehouse:policeAlert', function(warehouseId, coords)
    -- coords es un vector3 del almacén. Decide tu propia lógica de CAD/dispatch aquí.
end)
```

1. **Alarma de apertura legítima** (almacenes `type='illegal'`, roll de
   `alarmChance`/`policeNotifyChance` al abrir normalmente).
2. **Roll de aviso al intentar un robo** (cualquier almacén `robbable=true`,
   tanto en un robo exitoso como en uno fallido).

Ver el README del repositorio (sección "Sistema de robo server-validado")
para el detalle completo de las tres condiciones que debe cumplir un intento
de robo antes de otorgar acceso.
