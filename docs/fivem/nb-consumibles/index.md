# nb-consumibles

Sistema de consumibles configurable **desde dentro del juego**. Olvidate de editar `Config.lua` para cada item: creas, editas y eliminas items con un panel NUI y los efectos se aplican al instante gracias al hot-reload.

---

## Caracteristicas

- 🎛️ **Panel NUI in-game** — CRUD completo de items, efectos, presets de animacion/prop y traducciones sin tocar ficheros.
- ⚡ **30+ efectos listos** — hambre, sed, salud, chaleco, stamina, camara lenta, vision nocturna, super salto, buffs de dano, teletransporte aleatorio, recompensas economicas y mas.
- 🔗 **Efectos encadenables** — cada item puede ejecutar N efectos en el orden que elijas, con un toggle por efecto.
- 🧪 **Catalogo auto-documentado** — la UI genera los inputs correctos (numero, dropdown, bool, lista JSON) a partir del schema del catalogo.
- 🎨 **Presets reutilizables** — dicts de animacion y props se definen una vez y se reutilizan entre items.
- 🌎 **Traducciones dinamicas** — strings editables desde el panel, con soporte para `db::mykey` dentro del propio item.
- 🌐 **Multi-framework** — ESX y QBCore via nb-bridge.
- 📦 **Multi-inventario** — ox_inventory, qb-inventory, qs-inventory o default del framework.
- 🔥 **Hot-reload** — cambios del panel se propagan a todos los clientes sin reinicio.
- 🧩 **Extensible** — exports para anadir items y efectos custom desde otros scripts.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy (1.9+) / QBCore |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge con `Bridge.RegisterUsableItem` |
| **Inventario** | ox_inventory, qb-inventory, qs-inventory o default |
| **HUD (opcional)** | Compatible con `hud:client:UpdateNeeds` y `hud:server:RelieveStress` |
| **Policia (opcional)** | Compatible con `evidence:client:SetStatus` |

---

## Secciones

- **[Instalacion](instalacion.md)** — Requisitos, base de datos y puesta en marcha.
- **[Configuracion](configuracion.md)** — Opciones en `shared/config.lua`.
- **[Panel in-game](panel.md)** — Como usar el panel: items, efectos, presets, idiomas, ajustes.
- **[Efectos disponibles](efectos.md)** — Catalogo completo con parametros y ejemplos.
- **[Comandos](comandos.md)** — Comandos y keybinds.
- **[Exports](exports.md)** — API para usar nb-consumibles desde otros scripts.
