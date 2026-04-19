# Flujos principales

---

## 1. Admin crea un restaurante

1. `/restaurants` como admin.
2. Tab **Restaurantes** → **Nuevo**.
3. Rellena:
    - Nombre, URL del logo (opcional), job.
    - Modo de facturacion: `native` / `nb-billings` / `external`.
    - Toggles: `enabled`, `allow_owner_recipes`, `allow_laundering`, `self_service_open`.
    - Tax de lavado (%) si activaste `allow_laundering`.
4. Guardar. El restaurante queda vacio — sin markers ni recetas.

---

## 2. Staff configura markers y recetas

El comando `/restaurants` con un personaje del job del restaurante abre el **panel staff**, filtrado automaticamente al restaurante de ese job.

### Markers

Pestana **Marcadores**. Crear uno de cada:

- **boss_menu** — donde el boss ejecuta hire/fire, sociedad, autoservicio, lavado.
- **billing_desk** — el camarero factura a clientes cercanos.
- **crafting_station** — el staff cocina recetas que matchean `station_type`.
- **self_service** — expositor de productos para clientes.
- **warehouse** — almacen compartido (stash). Se registra automaticamente via `Bridge.RegisterStash`.
- **supplier** — puede apuntar a una tienda de nb-shops o tener items nativos.

Cada marker permite definir `access_grades` (array de grados permitidos). Si lo dejas vacio, lo ven todos los miembros del job.

### Recetas

Pestana **Recetas**:

- **Importar plantilla** — abre el picker con las 15 plantillas base. Al importar, la receta se clona al restaurante como propia.
- **Crear receta** — solo disponible si el admin habilito `allow_owner_recipes`. El editor pide nombre, estacion, output, ingredientes (cada uno con fuente `warehouse` o `personal`), duracion y grado minimo.

---

## 3. Boss menu in-game

El boss (o un admin) se acerca al marker `boss_menu` y pulsa **E**. Se abre un modal con 4 pestanas:

### Staff

- Lista de empleados con grade editable inline.
- Contratar por server ID o identifier.
- Despedir.

### Sociedad

- Balance de la cuenta `society_<job>` (o equivalente via nb-jobmanagers).
- Depositar desde el bank del boss.
- Retirar al bank del boss.

### Autoservicio

- Toggle rapido de `self_service_open`.
- Afecta si los markers `self_service` son accesibles a clientes.

### Lavado *(si `allow_laundering` esta activo)*

- Campo importe + preview del impuesto y el recibido.
- Historial de las ultimas 20 operaciones.

---

## 4. Staff cocina en la mesa de crafteo

1. Acercarse al marker `crafting_station` y pulsar E.
2. Ver recetas filtradas por:
    - `station_type` del marker = `station_type` de la receta.
    - Grade del jugador ≥ `min_grade`.
3. Click en **Cocinar**:
    - Se consumen ingredientes del `warehouse` y/o personales.
    - Se entrega el output al inventario del staff.
    - Si la receta tiene `effects` y **nb-consumibles** esta presente, el item queda registrado como consumible.

---

## 5. Staff rellena el autoservicio

1. Acercarse al marker `self_service` y pulsar E (*reserved a staff del job*).
2. La UI muestra los slots. CRUD inline:
    - Item, label, precio, payment (`cash`/`bank`/`both`).
    - Transfer: cuantos items trasladar de tu inventario al slot (suma al stock).
    - Stock `-1` = ilimitado.

---

## 6. Cliente compra en el autoservicio

1. Cualquier cliente se acerca al marker `self_service` y pulsa E.
2. Se abre el UI tipo shop con grid + carrito.
3. Elige pago (`cash` / `bank`).
4. Checkout:
    - Validacion server-side: stock atomico, fondos, can-carry.
    - El importe va a la **sociedad** del restaurante (no al staff).

---

## 7. Camarero factura al cliente

1. Staff se acerca al marker `billing_desk` y pulsa E.
2. UI muestra jugadores cercanos (radio configurable).
3. Selecciona cliente, introduce importe + concepto.
4. **Emitir** — el routing depende del `billing_mode` del restaurante:

    | Modo | Flujo |
    |------|-------|
    | `native` | El cliente recibe un modal NUI para aceptar/rechazar con metodo de pago. |
    | `nb-billings` | Factura formal via `exports['nb-billings']:createInvoice` — el cliente la ve en su panel `/myinvoices`. |
    | `external` | esx_billing / qb-billing / okokBilling (el primero detectado). |

---

## 8. Lavado de dinero

1. Admin habilita `allow_laundering` en el restaurante + ajusta el tax.
2. Boss entra al boss menu → pestana **Lavado**.
3. Introduce importe, ve preview (tax + clean).
4. **Blanquear**:
    - Se consume dinero sucio segun `Config.Laundering.DirtyMoneySource` (item / cash / account).
    - `tax = floor(dirty * tax_rate / 100)`.
    - `clean = dirty - tax` se deposita en la **sociedad** del restaurante.
    - Auditoria en `nb_restaurants_laundering` + webhook Discord opcional.

Salvaguardas:

- Cooldown (`MinIntervalSeconds`, default 60s).
- Cap por operacion (`MaxPerOperation`, default 100000).
- Solo boss o admin pueden accionar.
