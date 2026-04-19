# Exports y eventos

Api para integrar nb-shops con otros recursos.

---

## Exports (server)

### getShops

```lua
local shops = exports['nb-shops']:getShops()
```

Devuelve todas las tiendas desde la cache interna (array). Cada tienda incluye sus metadatos pero **no** sus items — para eso usa `getShopItems`.

### getShopItems

```lua
local items = exports['nb-shops']:getShopItems(shopId)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `shopId` | number | Id de la tienda en `nb_shops`. |

Devuelve los items de una tienda. Util para tiendas custom integradas en otros scripts.

---

## Callbacks (server -> client)

Registrados con `Bridge.CreateCallback` de nb-bridge.

### nb-shops:getRegisteredItems

```lua
Bridge.TriggerServerCallback('nb-shops:getRegisteredItems', function(items)
    -- items: { { name, label, image }, ... }
end)
```

Devuelve todos los items registrados en el inventario activo (ox / qb / qs / origen) con su icono ya resuelto. Usado internamente por el panel admin para el item picker.

### nb-shops:getShopData

```lua
Bridge.TriggerServerCallback('nb-shops:getShopData', function(data)
    -- data: { shop, items, imageMap }
end, shopId)
```

Para abrir la tienda desde otro script sin usar el marker por defecto.

---

## Eventos del servidor (disparados por el cliente)

| Evento | Payload | Descripcion |
|--------|---------|-------------|
| `nb-shops:server:requestShops` | — | Cliente pide la lista de tiendas al cargar. |
| `nb-shops:server:requestAdminData` | — | Admin pide el dataset completo para el wizard. |
| `nb-shops:server:purchaseItems` | `shopId, cart, paymentMethod` | Ejecuta la compra (valida acceso, stock, fondos). |
| `nb-shops:server:createShop` | `data` | Crea una tienda (admin only). |
| `nb-shops:server:updateShop` | `shopId, data` | Actualiza una tienda (admin only). |
| `nb-shops:server:deleteShop` | `shopId` | Elimina una tienda (admin only, cascade items). |
| `nb-shops:server:addItem` | `shopId, item` | Anade un item a una tienda (admin only). |
| `nb-shops:server:updateItem` | `shopId, itemId, item` | Actualiza un item (admin only). |
| `nb-shops:server:removeItem` | `shopId, itemId` | Elimina un item (admin only). |
| `nb-shops:server:restockItem` | `shopId, itemId` | Resetea `current_stock` al `stock` del item. |
| `nb-shops:server:restockShop` | `shopId` | Restock de todos los items no infinitos. |

---

## Eventos del cliente (disparados por el servidor)

| Evento | Payload | Descripcion |
|--------|---------|-------------|
| `nb-shops:client:syncShops` | `shops[]` | Refresca markers/blips/peds. Se dispara al crear, editar o eliminar tiendas. |
| `nb-shops:client:refreshPeds` | `shops[]` | Re-spawnea los NPCs tenderos. |
| `nb-shops:client:receiveAdminData` | `data` | Respuesta al `requestAdminData` del panel. |
| `nb-shops:client:response` | `{ success, message, data? }` | Respuesta generica para acciones admin / compras. |

---

## Ejemplo: abrir tienda desde una mision

```lua
-- client
RegisterNetEvent('my-missions:rewardShop', function(shopId)
    Bridge.TriggerServerCallback('nb-shops:getShopData', function(data)
        SendNUIMessage({ action = 'openShop', data = data })
        SetNuiFocus(true, true)
    end, shopId)
end)
```

---

## Ejemplo: crear una tienda programaticamente

```lua
-- server
TriggerEvent('nb-shops:server:createShop', {
    name = 'Tienda Evento',
    type = 'civil',
    coords = vector3(24.5, -1347.3, 29.5),
    use_marker = true, use_ped = false, use_blip = true,
    categories = { 'comida', 'bebidas' },
    payment_methods = { 'cash' },
}) -- source = 0 actua como admin
```

> Llamado desde `source = 0` (consola), el check `Bridge.IsAdmin(0)` asume permisos de sistema en la mayoria de frameworks.

---

## Gotchas

- **`getShopItems`** no filtra por stock — devuelve items con `current_stock = 0`. Filtralo tu si quieres ocultarlos.
- **Item picker** depende del inventario: si usas un inventario no soportado directamente, implementa un override en `bridge/inventory.lua` que devuelva `{ name, label, image }[]`.
- **Cache**: las ediciones desde el panel actualizan cache + DB. Si editas la tabla `nb_shops` a mano, ejecuta `exports['nb-shops']:reload()` (no expuesto actualmente — reinicia el recurso).
