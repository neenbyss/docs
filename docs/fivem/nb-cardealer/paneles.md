# Paneles

Dos paneles principales, separados por permisos.

---

## Panel admin — `/nb_admin`

**Permiso:** `Config.AdminGroups`.

### Funciones

- **Lista de concesionarios** — nombre, job, cuenta de sociedad.
- **Crear** (o `/nb_createdealer`) — modal con nombre + job.
- **Eliminar** — con confirmacion; borra en cascada requests, stock y showroom (los vehiculos del showroom se eliminan del mundo).

Advertencia: al eliminar un concesionario el dinero `society_<job>` queda fuera del alcance del panel. Si necesitas recuperarlo, hazlo via `esx_addonaccount` directamente.

---

## Panel boss — `/nb_bossmenu`

**Permiso:** ser boss del job del concesionario, o admin.

Cuatro pestanas:

### 1. Peticiones

Lista de `nb_dealer_sell_requests` con estado `PENDING` y `COUNTERED`.

- **Aceptar** — compra al vendedor por `asking_price`.
- **Rechazar** — descartar.
- **Contra-oferta** — introducir `counter_price`; el vendedor vera la contra y decide.

Cada fila muestra:

- Nombre + identifier del vendedor.
- Modelo del vehiculo.
- Plate.
- `asking_price`.
- Timestamp.

### 2. Stock

Lista de `nb_dealer_stock` — los coches comprados pendientes de mover al showroom.

- **Poner en showroom** — abre modal con:
    - Selector de slot libre (auto-sugiere el siguiente disponible).
    - Precio de venta.
    - Coordenadas (las coge del jugador que abre el menu; el coche se spawnea ahi).
- **Eliminar** — descartar del stock sin reembolso.

### 3. Dinero (society)

Balance de `society_<job>`. **Solo boss o admin pueden depositar/retirar (v1.6)**.

- **Depositar** — transfiere cash/bank del jugador a la sociedad.
- **Retirar** — transfiere de la sociedad al bank del jugador.

### 4. Ajustes

- **Mover sell point** — el menu coge tu posicion actual y la guarda como punto de venta.
- **Mover manage point** — el menu coge tu posicion actual y la guarda como punto de gestion (donde se abre `/nb_bossmenu`).

---

## Modales auxiliares

### Counter-offer

- Campo numerico con el `counter_price`.
- Validacion `1 <= counter_price <= Config.MaxPrice`.
- Submit envia el evento al servidor y actualiza el estado del request a `COUNTERED`.

### Showroom placement

- Preview del modelo con su spawn name.
- Slider de precio con cap `Config.MaxPrice`.
- Boton **Colocar aqui** para usar las coords del jugador, o **Cancelar**.

### Admin details

- Detalles del concesionario seleccionado en `/nb_admin`:
    - Job + sociedad.
    - Contadores de requests, stock, showroom.
    - Balance de la sociedad.

---

## Seguridad del panel

- Todos los eventos del boss menu re-validan que el invocador sea boss del job, no solo que haya abierto la UI. Si un cliente modificado dispara un evento sin ser boss, el servidor lo descarta.
- Rate limiting (2s) en dealer actions y deposito/retiro.
- Locks anti-doble click — si dos bosses aceptan la misma peticion simultaneamente, solo uno gana.
