# Instalacion

---

## Requisitos

| Requisito | Descripcion |
|-----------|-------------|
| **FiveM** | Servidor con artifacts recientes (5181+) |
| **oxmysql** | Recurso para MySQL/MariaDB |
| **nb-bridge** | Bridge centralizado de Neenbyss |
| **Framework** | ESX Legacy o QBCore |

---

## 1. Instalar el recurso

1. Coloca la carpeta **nb-gangwars** dentro de `resources` (o de una carpeta de categoria).
2. Asegurate de que **nb-bridge** este instalado.
3. Comprueba que **oxmysql** arranca antes.

---

## 2. Base de datos

La tabla se crea automaticamente al arrancar por primera vez (`CREATE TABLE IF NOT EXISTS`). No tienes que importar nada manualmente.

```sql
CREATE TABLE IF NOT EXISTS `nb_gangwars_zones` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `name` VARCHAR(50) NOT NULL,
    `type` ENUM('square', 'circle') DEFAULT 'square',
    `x` FLOAT NOT NULL,
    `y` FLOAT NOT NULL,
    `z` FLOAT NOT NULL,
    `width` FLOAT DEFAULT 100.0,
    `height` FLOAT DEFAULT 100.0,
    `radius` FLOAT DEFAULT 50.0,
    `rotation` FLOAT DEFAULT 0.0,
    `capture_price` INT DEFAULT 10000,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 3. Configuracion minima

Edita `config.lua`:

```lua
Config.Language = 'es'           -- 'en' | 'es' | 'fr'
Config.ActiveZones = 4           -- zonas simultaneas en rotacion
Config.PhaseDuration = 40        -- segundos por fase
Config.PhaseCooldown = 60        -- segundos de cooldown entre fases
Config.PhasesToCapture = 3       -- fases necesarias para capturar
Config.NextZoneDelay = 10800     -- 3 horas antes de respawnear la zona
```

Edita tambien las **tres funciones custom** al final del config segun tu sistema de gangs. Ver [Integracion](integracion.md).

---

## 4. Arrancar el recurso

En `server.cfg`:

```cfg
ensure oxmysql
ensure es_extended       # o qb-core
ensure nb-bridge
ensure nb-gangwars
```

---

## 5. Comprobar que funciona

1. Entra al servidor como admin.
2. Ejecuta `/gangzones` — se abre el panel.
3. Crea una zona de prueba con el boton "Usar mi posicion".
4. Activa la zona. Verifica el blip en el mapa.
5. Entra a la zona con un personaje de una gang y confirma que empieza la fase de captura.
