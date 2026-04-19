# Panel staff

Se abre con `/restaurants` por un jugador que pertenece al job de algun restaurante. El sistema detecta automaticamente el restaurante y filtra la vista.

Tres pestanas.

---

## Overview

Resumen del restaurante:

- Logo.
- Nombre + job.
- Badge del autoservicio (**Open** / **Closed**) con toggle inline (si eres boss).
- Estado de las integraciones detectadas.

---

## Marcadores

CRUD polimorfico. Cada `kind` tiene su propio formulario dinamico.

### Comunes a todos

| Campo | Descripcion |
|-------|-------------|
| **Tipo** | `boss_menu`, `billing_desk`, `crafting_station`, `self_service`, `warehouse`, `supplier`. |
| **Etiqueta** | Visible en el help text al acercarse. |
| **Coords X/Y/Z** | Posicion del marker. Boton **"Usar mi posicion"** auto-rellena. |
| **Activo** | Toggle para deshabilitar sin borrar. |

### Por kind

| Kind | Params |
|------|--------|
| `boss_menu` | — |
| `billing_desk` | `default_category` (categoria sugerida por defecto). |
| `crafting_station` | `station_type` (matchea con el mismo campo en las recetas). |
| `self_service` | `slots` (cuantos slots caben en el expositor). |
| `warehouse` | `slots`, `max_weight` (overrides de `Config.Warehouse`). |
| `supplier` | `mode`: `native` con `items[]` (list de `{ item, price, payment }`), o `nb-shops` con `shop_id`. |

### Rangos permitidos

`access_grades` es opcional. Si lo dejas vacio, lo ven todos los miembros del job. Si pones `[2, 3]`, solo grados 2 y 3.

---

## Recetas

Tres acciones:

### Importar plantilla

Boton siempre visible. Abre el picker con las plantillas de admin.

Al importar:

1. Se duplica la plantilla a `nb_restaurants_recipes` ligada al restaurante.
2. Se clonan los ingredientes.
3. El staff puede editarla si `allow_owner_recipes = true`.

### Crear receta (si `allow_owner_recipes = true`)

Editor con los mismos campos que las plantillas de admin. El `created_by` queda en la fila para auditoria.

### Editar / eliminar

Solo visible si `allow_owner_recipes = true` y eres boss. Si el admin creo la receta directamente, tambien puede editarla.

---

## Warehouse

No es una pestana propia — es el marker `warehouse`. Pulsar E en el sitio abre el inventario real (stash registrado en tu inventario activo). Todo lo que meta el staff ahi queda disponible como ingrediente para las recetas con `source = warehouse`.

---

## Empleados & sociedad

Se gestionan desde el **boss menu** (marker `boss_menu`), no desde el panel `/restaurants`.

Ver [Flujos](flujos.md) para el detalle del boss menu.
