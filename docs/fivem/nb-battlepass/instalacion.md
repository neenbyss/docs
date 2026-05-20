# Instalacion

Pasos para instalar nb-battlepass en tu servidor FiveM.

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Servidor con artifacts recientes (5181+) |
| **oxmysql** | Recurso para MySQL/MariaDB |
| **nb-bridge** | Bridge centralizado de Neenbyss |
| **Framework** | ESX Legacy (1.9+) o QBCore |

---

## 1. Instalar el recurso

1. Coloca la carpeta **nb-battlepass** dentro de `resources` (o `[neenbyss]/nb-battlepass`).
2. Asegurate de que **nb-bridge** este instalado.
3. Asegurate de que **oxmysql** este configurado.

---

## 2. Base de datos

Importa el esquema ejecutando el archivo SQL:

```bash
mysql -u usuario -p nombre_base_datos < nb-battlepass/[sql]/nb-battlepass.sql
```

Tablas creadas:

| Tabla | Descripcion |
|-------|-------------|
| `nb_battlepass_seasons` | Seasons (configuracion + estado activo) |
| `nb_battlepass_tiers` | Niveles de cada season + recompensas JSON |
| `nb_battlepass_players` | Progreso de cada jugador en una season |
| `nb_battlepass_claims` | Registro de recompensas ya reclamadas |

---

## 3. Configuracion minima

Edita **shared/config.lua**:

```lua
-- Grupos admin con acceso al panel /battlepassadmin
Config.AdminGroups = { 'admin', 'superadmin', 'god' }

-- Idioma ('en' o 'es')
Config.Locale = 'es'

-- Activar/desactivar el boton "Comprar Premium" globalmente
Config.Premium.PurchaseButtonEnabled = true
```

El resto tiene valores por defecto. Ver [Configuracion](configuracion.md).

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended      # o qb-core
ensure nb-bridge
ensure nb-battlepass
```

El orden importa: **nb-bridge** debe arrancar antes que **nb-battlepass**.

---

## 5. Crear tu primera season

1. Conectate al server como admin.
2. Ejecuta `/battlepassadmin` → se abre el panel.
3. Click en **New Season**:
    - **Name**: ej. `Season 1 - Lanzamiento`
    - **Max Level**: 50
    - **XP per Level**: 1000
    - **Premium Enabled**: Yes
    - **Purchasable In-Game**: Yes (si quieres permitir compra desde la UI)
    - **Premium Price**: ej. 50000
4. Guarda y luego **Activate** la season (boton ⚡).
5. Ahora abre la pestaña **Tiers** y crea recompensas para cada nivel.

!!! tip "Solo una season activa a la vez"
    Al activar una season, el sistema desactiva automaticamente cualquier otra. El progreso de los jugadores se guarda por season, asi que activar/desactivar no destruye datos.

---

## 6. Otorgar XP desde otros recursos

nb-battlepass **no tiene tareas/misiones propias**: la XP se otorga via export desde donde tu quieras (jobs, misiones, kills, tiempo jugado, eventos, etc.).

```lua
-- En cualquier recurso server-side
exports['nb-battlepass']:AddXP(source, 100, 'completaste_mision')
```

Ver [Integraciones](integraciones.md) para ejemplos completos.

---

## 7. Comprobar que funciona

1. Entra al server.
2. Usa `/battlepass` → se abre el track con tus niveles.
3. Otorga XP de prueba desde consola (o un script de test) y verifica que sube de nivel.
4. Reclama una recompensa.
