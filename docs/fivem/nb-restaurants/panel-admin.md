# Panel admin

Abre con `/restaurants` como usuario con grupo admin (`Config.AdminGroups`). Cuatro pestanas.

---

## Restaurantes

CRUD completo de `nb_restaurants`.

| Campo | Descripcion |
|-------|-------------|
| **Nombre** | Visible al cliente. |
| **Job** | Obligatorio y unico por restaurante. Debe existir en el framework. |
| **Logo URL** | Opcional. Si vacio, se muestra `Config.DefaultLogo`. |
| **Modo de facturacion** | `native`, `nb-billings` o `external`. |
| **Activo** | Desactivar para pausar el restaurante sin perder datos. |
| **Dueno puede crear recetas** | Si activado, el boss puede CRUD recetas desde el panel staff. |
| **Lavado de dinero** | Habilita la pestana Lavado en el boss menu. |
| **Autoservicio abierto** | Si esta ON y el boss no lo ha cerrado, los clientes pueden comprar. |
| **Impuesto de lavado (%)** | Solo si `allow_laundering = true`. |

---

## Plantillas

Biblioteca editable de recetas que luego los staff importan a sus restaurantes.

**15 plantillas seed** (pancakes, scrambled eggs, coffee, orange juice, burger, cheeseburger, hotdog, pizza, fries, chicken wings, taco, salad, sandwich, cocktail, soft drink) con sus ingredientes. Puedes editarlas o borrar y crear nuevas.

Campos del editor de plantilla:

| Campo | Descripcion |
|-------|-------------|
| **Nombre** | Visible en el picker. |
| **Categoria** | `breakfast`, `main`, `drinks`, etc. Agrupamiento visual. |
| **Item resultante** | Nombre del item que se crea (debe existir en el inventario). |
| **Cantidad** | Cantidad entregada. |
| **Estacion** | `grill`, `fryer`, `oven`, `bar`, etc. |
| **Rango minimo** | Grade minimo para usarla. |
| **Duracion (ms)** | Tiempo del progressbar. |
| **Ingredientes** | Lista item + count + source (`warehouse` / `personal`). |

Cuando un staff importa una plantilla, se duplica a su restaurante — modificaciones posteriores no afectan a la original.

---

## Idiomas

Editor de `nb_restaurants_locales` — strings dinamicos que se pueden referenciar desde el resto del script con el prefijo `db::`.

Form inline:

- Selector de locale (`es`, `en`, o lo que anadas en `Config.PanelLanguages`).
- Campo `key`.
- Campo `value`.
- Boton `+` para guardar.

Tabla principal con busqueda, edicion inline (al cambiar el value se guarda automaticamente) y eliminacion por fila.

---

## Ajustes

### Idioma por defecto

Sobrescribe `Config.Locale` al arrancar el servidor.

### Ajustes custom

Tabla `nb_restaurants_settings` — clave/valor libre. Puedes anadir tus propios ajustes y leerlos desde exports:

```lua
local settings = Bridge.DB.GetSettings()
local mio = settings.mi_ajuste
```

### Integraciones detectadas

Vista de solo lectura con el estado de las 4 integraciones (jobmanagers, billings, shops, consumibles). Util para debugging del orden en `server.cfg`.
