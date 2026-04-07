# Comandos

Comandos y atajos de teclado disponibles en nb-vip.

---

## Comandos de jugador

| Comando | Descripcion |
|---------|-------------|
| `/vip` | Abre la tienda VIP (configurable en `Config.Commands.Open`) |
| **F5** | Abre el menu VIP (configurable en `Config.OpenKey`, `false` para desactivar) |

---

## Comandos de admin

| Comando | Descripcion |
|---------|-------------|
| `/vipadmin` | Abre el panel de administracion (configurable en `Config.Commands.Admin`) |

Requiere que el jugador pertenezca a uno de los grupos definidos en `Config.AdminGroups`.

### Funciones del panel admin

- **Dar Coins** — Agrega coins al balance de un jugador por su ID.
- **Quitar Coins** — Remueve coins del balance de un jugador.
- **Generar Codigo** — Crea un codigo canjeable de tipo coins o paquete, con usos maximos y expiracion.
- **Codigos Activos** — Lista de codigos activos con opcion de desactivar.
- **Estadisticas** — Total de compras, codigos canjeados y coins en circulacion.
