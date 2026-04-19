# nb-actions

Panel contextual de **acciones rapidas** para policia, EMS y cualquier rol que lo necesite. Pulsas F6 y aparecen las acciones disponibles sobre el jugador/vehiculo mas cercano: esposar, arrastrar, meter en vehiculo, comprobar identidad y licencias, consultar duenos de vehiculos, registrar inventario.

---

## Caracteristicas

- 🎯 **Deteccion automatica** del jugador o vehiculo mas cercano.
- 👥 **Selector multiple** — si hay varios objetivos cerca, el panel muestra una lista para elegir.
- 💀 **Detecta muertos** — las acciones relevantes cambian (registrar cadaver vs jugador vivo, etc.).
- 🚔 **Workflow de arresto** — esposar → arrastrar → meter en vehiculo con animaciones encadenadas.
- 🪪 **Checks de licencias** — identidad, carnet de conducir y licencia de armas con auto-deteccion de provider (okokLicenses, esx_license, QBCore metadata).
- 🚗 **Consulta de vehiculos** — lookup del dueno por matricula en la tabla del framework.
- 📦 **Registro de inventario** — fuerza abrir el inventario del target (vivo o muerto).
- 🛠️ **Permisos granulares** — por grupo admin, por trabajo/grado, por identificador especifico.
- 🧩 **Bridges editables** — todas las integraciones viven en archivos abiertos (licencias, vehiculos, inventario).
- 🌐 **Multi-framework** — ESX + QBCore via nb-bridge.
- 🌎 **i18n** — EN / ES, extensible.

---

## Compatibilidad

| Requisito | Versiones |
|-----------|-----------|
| **Framework** | ESX Legacy / QBCore (via nb-bridge) |
| **Base de datos** | oxmysql |
| **Bridge** | nb-bridge |
| **Licencias (opcional)** | okokLicenses, esx_license, QBCore metadata |
| **Inventario** | ox_inventory, qb-inventory, qs-inventory, origen_inventory |

---

## Secciones

- **[Instalacion](instalacion.md)** — Pasos de puesta en marcha.
- **[Configuracion](configuracion.md)** — Todas las opciones.
- **[Acciones disponibles](acciones.md)** — Que hace cada accion y cuando se muestra.
- **[Permisos](permisos.md)** — Grupos admin, jobs/grados, identificadores.
- **[Exports](exports.md)** — API para otros recursos.
