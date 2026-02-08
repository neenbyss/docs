# Instalación

Pasos para instalar y poner en marcha nb-crafting en tu servidor FiveM.

---

## Requisitos

| Requisito | Descripción |
|-----------|-------------|
| **FiveM** | Servidor con artifacts recientes (5181+) |
| **oxmysql** | Recurso para MySQL/MariaDB |
| **Framework** | ESX Legacy (1.9.0+) o QBCore |
| **Inventario** | ox_inventory, qb-inventory, ps-inventory, lj-inventory, qs-inventory u origen_inventory |

---

## 1. Instalar el recurso

1. Coloca la carpeta **nb-crafting** dentro de `resources` (o dentro de una carpeta tipo `[neenbyss]/nb-crafting`).
2. Asegúrate de que **oxmysql** esté instalado y configurado con tu base de datos.

---

## 2. Base de datos

1. Crea la base de datos o usa una existente.
2. Importa el esquema inicial:
   ```bash
   mysql -u usuario -p nombre_base_datos < nb-crafting/sql/crafting.sql
   ```
3. Si actualizas desde una versión anterior, ejecuta también las migraciones:
   ```bash
   mysql -u usuario -p nombre_base_datos < nb-crafting/sql/crafting_migrations.sql
   ```

---

## 3. Configuración mínima

Edita **config.lua** en la raíz del recurso:

```lua
Config.Framework = 'ESX'   -- o 'QBCore'
Config.Inventory = 'ox_inventory'   -- según tu inventario
Config.Permissions.adminGroup = 'admin'   -- grupo que puede usar el panel admin
```

El resto de opciones tienen valores por defecto. Ver [Configuración](configuracion.md).

No hace falta tocar **bridge/** si usas ESX o QBCore con un inventario soportado. Si usas **Origen Ilegal** (bandas), sigue [Origen Ilegal](origen-ilegal.md).

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended   # o qb-core
ensure ox_inventory  # o tu inventario
ensure nb-crafting
```

O para reiniciar solo el crafteo:

```cfg
restart nb-crafting
```

---

## 5. Comprobar que funciona

1. Entra al servidor con un personaje que tenga el grupo de admin configurado.
2. Acércate a una estación de crafteo (si ya hay alguna en la base de datos) o crea una con el panel admin.
3. Abre el panel admin con el comando indicado en el recurso (por ejemplo `/craftingadmin`) y crea una estación y una receta de prueba.
4. Con otro personaje (o sin permisos de admin) prueba a abrir la estación, craftear y recoger ítems.

Si algo falla, revisa la consola del servidor y del cliente (F8) y que `config.lua` y `bridge` coincidan con tu framework e inventario.
