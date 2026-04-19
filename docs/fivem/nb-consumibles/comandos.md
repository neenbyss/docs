# Comandos

nb-consumibles expone un unico comando principal. Todo lo demas ocurre por uso normal de items.

---

## Comandos

| Comando | Descripcion |
|---------|-------------|
| `/consumibles` | Abre el panel de administracion (configurable en `Config.PanelCommand`). Requiere pertenecer a un grupo de `Config.AdminGroups`. |

Si rellenas `Config.PanelKeybind = 'F7'` (o la tecla que prefieras), se registra un `RegisterKeyMapping` para abrir el panel sin comando.

---

## Uso de items

Los items consumibles no usan comandos: el jugador los **usa desde el inventario** como cualquier otro item. nb-consumibles se encarga de:

1. Detectar el uso via `Bridge.RegisterUsableItem` (abstrae `ESX.RegisterUsableItem` / `QBCore.Functions.CreateUseableItem`).
2. Validar cooldown + permisos del item.
3. Reproducir la progressbar + animacion + prop.
4. Si no se cancela, aplicar la cadena de efectos configurada.

---

## Hot reload remoto

Desde el panel (boton 🔁 en la cabecera) se dispara `nb-consumibles:server:reload`, que:

1. Reinicia la cache de items del servidor.
2. Reinscribe usable items nuevos (los registrados previamente no se duplican gracias al tracker interno).
3. Propaga el catalogo actualizado a todos los clientes conectados.

Tambien se puede llamar programaticamente desde otro recurso:

```lua
TriggerEvent('nb-consumibles:server:reload')
```

---

## Comandos de diagnostico

En modo debug (`Config.Debug = true`) todos los eventos internos se imprimen con formato:

```
[nb-consumibles][SERVER][Server] loaded 12 items / 45 locale rows
[nb-consumibles][CLIENT][Effects] client handler failed sprint_multiplier ...
```

No hay un comando `/consumiblesdump` dedicado — si necesitas uno en tu servidor, registralo llamando a `exports['nb-consumibles']:GetAllItems()`.
