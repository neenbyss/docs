# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Artifacts 5181+ |
| **oxmysql** | Queries |
| **nb-bridge** | Framework abstraction |
| **Framework** | ESX Legacy o QBCore |
| **Inventario** | ox_inventory (recomendado) o alternativas soportadas por nb-bridge |

---

## 1. Instalar el recurso

1. Compra nb-restaurants en [neenbyss.tebex.io](https://neenbyss.tebex.io/) y descarga el ZIP desde tu area de cliente.
2. Descomprime la carpeta en `resources/nb-restaurants/`.
3. Verifica que **nb-bridge** este instalado y arrancando antes (nb-bridge es gratuito y se descarga desde [github.com/neenbyss/nb-bridge](https://github.com/neenbyss/nb-bridge)).

---

## 2. Base de datos

Importa `[sql]/nb-restaurants.sql`:

```bash
mysql -u usuario -p nombre_db < nb-restaurants/\[sql\]/nb-restaurants.sql
```

Crea 9 tablas + seeds con **15 plantillas de recetas** listas para usar.

| Tabla | Descripcion |
|-------|-------------|
| `nb_restaurants` | Restaurantes (name, logo, job, toggles, billing_mode, laundering). |
| `nb_restaurants_markers` | Marcadores polimorficos (boss, billing, crafting, self-service, warehouse, supplier). |
| `nb_restaurants_recipes` (+ `_ingredients`) | Recetas por restaurante con ingredientes. |
| `nb_restaurants_recipe_templates` (+ `_ingredients`) | Biblioteca global editable por admin. |
| `nb_restaurants_self_service_items` | Slots del expositor con stock y precio. |
| `nb_restaurants_staff` | Fallback cuando no hay nb-jobmanagers. |
| `nb_restaurants_laundering` | Auditoria de lavado de dinero. |
| `nb_restaurants_locales` | Traducciones dinamicas. |
| `nb_restaurants_settings` | Ajustes globales clave/valor. |

El SQL es **idempotente** (`IF NOT EXISTS` + `INSERT IGNORE`). Seguro de re-ejecutar.

---

## 3. Configuracion minima

Edita `shared/config.lua`:

```lua
Config.AdminGroups  = { 'admin', 'superadmin', 'god' }
Config.Locale       = 'es'
Config.PanelCommand = 'restaurants'
Config.DefaultLogo  = 'https://i.imgur.com/5gJp0Gk.png'

-- Integraciones: 'auto' | 'off' | 'force'
Config.Integrations.JobManagers = 'auto'
Config.Integrations.Billings    = 'auto'
Config.Integrations.Shops       = 'auto'
Config.Integrations.Consumibles = 'auto'

-- Lavado de dinero: fuente del dinero sucio
Config.Laundering.DirtyMoneySource = 'item'   -- 'item' | 'cash' | 'account'
Config.Laundering.DirtyMoneyItem   = 'dirty_money'
```

Nada mas necesita tocarse en config — el resto se configura desde el panel in-game.

---

## 4. Orden en `server.cfg`

```cfg
ensure oxmysql
ensure es_extended       # o qb-core
ensure nb-bridge
ensure nb-restaurants
```

Si usas las integraciones opcionales, asegurate de que arranquen **antes** que nb-restaurants:

```cfg
ensure nb-jobmanagers
ensure nb-billings
ensure nb-shops
ensure nb-consumibles
ensure nb-restaurants
```

---

## 5. Comprobar que funciona

1. Entra al servidor como admin.
2. `/restaurants` — abre el panel admin.
3. Crea un restaurante con nombre, job, logo URL opcional.
4. Con un empleado de ese job, `/restaurants` abre el panel staff.
5. Crea un marcador `crafting_station` con coords y ve al sitio — aparece el marker en el mundo.
6. Pulsa E — se abre el crafting UI con recetas (vacio al principio). Importa plantillas desde el panel staff.

Consola server:

```
[nb-restaurants] Loaded (ESX / ox_inventory)
```
