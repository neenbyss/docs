# Instalacion

Pasos para instalar nb-consumibles en tu servidor FiveM.

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Servidor con artifacts recientes (5181+) |
| **oxmysql** | Recurso para MySQL/MariaDB |
| **nb-bridge** | Bridge centralizado de Neenbyss (incluye `Bridge.RegisterUsableItem`) |
| **Framework** | ESX Legacy (1.9+) o QBCore |
| **Inventario** | ox_inventory, qb-inventory, qs-inventory o default del framework |

---

## 1. Instalar el recurso

1. Coloca la carpeta **nb-consumibles** dentro de `resources` (o dentro de una carpeta tipo `[neenbyss]/nb-consumibles`).
2. Asegurate de que **nb-bridge** este instalado y con la version que expone `Bridge.RegisterUsableItem`.
3. Asegurate de que **oxmysql** este instalado y configurado.

---

## 2. Base de datos

Importa el esquema ejecutando `[sql]/nb-consumibles.sql`:

```bash
mysql -u usuario -p nombre_base_datos < nb-consumibles/\[sql\]/nb-consumibles.sql
```

Esto crea las siguientes tablas:

| Tabla | Descripcion |
|-------|-------------|
| `nb_consumibles_items` | Items consumibles (nombre, categoria, flags, cooldown, presets) |
| `nb_consumibles_item_effects` | Cadena de efectos por item (tipo + params JSON + orden) |
| `nb_consumibles_anim_presets` | Presets reutilizables de `dict + anim + flags` |
| `nb_consumibles_prop_presets` | Presets reutilizables de prop atado al ped |
| `nb_consumibles_locales` | Traducciones dinamicas editables desde el panel |
| `nb_consumibles_settings` | Ajustes globales clave/valor |

El script incluye seeds iniciales para que el panel no abra vacio: `bread`, `water`, `beer`, `oxy` y `armor`.

> El fichero `.sql` es idempotente (usa `IF NOT EXISTS` y `INSERT IGNORE`). Puedes re-ejecutarlo sin perder datos.

---

## 3. Configuracion minima

Edita `shared/config.lua`:

```lua
-- Grupos que pueden abrir el panel.
Config.AdminGroups = { 'admin', 'superadmin', 'god' }

-- Idioma por defecto del servidor (se puede sobrescribir desde el panel).
Config.Locale = 'es'

-- Comando para abrir el panel.
Config.PanelCommand = 'consumibles'
Config.PanelKeybind = nil           -- opcional, ej. 'F7'
```

Todo lo demas (efectos, items, animaciones, props, traducciones) se gestiona desde el panel.

---

## 4. Arrancar el recurso

En `server.cfg`, **respeta el orden**:

```cfg
ensure oxmysql
ensure es_extended   # o qb-core
ensure nb-bridge
ensure nb-consumibles
```

nb-bridge debe iniciar antes que nb-consumibles.

---

## 5. Comprobar que funciona

1. Entra al servidor.
2. Ejecuta `/consumibles` (requiere ser admin). Se abre el panel.
3. Consume alguno de los items seed (por ejemplo `bread`) — deberias ver la progressbar, la animacion, y el efecto aplicado.
4. En la consola del servidor deberia aparecer:

    ```
    [nb-consumibles] Loaded (ESX / ox_inventory)
    ```

Si ves `Bridge.Framework = nil`, revisa que **nb-bridge** arranca antes.

---

## 6. Items en el inventario

nb-consumibles registra cada item como **usable** automaticamente, pero el item **tiene que existir** previamente en tu inventario:

- **ox_inventory** — anade el item en `ox_inventory/data/items.lua`.
- **qb-inventory** — anade el item en `qb-core/shared/items.lua`.
- **qs-inventory** — anade el item en `qs-inventory/shared/items.lua`.
- **ESX default** — inserta el item en la tabla `items`.

Despues registra el mismo nombre en el panel de nb-consumibles y define sus efectos.
