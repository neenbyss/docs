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
2. Descarga el `.zip` de la ultima version (`v1.2.0` o superior).
3. Descomprime la carpeta en `resources/` de tu servidor.
4. Renombra a `nb-bridge` si la carpeta incluye el hash en el nombre.

> Usa siempre la ultima release estable — las mejoras benefician a TODOS los recursos `nb-*` de tu servidor a la vez.

---

## 2. Dependencias

nb-bridge no trae base de datos propia, pero requiere:

| Dependencia | Obligatoria | Proposito |
|-------------|-------------|-----------|
| **oxmysql** | si | Queries de vehiculo y licencias. |
| **es_extended** o **qb-core** | si | Framework a detectar. Uno de los dos. |
| **ox_lib** | no | Activa progress bars y notificaciones mejoradas. |

---

## 3. Orden en `server.cfg`

El orden es **critico**. nb-bridge tiene que iniciar antes que cualquier recurso `nb-*`:

```cfg
ensure oxmysql
ensure es_extended          # o qb-core
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
[nb-bridge] Framework detected: ESX
[nb-bridge] Inventory detected: ox_inventory
```

Ademas, desde cualquier recurso puedes comprobarlo:

```lua
CreateThread(function()
    Wait(500)
    print('Framework:', Bridge.Framework)
    print('Inventory:', Bridge.InventorySystem)
end)
```

Si `Bridge.Framework` es `nil`, significa que ni ESX ni QBCore arrancaron antes — revisa el orden en `server.cfg`.

---

## 5. Agregar la dependencia en tu script

En el `fxmanifest.lua` de cualquier recurso consumidor:

```lua
dependencies {
    'oxmysql',
    'nb-bridge',
}
```

No hace falta listar modulos en `shared_scripts`. El `Bridge` global ya esta disponible cuando tu script arranca.

---

## 6. Actualizar

1. Detener el servidor (o los recursos que usan nb-bridge).
2. Reemplazar la carpeta `nb-bridge` por la nueva version.
3. Revisar el [changelog](changelog.md) por cambios que afecten tus overrides.
4. Arrancar de nuevo.

Las versiones siguen [semver](https://semver.org/): las **minor** y **patch** son compatibles hacia atras; una **major** (ej. `2.0.0`) puede requerir ajustes.
