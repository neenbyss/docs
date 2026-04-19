# Acciones disponibles

El panel muestra solo las acciones que cumplen:

1. El jugador tiene permiso (ver [Permisos](permisos.md)).
2. El contexto lo permite (no puedes arrestar a alguien ya esposado, no puedes registrar a un muerto con la accion de vivos, etc.).

---

## Sobre jugadores

### `handcuff` — Esposar

Esposa al target. Si `Config.Handcuffs.RequireItem = true`, consume un item del inventario.

- Animacion encadenada (script pone esposas, target queda restringido).
- Estado client-side persistente (ver `IsPlayerHandcuffed`).
- Segundo uso = quitar esposas.

### `drag` — Arrastrar

Engancha al target detras de ti — se queda pegado hasta que lo sueltes.

- Solo funciona si el target esta esposado.
- Segundo uso = soltar.

### `putInVehicle` — Meter en vehiculo

Sienta al target en el asiento trasero mas cercano libre (vehiculo debe estar a <5m y tener plazas).

### `takeFromVehicle` — Sacar del vehiculo

Saca al target del vehiculo donde este sentado.

### `searchPlayer` — Registrar (vivo)

Fuerza abrir el inventario del target. El target ve una animacion de cacheo.

### `searchDeadPlayer` — Registrar (muerto)

Fuerza abrir el inventario de un target con `IsEntityDead`. Util para forense / EMS.

---

## Consultas (identidad / licencias)

### `checkIdentity` — Comprobar identidad

Devuelve nombre + apellidos + fecha de nacimiento + sexo desde el provider de licencias detectado por nb-bridge (`Bridge.GetIdentity`).

### `checkDriverLicense` — Carnet de conducir

Boolean + label. Si el provider lo soporta, tambien informa de suspensiones.

### `checkWeaponLicense` — Licencia de armas

Boolean + label.

---

## Consultas (vehiculos)

### `checkVehicleOwner` — Dueno del vehiculo

Busca por matricula en la tabla de vehiculos del framework (`owned_vehicles` en ESX, `player_vehicles` en QBCore) y muestra el nombre del propietario (o "Sin dueno" si es NPC/no existe).

---

## Filtros automaticos

El UI filtra las acciones en tiempo real:

| Escenario | Acciones ocultas |
|-----------|------------------|
| Target vivo | `searchDeadPlayer` |
| Target muerto | `handcuff`, `drag`, `putInVehicle`, `takeFromVehicle`, `searchPlayer` |
| Target ya esposado | `handcuff` cambia a "Quitar esposas" |
| Target no dragable | `drag` oculto si no esta esposado |
| Sin vehiculo cerca | `putInVehicle`, `checkVehicleOwner` |

---

## Selector multiple

Cuando hay varios jugadores/vehiculos en el radio, el panel muestra primero una **lista de targets** con nombre + distancia. Tras elegir uno, aparecen las acciones.

Atajos:

- **Numeros 1-9** — seleccionar el target N de la lista.
- **ESC** — cerrar el panel.

---

## Notificaciones

Cada accion dispara notificaciones a ambos lados — ejecutor y target — usando `Bridge.Notify` / `Bridge.ShowNotification`. Los textos se personalizan en `shared/locale.lua`.
