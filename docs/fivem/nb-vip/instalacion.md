# Instalacion

Pasos para instalar nb-vip en tu servidor FiveM.

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Servidor con artifacts recientes (5181+) |
| **oxmysql** | Recurso para MySQL/MariaDB |
| **nb-bridge** | Bridge centralizado de Neenbyss |
| **Framework** | ESX Legacy (1.9+) o QBCore |
| **Inventario** | ox_inventory, qb-inventory, qs-inventory o default del framework |

---

## 1. Instalar el recurso

1. Coloca la carpeta **nb-vip** dentro de `resources` (o dentro de una carpeta tipo `[neenbyss]/nb-vip`).
2. Asegurate de que **nb-bridge** este instalado en la misma carpeta o accesible como recurso.
3. Asegurate de que **oxmysql** este instalado y configurado.

---

## 2. Base de datos

Importa el esquema ejecutando `install.sql` en tu base de datos:

```bash
mysql -u usuario -p nombre_base_datos < nb-vip/install.sql
```

Esto crea las siguientes tablas:

| Tabla | Descripcion |
|-------|-------------|
| `nb_vip_coins` | Balance de coins por jugador |
| `nb_vip_transactions` | Historial de transacciones |
| `nb_vip_codes` | Codigos canjeables generados |
| `nb_vip_redemptions` | Registro de canjes realizados |
| `nb_vip_purchases` | Registro de compras |

---

## 3. Configuracion minima

Edita **shared/config.lua**:

```lua
-- Garage: elige tu sistema
Config.Garage = {
    garage = "esx_garage",          -- 'esx_garage', 'qb-garages', 'lunar_garage' o 'custom'
    DefaultGarage = "pillboxgarage"
}

-- Admin: grupos con acceso al panel
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
```

Edita **shared/config_coins.lua** para ajustar la moneda:

```lua
Config.Coins = {
    Currency = 'Coins',
    StartingBalance = 0,
    MaxBalance = 999999,
    AllowTransfers = false,
}
```

El resto de opciones tienen valores por defecto. Ver [Configuracion](configuracion.md).

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended   # o qb-core
ensure nb-bridge
ensure nb-vip
```

El orden es importante: **nb-bridge** debe iniciar antes que **nb-vip**.

---

## 5. Comprobar que funciona

1. Entra al servidor.
2. Usa `/vip` para abrir la tienda VIP.
3. Si tienes permisos de admin, usa `/vipadmin` para abrir el panel de administracion.
4. Verifica en la consola del servidor que aparezca el mensaje de framework detectado.
