# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Servidor con artifacts recientes (5181+) |
| **oxmysql** | Recurso para MySQL/MariaDB |
| **nb-bridge** | Bridge centralizado de Neenbyss |
| **Framework** | ESX Legacy (1.9+) o QBCore |
| **Inventario** | ox_inventory, qb-inventory, qs-inventory o origen_inventory |

---

## 1. Instalar el recurso

1. Coloca la carpeta **nb-shops** dentro de `resources`.
2. Asegurate de tener **nb-bridge** instalado.

---

## 2. Base de datos

Importa `[sql]/install.sql`:

```bash
mysql -u usuario -p nombre_db < nb-shops/\[sql\]/install.sql
```

Tablas creadas:

| Tabla | Descripcion |
|-------|-------------|
| `nb_shops` | Cabecera de la tienda (tipo, ubicacion, marker/blip/ped, metodos de pago). |
| `nb_shop_items` | Items de la tienda (`FK → nb_shops` con `CASCADE DELETE`). |

Schema resumido:

```sql
CREATE TABLE `nb_shops` (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50) NOT NULL,
  type ENUM('civil', 'business', 'illegal'),
  job JSON, gang JSON, coords JSON,
  use_marker TINYINT(1), use_ped TINYINT(1), use_blip TINYINT(1),
  ped_model VARCHAR(50), ped_scenario VARCHAR(100),
  blip_sprite INT, blip_color INT, blip_scale FLOAT,
  categories JSON, payment_methods JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE `nb_shop_items` (
  id INT AUTO_INCREMENT PRIMARY KEY,
  shop_id INT NOT NULL,
  item_name VARCHAR(50), label VARCHAR(100),
  category VARCHAR(50) DEFAULT 'general',
  price INT DEFAULT 0,
  price_type VARCHAR(50) DEFAULT 'cash',
  item_price VARCHAR(50), item_price_amount INT DEFAULT 0,
  stock INT DEFAULT -1, current_stock INT DEFAULT -1,
  sort_order INT DEFAULT 0
);
```

---

## 3. Configuracion minima

Edita `shared/config.lua`:

```lua
Config.AdminGroups = { 'admin', 'superadmin', 'god' }
Config.Locale      = 'es'
Config.Command     = 'shops'        -- comando para abrir el panel
```

El resto (markers, blips, peds) tiene defaults razonables. Ver [Configuracion](configuracion.md).

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended       # o qb-core
ensure nb-bridge
ensure nb-shops
```

---

## 5. Comprobar que funciona

1. Entra al servidor como admin.
2. Ejecuta `/shops` — se abre el panel con el listado vacio.
3. Pulsa **"Nueva tienda"** y sigue el wizard.
4. Al terminar deberias ver el marker en el sitio configurado. Pulsa **E** para abrir la tienda.
