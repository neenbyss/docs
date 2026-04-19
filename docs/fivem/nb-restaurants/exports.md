# Exports

API server para integrar nb-restaurants desde otros recursos.

---

## GetRestaurant

```lua
local r = exports['nb-restaurants']:GetRestaurant(id)
```

Devuelve la fila completa con toggles deserializados, o `nil`.

---

## GetRestaurantByJob

```lua
local r = exports['nb-restaurants']:GetRestaurantByJob('bahama_mamas')
```

Util cuando tienes el job del jugador y quieres saber si pertenece a algun restaurante.

---

## GetAllRestaurants

```lua
local list = exports['nb-restaurants']:GetAllRestaurants()
```

Devuelve todos los restaurantes cacheados.

---

## GetRecipes

```lua
local recipes = exports['nb-restaurants']:GetRecipes(restaurantId, stationType?)
```

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `restaurantId` | number | Id del restaurante. |
| `stationType` | string? | Filtrar por estacion (opcional). |

---

## RegisterRecipe

```lua
local id = exports['nb-restaurants']:RegisterRecipe(restaurantId, {
    name          = 'Special Burger',
    output_item   = 'special_burger',
    output_count  = 1,
    station_type  = 'grill',
    duration_ms   = 6000,
    min_grade     = 0,
    enabled       = true,
    ingredients = {
        { item = 'bun',  count = 1, source = 'warehouse' },
        { item = 'meat', count = 1, source = 'warehouse' },
    },
})
```

Persiste una receta programaticamente (sin pasar por el panel).

---

## Public API — integraciones externas

Estos exports estan disenados para ser llamados desde **cualquier recurso** (telefonos tipo `lb-phone`, apps web, paneles discord). Todos devuelven payloads "publicos": nada de identificadores de staff / clientes, solo la informacion que un jugador podria ver en la app.

### GetPublicRestaurants

```lua
local list = exports['nb-restaurants']:GetPublicRestaurants()
--[[
  [1] = {
    id = 1, name = 'Burger Shot', logo_url = '...', category = 'food',
    is_open = true, staff_online = true,
    avg_rating = 4.2, rating_count = 87,
  },
  ...
]]
```

Lista resumida de todos los restaurantes con su estado abierto/cerrado y rating medio.

### GetRestaurantInfo

```lua
local info = exports['nb-restaurants']:GetRestaurantInfo(restaurantId)
--[[
  {
    id, name, logo_url, category,
    is_open, staff_online, avg_rating, rating_count,
    recent_ratings = [
      { id, item, stars, comment, created_at },
      ...
    ],
    item_ratings = [
      { item = 'burger',       avg_stars = 4.5, count = 42 },
      { item = 'pizza',        avg_stars = 3.8, count = 12 },
      { item = '__restaurant__', avg_stars = 4.2, count = 33 },
    ],
  }
]]
```

Todo lo de `GetPublicRestaurants` + ratings recientes + rating **por plato**. El item especial `__restaurant__` agrega los ratings que no se asociaron a un plato concreto (facturas sin item, compras multi-item).

### GetRestaurantMenu

```lua
local menu = exports['nb-restaurants']:GetRestaurantMenu(restaurantId)
--[[
  [1] = {
    item = 'burger', label = 'Burger', price = 15,      -- price = con descuento aplicado
    original_price = 20, discount_pct = 25,             -- si hay promo activa
    payment = 'both',
    in_stock = true, stock = 42,
    image = 'nui://ox_inventory/web/images/burger.png',
    marker_coords = { x, y, z },
  },
  ...
]]
```

Agrega todos los slots de self_service de todos los markers del restaurante. Incluye descuentos activos al momento de la llamada.

### GetItemRatings

```lua
local ratings = exports['nb-restaurants']:GetItemRatings(restaurantId)
--[[
  [1] = { item = 'burger', avg_stars = 4.5, count = 42 },
  [2] = { item = 'pizza',  avg_stars = 3.8, count = 12 },
  [3] = { item = '__restaurant__', avg_stars = 4.2, count = 33 },
]]
```

### GetRecentRatings

```lua
local recent = exports['nb-restaurants']:GetRecentRatings(restaurantId, 10)
--[[
  [1] = { id, item, stars, comment, created_at },
  ...
]]
```

### IsRestaurantOpen

```lua
if exports['nb-restaurants']:IsRestaurantOpen(restaurantId) then ... end
```

Boolean rapido. `enabled AND self_service_open`.

### SubmitPhoneRating

Permite que una app de telefono envie un rating **en nombre del jugador**. Valida que el jugador haya comprado al menos una vez en ese restaurante (anti-review-bomb).

```lua
-- server (desde tu app de telefono)
local ok, err = exports['nb-restaurants']:SubmitPhoneRating(playerSource, {
    restaurant_id = 1,
    stars         = 5,
    item          = 'burger',   -- opcional, nil = rating de restaurante
    comment       = 'Excelente como siempre',
})
```

Retorna `(true)` en exito o `(false, 'never_purchased' | 'invalid_data' | 'ratings disabled' | 'no_identifier')`.

---

## Eventos internos

### Client -> Server

Los eventos internos llevan el prefijo `nb-restaurants:server:*` y **no** se consideran API publica estable. Si necesitas interactuar desde otro recurso, prefiere los exports o dispara el flow completo a traves de `TriggerServerEvent` con el mismo payload que envia el panel.

### Server -> Client

| Evento | Payload | Descripcion |
|--------|---------|-------------|
| `nb-restaurants:client:catalogRefreshed` | `{ restaurants, markers }` | Se dispara en hot reload. Util para que tu recurso refresque caches que dependan del catalogo. |
| `nb-restaurants:client:nativeBillPrompt` | `{ restaurantId, restaurantName, logo, amount, title, category }` | El cliente recibe una factura `native`. |

---

## Ejemplos

### Dar bonificacion al empleado mas activo del restaurante

```lua
-- server
local function bonus(restId)
    local rest = exports['nb-restaurants']:GetRestaurant(restId)
    if not rest then return end

    -- Your logic to pick the winning employee...
    local winnerIdentifier = 'license:abc...'
    local winnerSource     = GetPlayerFromIdentifier(winnerIdentifier)
    if winnerSource then
        Bridge.AddMoney(winnerSource, 'bank', 1000, 'restaurant_bonus')
        Bridge.Notify(winnerSource, ('Bonus de %s!'):format(rest.name), 'success')
    end
end
```

### Registrar una receta desde un evento custom

```lua
-- server (inside your mission script)
AddEventHandler('my-event:unlockRecipe', function(source, restId, recipeData)
    local id = exports['nb-restaurants']:RegisterRecipe(restId, recipeData)
    if id then
        Bridge.Notify(source, 'Receta nueva desbloqueada!', 'success')
    end
end)
```

---

## Ejemplo: app de telefono (lb-phone / qs-smartphone)

Una app de telefono que lista restaurantes, muestra su menu, rating y permite dejar reseña.

### Backend de la app (server.lua)

```lua
-- Llama desde la app cuando el jugador abre la lista.
RegisterNetEvent('my-phone:restaurants:list', function()
    local src = source
    local list = exports['nb-restaurants']:GetPublicRestaurants()
    TriggerClientEvent('my-phone:restaurants:list:result', src, list)
end)

-- Detalle + menu de un restaurante.
RegisterNetEvent('my-phone:restaurants:open', function(restaurantId)
    local src = source
    local info = exports['nb-restaurants']:GetRestaurantInfo(restaurantId)
    local menu = exports['nb-restaurants']:GetRestaurantMenu(restaurantId)
    TriggerClientEvent('my-phone:restaurants:open:result', src, {
        info = info,
        menu = menu,
    })
end)

-- Submit rating desde la app.
RegisterNetEvent('my-phone:restaurants:rate', function(data)
    local src = source
    local ok, err = exports['nb-restaurants']:SubmitPhoneRating(src, data)
    TriggerClientEvent('my-phone:restaurants:rate:result', src, { ok = ok, err = err })
end)
```

### UI de la app (pseudo-Vue)

```html
<div v-for="r in restaurants" :key="r.id" class="rest-card"
     :class="{ closed: !r.is_open }"
     @click="openDetail(r.id)">
    <img :src="r.logo_url" />
    <h3>{{ r.name }}</h3>
    <div class="status">
        <span v-if="r.is_open" class="open">ABIERTO</span>
        <span v-else class="closed">CERRADO</span>
    </div>
    <div class="rating">
        ⭐ {{ r.avg_rating.toFixed(1) }} ({{ r.rating_count }})
    </div>
</div>
```

### Resultado

Menu screen muestra:

- Precio con el **descuento activo aplicado** (la app pinta un tachado sobre `original_price` si `discount_pct > 0`).
- Disponibilidad real (`in_stock` por plato).
- Ratings por plato → los clientes pueden decidir si pedir el `burger` (4.8) o el `pizza` (3.1).

Submit de rating pasa el anti-abuse: el jugador **tiene que haber comprado al menos una vez** en ese restaurante. Si no, recibe `never_purchased`.
