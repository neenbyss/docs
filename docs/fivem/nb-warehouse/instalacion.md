# Instalación

---

## Requisitos

| Requisito | Descripción |
|-----------|-------------|
| **oxmysql** | Persistencia de almacenes |
| **nb-bridge** | v2.3.0+ — requiere `Bridge.log.*` / `Bridge.ui.*` |
| **ox_lib** | Menús contextuales, diálogos de input/confirmación y el skillcheck del robo |
| **Framework** | ESX Legacy, QBCore o QBX (vía `nb-bridge`, nunca acceso directo) |
| **OneSync** | Infinity recomendado, no estrictamente requerido |

---

## 1. Instalar el recurso

Coloca **nb-warehouse** en `resources/[neenbyss]/nb-warehouse/`.

---

## 2. Base de datos — se instala sola

`server/migrations.lua` corre en cada arranque del recurso, **antes de cargar
cualquier almacén**, y:

- Crea las tablas `nb_warehouses` / `nb_warehouse_access` con
  `CREATE TABLE IF NOT EXISTS` si no existen.
- Aplica `ALTER TABLE` idempotentes (columna por columna, verificando
  `information_schema` antes de cada uno) si la instalación ya existe pero
  le faltan columnas de una versión anterior — incluye agregar el valor
  `'player'` al ENUM `owner_type` si venías de una instalación previa a la
  1.1.0.

**No hace falta ejecutar nada a mano.** `[sql]/nb-warehouse.sql` se conserva
solo como referencia/documentación del esquema final (útil para inspección
manual o importación en un entorno sin arrancar el recurso primero) — ya no
es un paso obligatorio de instalación.

---

## 3. Orden en `server.cfg`

```cfg
ensure ox_lib
ensure nb-bridge
ensure nb-warehouse
```

Debe ir **después** del framework y de `nb-bridge`/`ox_lib`.

---

## 4. Configuración mínima

Edita `shared/config.lua` según necesites — ver **[Configuración](configuracion.md)**
para la referencia completa. Los valores por defecto funcionan out-of-the-box.

---

## 5. Almacenes

nb-warehouse arranca **sin almacenes** registrados. Como admin, ejecuta:

```
/warehouseadmin
```

Crea un almacén → captura coordenadas in-game (`[E]` captura, `[BACKSPACE]`
cancela) → configura tipo, owner, precio/renta, riesgo de robo, whitelist →
**Save**. El stash se registra y el sync se propaga a todos los clientes al
instante, sin reiniciar el recurso.

---

## 6. Verificación

1. Reinicia el recurso. La consola debe mostrar la creación/verificación de
   tablas por `migrations.lua` sin errores.
2. Como admin: `/warehouseadmin` abre el creador.
3. Camina hasta un almacén habilitado: debe aparecer el marker + prompt `[E]`.
