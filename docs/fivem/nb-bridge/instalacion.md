# Instalacion

nb-bridge es la dependencia de todos los recursos `nb-*`. Instalalo **una sola vez** y todos los recursos lo reutilizaran.

---

## 1. Descargar desde GitHub

Repositorio oficial: **[github.com/neenbyss/nb-bridge](https://github.com/neenbyss/nb-bridge)**.

Opcion A — clonar:

```bash
cd resources
git clone https://github.com/neenbyss/nb-bridge.git
```

Opcion B — release estable (recomendado):

1. Abre la pagina de [releases](https://github.com/neenbyss/nb-bridge/releases).
2. Descarga el `.zip` de la **ultima release publicada** — siempre la mas reciente, nunca fijes una version antigua.
3. Descomprime la carpeta en `resources/` de tu servidor.
4. Renombra a `nb-bridge` si la carpeta incluye el hash en el nombre.

> Usa siempre la ultima release estable — las mejoras benefician a TODOS los recursos `nb-*` de tu servidor a la vez.

---

## 2. Dependencias

nb-bridge no trae base de datos propia, pero requiere:

| Dependencia | Obligatoria | Proposito |
|-------------|-------------|-----------|
| **oxmysql** | si | Queries de vehiculo y licencias. |
| **es_extended**, **qb-core** o **qbx_core** | si | Framework a detectar. Uno de los tres. |
| **ox_lib** | no | Activa progress bars y notificaciones mejoradas. Obligatorio de facto en servidores QBX. |

---

## 3. Orden en `server.cfg`

El orden es **critico**. nb-bridge tiene que iniciar antes que cualquier recurso `nb-*`:

```cfg
ensure oxmysql
ensure es_extended          # o qb-core / qbx_core
ensure ox_lib               # opcional pero recomendado
ensure nb-bridge            # OBLIGATORIO antes de cualquier nb-*

# Tus recursos consumidores (orden entre ellos libre)
ensure nb-crafting
ensure nb-cars
ensure nb-jobmanagers
ensure nb-vip
ensure nb-consumibles
```

---

## 4. Comprobar que funciona

Al arrancar el servidor, busca en consola un log parecido a:

```
[nb-bridge] Framework detected: QBX (qbx_core)
[nb-bridge] Inventory detected: ox_inventory
```

(o `ESX` / `QBCore` segun tu framework).

Desde cualquier recurso consumidor puedes comprobarlo con el export `get()` — el `Bridge` global de nb-bridge **no** es accesible directamente desde otro recurso, cada resource tiene su propio estado de Lua:

```lua
local bridge = exports['nb-bridge']:get()

CreateThread(function()
    Wait(500)
    print('Framework:', bridge.Framework)
    print('Inventory:', bridge.InventorySystem)
end)
```

Si `bridge.Framework` es `nil`, significa que ni ESX, ni QBCore ni QBX arrancaron antes — revisa el orden en `server.cfg`.

Tambien puedes usar el comando `/nbdiag` (admin o consola) para un snapshot completo, o el export directo `exports['nb-bridge']:diagnostics()` — ver [Modulos](modulos.md#diagnostics).

---

## 5. Agregar la dependencia en tu script

En el `fxmanifest.lua` de cualquier recurso consumidor:

```lua
dependencies {
    'oxmysql',
    'nb-bridge',
}
```

No hace falta listar modulos en `shared_scripts`. Pero a diferencia de v1.x, **no hay un `Bridge` global disponible automaticamente** en tu recurso — tenes que pedirlo explicitamente via el export `get()` al inicio de cada script:

```lua
-- server.lua / client.lua
local bridge = exports['nb-bridge']:get()

-- Ahora usas los metodos namespaced:
bridge.player.addMoney(source, 'bank', 500, 'salary')
bridge.notify.send(source, 'Pago recibido', 'success')
```

Llama a `get()` una sola vez por archivo y reutiliza esa variable local. El viejo patron `shared_scripts { '@nb-bridge/loader.lua' }` que inyectaba un `Bridge` global en tu recurso esta **deprecado** desde v2.0.0 y se eliminara en una futura major — ver [Modulos](modulos.md) para la API completa.

---

## 6. Actualizar

1. Detener el servidor (o los recursos que usan nb-bridge).
2. Reemplazar la carpeta `nb-bridge` por la nueva version.
3. Revisar el [changelog](changelog.md) por cambios que afecten tus overrides.
4. Arrancar de nuevo.

Las versiones siguen [semver](https://semver.org/): las **minor** y **patch** son compatibles hacia atras; una **major** (ej. `2.0.0`, que reemplazo toda la API plana por una namespaced) puede requerir ajustes en tu codigo.
