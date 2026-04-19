# Flujo de venta

El ciclo de vida de un vehiculo en nb-cardealer tiene cuatro paradas:

```
  Jugador (owner)
        |
        | (lleva el coche al sell point)
        v
  Sell Request (PENDING)
        |
        +---> REJECTED  (fin)
        |
        +---> COUNTERED -- player acepta / rechaza
        |       |
        |       +----> ACCEPTED
        |       |
        |       +----> CANCELLED (fin)
        |
        +---> ACCEPTED
                |
                v
  Stock del concesionario
                |
                | (staff elige slot + precio)
                v
          Showroom (spawn fisico)
                |
                | (cliente compra)
                v
         Vehiculo cliente (ownership)
```

---

## 1. Sell Request

El jugador conduce su vehiculo al **sell point** del concesionario. Pulsa **E**:

- Se abre el formulario **"Vender tu vehiculo"**.
- Campos:
    - **Precio que pides** (asking_price) — limitado a `Config.MaxPrice`.
    - El modelo y plate se auto-rellenan del vehiculo.
- Pulsa **Enviar peticion**.

Validaciones server-side:

1. El jugador posee el plate (`Bridge.Framework.PlayerOwnsPlate`). Configurable con `Config.EnablePlateLocks`.
2. La plate es valida (`len 1-8`, trim, uppercase).
3. `asking_price` dentro de `[1, Config.MaxPrice]`.
4. Rate limiting — 3s entre peticiones del mismo jugador.
5. Distancia servidor-valido al sell point.

Si todo OK, se inserta la fila en `nb_dealer_sell_requests` con estado `PENDING` y el vehiculo **no** se mueve — sigue en posesion del jugador.

---

## 2. Dealer action

El boss o empleados ejecutan `/nb_bossmenu` y van a la pestana **Peticiones**. Ven la lista de pendientes y pueden:

### Aceptar

- Estado pasa a `ACCEPTED`.
- Se paga al vendedor `asking_price` desde `society_<job>`.
- El vehiculo se transfiere al stock (`nb_dealer_stock`).
- El plate desaparece de `owned_vehicles`.
- Discord log (si configurado).

### Rechazar

- Estado pasa a `REJECTED`. Fin del flujo, el jugador se queda con el coche.

### Contra-oferta

- Estado pasa a `COUNTERED`.
- Se guarda `counter_price` en la fila.
- El jugador recibe un toast con la contra-oferta.

---

## 3. Jugador decide la contra

El jugador abre `/nb_myrequests` (o via notification) y ve el counter. Puede:

- **Aceptar** — mismo flow que dealer accept, pero con el `counter_price`. El plate pasa al stock, el jugador cobra.
- **Rechazar** — `CANCELLED`. Fin.

---

## 4. Stock del concesionario

Cuando un vehiculo entra en stock, los staff lo ven en `/nb_bossmenu` → pestana **Stock**. Pueden:

- **Poner en showroom** — elegir un slot + precio de venta.
- **Eliminar del stock** — descartar (sin reembolso al concesionario).

No se puede sacar vehiculos del stock al inventario personal del staff — son del concesionario.

---

## 5. Showroom

Al poner un vehiculo en el showroom:

- Se crea fila en `nb_dealer_showroom` con `slot`, `stock_id`, `price`, `coords`.
- El servidor spawnea el vehiculo en las coords (con `invincible + frozen + locked` segun config).
- El cliente dibuja un texto 3D sobre el vehiculo con el precio y el modelo.

**Persistencia**: si el servidor reinicia, se re-spawnan al arrancar.

---

## 6. Compra del cliente

Un cliente se acerca al vehiculo del showroom y pulsa **E** (a menos de `Config.MaxBuyDistance`). Se abre confirmacion:

- **Modelo / plate / precio** visibles.
- **Pagar** → se descuenta del bank del cliente.
- Locks v1.6 impiden que dos compradores simultaneos compren el mismo coche — solo uno gana.

Al pagar:

1. Se inserta el coche en `owned_vehicles` del cliente.
2. El dinero va a `society_<job>` del concesionario.
3. Se borra del `nb_dealer_stock` + `nb_dealer_showroom`.
4. El vehiculo fisico se elimina del mundo.
5. Discord log.
6. Cliente recibe las llaves (via framework — requiere que tu `nb-garages` o `qb-vehiclekeys` escuche `esx:giveInventoryItem` o similar, ver integracion con garajes).

---

## 7. Integracion con garajes

Los vehiculos comprados en showroom entran en **`owned_vehicles` (ESX)** como cualquier otro. Si usas [nb-garages](../nb-garages/index.md), el cliente los vera en sus garajes publicos automaticamente — nb-cardealer no se ocupa de las llaves ni del despliegue posterior, lo delega al framework.

Si quieres enviar el coche a un garaje especifico, edita `server/main.lua` antes de `nb_dealer_showroom` delete para llamar a:

```lua
Bridge.GiveVehicle(source, model, { plate = plate, parking = 'pillboxgarage' })
```

Esto requiere adaptar `framework.lua` si tu flujo de entrega es distinto.
