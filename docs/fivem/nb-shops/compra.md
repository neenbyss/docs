# Experiencia de compra

Como funciona la tienda desde el punto de vista del jugador.

---

## 1. Localizacion

La tienda se presenta con hasta 3 elementos opcionales:

- **Blip en el mapa** — color segun tipo (civil / business / illegal).
- **Marker en el suelo** — animado, visible a 15 metros por defecto.
- **NPC tendero** — con escenario (`WORLD_HUMAN_STAND_IMPATIENT`) para que no parezca un maniqui.

Si una tienda es **business** o **illegal** y el jugador no cumple la restriccion, no vera ni marker ni ped (pero si ve el blip, configurable).

---

## 2. Abrir la tienda

Cuando el jugador esta a < 2 metros del marker, aparece la indicacion de pulsar **E** (configurable en `Config.Marker.InteractionKey`). El cliente:

1. Llama a la callback `nb-shops:getShopData(shopId)`.
2. Recibe `{ shop, items, imageMap }`.
3. Abre la NUI y pone focus.

> `imageMap` traduce cada `item_name` a la URL NUI final (`nui://<inventory>/images/<item>.png`), respetando los overrides de `client.image` definidos en tu inventario.

---

## 3. Navegar

- **Sidebar** — lista de categorias definidas al crear la tienda.
- **Grid** — items de la categoria seleccionada. Cada card tiene icono, nombre, precio y stock disponible.
- **Detalle** — click en un item para ver su descripcion y elegir cantidad.

---

## 4. Carrito

- Se puede sumar N items antes de pagar.
- El carrito acumula **por metodo de pago** cuando hay varios (ej. la tienda acepta `cash` y `bank` — el jugador elige al checkout).
- Si un item usa **otro item como moneda** (ej. 1 `gold_coin` por unidad), el carrito detalla la cantidad de monedas requeridas.

---

## 5. Checkout

Pulsar **"Pagar"** envia:

```lua
TriggerServerEvent('nb-shops:server:purchaseItems', shopId, cart, paymentMethod)
```

El servidor valida en este orden:

1. **Acceso** — `civil` / `business` con `Bridge.GetJob(src)` / `illegal` con `Bridge.GetGang(src)`.
2. **Stock** — para cada item no infinito, `current_stock >= count`.
3. **Fondos** — `Bridge.GetMoney(src, paymentMethod)` si es cash/bank, `Bridge.HasItem(src, item_price, item_price_amount * count)` si es item-como-moneda.
4. **Capacidad** — `Bridge.CanCarry(src, item_name, count)` (lo que soporte el inventario).

Si todo OK:

- `Bridge.RemoveMoney` (o `Bridge.RemoveItem` para item-moneda).
- `Bridge.AddItem` con el item comprado.
- Actualiza `current_stock` en BD.
- Notificacion de exito al jugador.
- Si alguien mas esta dentro de la tienda, le llega la actualizacion de stock.

Si algo falla, la compra se anula completa (no se descuenta nada) y el jugador recibe `{ success = false, message = Locale('<razon>') }`.

---

## 6. Anti-bypass

- **Doble validacion** — el client no envia "ya pague": envia un cart y el server calcula el total.
- **Permisos** — validados en el servidor aunque el client marker este escondido.
- **Stock atomico** — el `UPDATE ... WHERE current_stock >= ?` evita compras simultaneas sobre el ultimo unit.

---

## 7. Cerrar la UI

- **ESC** — cierra.
- **Salirse del marker** a mas de `Config.Marker.InteractionDistance * 2` — cierra automaticamente.
- **Reiniciar el recurso** — se envia un `NUI: close` forzado al restart.
