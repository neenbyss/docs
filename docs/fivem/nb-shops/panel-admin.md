# Panel admin

Abre el panel con `/shops` (personalizable en `Config.Command`). El panel sirve tanto para listar tiendas existentes como para lanzar el **wizard** de creacion/edicion.

---

## Listado

- **Buscador** por nombre.
- Badges de tipo (civil / business / illegal) y contador de items.
- Acciones por tienda:
  - **Editar** — reabre el wizard pre-rellenado.
  - **Eliminar** — con confirmacion. Cascade: elimina todos los items (FK).
  - **Restock** — resetea `current_stock` al maximo (`stock`) de todos los items no infinitos.

---

## Wizard — Paso 1: Info

| Campo | Descripcion |
|-------|-------------|
| **Nombre** | Nombre visible de la tienda. |
| **Tipo** | `civil`, `business` o `illegal`. Cambia el color por defecto del marker/blip. |
| **Job / Gang** | Si el tipo es business o illegal, tags con los nombres permitidos. |
| **Categorias** | Tags libres. Se usan para agrupar items en la UI de compra. |
| **Metodos de pago** | Lista con `cash`, `bank` o nombres de items como moneda. |

---

## Wizard — Paso 2: Ubicacion

- **Coordenadas X/Y/Z** — con boton "Usar mi posicion".
- **Toggles** — `use_marker`, `use_ped`, `use_blip`.
- **Ped** — modelo (spawn name) + escenario (`WORLD_HUMAN_*`).
- **Blip** — sprite, color, escala. Preview en vivo.

> Puedes desactivar los tres toggles para hacer una tienda **invisible** accesible solo via eventos / otros scripts (util para tiendas ligadas a misiones).

---

## Wizard — Paso 3: Items

Este es el paso grande:

- **Buscador visual** (modal) con todos los items registrados en el inventario. Se muestra icono, nombre interno y label.
- Al anadir un item se rellena automaticamente el label desde tu inventario.
- Campos por item:
    - **Label** — override del nombre visible.
    - **Categoria** — elige de las categorias definidas en paso 1.
    - **Precio** — entero.
    - **Tipo de precio** — `cash`, `bank` o nombre de otro item (moneda).
    - **Cantidad del item moneda** — solo si el tipo de precio es un item.
    - **Stock** — `-1` para ilimitado, o numero. `current_stock` arranca igual a `stock`.
    - **Orden** — `sort_order` para ordenar en la UI.

---

## Wizard — Paso 4: Confirmar

Resumen de todo y boton **Guardar**. Al guardar:

1. Se hace `INSERT` (o `UPDATE`) en `nb_shops` + `nb_shop_items`.
2. Se recarga la cache interna.
3. Se envia `nb-shops:client:syncShops` a todos los clientes para actualizar markers, blips y peds sin reinicio.

---

## Restock

- **Por item** — reinicia `current_stock` al valor de `stock` (solo afecta si `stock >= 0`).
- **Por tienda** — misma operacion aplicada a todos los items no infinitos de la tienda.
- Items con `stock = -1` (infinito) no se tocan.

---

## Permisos

Cada mutacion del servidor (`createShop`, `updateShop`, `deleteShop`, `addItem`, `updateItem`, `removeItem`, `restockItem`, `restockShop`) hace un check `Bridge.IsAdmin(src)`. Si alguien llama el evento desde un cliente que no tiene permisos, el servidor responde con `{ success = false, message = Locale('no_permission') }`.

---

## Atajos de teclado

- **ESC** — cancela el paso / cierra el panel.
- **ENTER** dentro del buscador — salta al primer resultado.
