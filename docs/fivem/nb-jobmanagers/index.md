# nb-jobmanagers

Sistema completo de gestión de trabajos para FiveM (ESX/QBCore). Panel de administración, boss menu con dinero de sociedad, facturación, garaje de sociedad, acciones rápidas y sistema de permisos granular.

---

## Lo importante

- **[Instalacion](instalacion.md)** — Requisitos, base de datos, config minima y arranque.
- **[Configuracion](configuracion.md)** — Todas las opciones de config.lua.
- **[Permisos](permisos.md)** — Sistema de permisos bitwise por grado: boss menu, facturas, empleados, acciones.
- **[Boss Menu](bossmenu.md)** — Dinero de sociedad, gestion de empleados, facturas e historial.
- **[Multi-Job](multijob.md)** — Trabajos simultaneos por jugador, `/myjobs` (F7), auto-asignacion y comandos admin.
- **[Comandos](comandos.md)** — Comandos disponibles y teclas.
- **[Exports](exports.md)** — API para integrar nb-jobmanagers desde otros scripts.
- **[Compatibilidad](compatibilidad.md)** — Integracion con nb-billings, nb-garages y ejemplos de uso desde otros scripts.
- **[Adaptar scripts externos](adaptaciones.md)** — Como reemplazar el garaje, facturacion o dinero de sociedad por scripts de terceros.

---

## Caracteristicas

- **Panel Admin** — Crea, edita y elimina trabajos, grados y markers desde una interfaz NUI.
- **Boss Menu** — Depositar/retirar dinero de sociedad, invitar/promover/despedir empleados, ver historial de transacciones.
- **Facturacion** — Sistema de facturas nativo o integrado con nb-billings.
- **Garaje de Sociedad** — Vehiculos compartidos por trabajo, con integracion opcional con nb-garages.
- **Acciones Rapidas (F6)** — Esposar, registrar, verificar vehiculos y mas, segun permisos del grado.
- **Multi-Job (F7)** — Un jugador puede tener varios trabajos a la vez (policia + mecanico, p.ej.) y alternar entre ellos.
- **Permisos Bitwise** — Control granular por grado: cada permiso es un flag independiente.
- **Multi-framework** — Compatible con ESX y QBCore via nb-bridge.
- **Vestuario** — Outfits por trabajo, guardados en base de datos.

## Compatibilidad

- ESX Legacy (1.9.0+)
- QBCore
- oxmysql
- nb-bridge
