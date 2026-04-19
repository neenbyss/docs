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
