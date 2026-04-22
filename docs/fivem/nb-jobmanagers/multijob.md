# Multi-Job

Un jugador puede tener hasta N trabajos a la vez (policia + mecanico, por ejemplo) y cambiar entre ellos con `/myjobs` (F7). El trabajo **activo** es el que el framework reporta via `Bridge.GetJob` / `PlayerData.job` — los demas siguen asignados pero en reposo.

---

## Como funciona

- Los trabajos de cada jugador se guardan en la tabla `nb_player_jobs` (una fila por pareja jugador+trabajo).
- Solo una fila por jugador tiene `active = 1` y espeja el trabajo del framework.
- Al conectar, `nb-jobmanagers` reconcilia el framework con la fila activa. Si el jugador no tiene ninguna fila, siembra su trabajo actual del framework.
- Cuando despiden a un jugador desde el boss menu, promueve al trabajo de mayor grado restante en lugar de tirarlo a `unemployed`.

---

## Comandos de jugador

| Comando | Tecla | Descripcion |
|---------|-------|-------------|
| `/myjobs` | **F7** | Abre el menu de Multi-Job: lista tus trabajos y (si esta habilitado) los trabajos publicos a los que puedes unirte. |

El comando y la tecla se pueden cambiar en `Config.MultiJob.Command` y `Config.MultiJob.Keybind`.

---

## Comandos de admin

| Comando | Uso | Descripcion |
|---------|-----|-------------|
| `/assignjob` | `/assignjob [id\|citizenid] [job] [grade]` | Asigna un trabajo y lo activa en el framework (lo que `/job` reporta). Grado opcional (default 0). |
| `/removejob` | `/removejob [id\|citizenid] [job]` | Quita un trabajo. Si era el activo, promueve el siguiente de mayor grado o pone `unemployed`. |
| `/publicjob` | `/publicjob [job] [on\|off]` | Expone o oculta un trabajo en la pestaña "Trabajos Disponibles" de F7. |
| `/whichjobs` | `/whichjobs [id\|citizenid]` | Imprime en la consola del servidor cada fila de `nb_player_jobs` para el objetivo. Herramienta de diagnostico. |

Todos los comandos requieren que el admin este en `Config.AdminGroups` (o se ejecuten desde la consola del servidor).

### Ejemplos

```
/assignjob 5 police 0            # asigna police grado 0 al server id 5
/assignjob ABC12345 ems          # asigna ems grado 0 al jugador offline por citizenid (QBCore)
/removejob 5 mechanic            # quita mechanic al server id 5
/publicjob taxi on               # expone taxi en "Trabajos Disponibles"
/whichjobs 5                     # lista las filas de nb_player_jobs del server id 5
```

---

## QBCore: gotcha del identifier

En QBCore cada personaje tiene un `citizenid` distinto (multichar), y es ese el que la sesion usa para leer `nb_player_jobs`. Pasar `license:...` en `/assignjob` escribe la fila bajo una clave que F7 nunca consulta — el jugador queda "asignado" en la DB pero no ve nada.

**Regla:** usa **server ID** si el jugador esta online (se traduce al citizenid correcto del personaje activo) o el **citizenid** del personaje objetivo si esta offline. `license:...` se rechaza desde 1.2.1.

Si una asignacion vieja quedo con el identifier equivocado:

```
/whichjobs <serverID>        # muestra las filas que ve la sesion del jugador
/whichjobs <citizenidViejo>  # compara con lo que quedo escrito
```

```sql
UPDATE nb_player_jobs
   SET player_identifier = '<CITIZENID_CORRECTO>'
 WHERE player_identifier = '<IDENTIFIER_EQUIVOCADO>';
```

---

## Auto-asignacion (Trabajos Disponibles)

El sistema combina dos campos:

1. **`jobs.whitelisted`** — si es `1`, el trabajo nunca aparece en "Trabajos Disponibles".
2. **`nb_job_autoselect.enabled`** — override opcional por trabajo.

### Modo opt-in (default desde 1.2.0)

```lua
Config.MultiJob.AllowSelfAssign = true
Config.MultiJob.SelfAssignDefault = false
```

Ningun trabajo aparece hasta que el admin lo exponga con `/publicjob <job> on`. Es el comportamiento recomendado: evita confusion de "por que puedo ver este trabajo si no me lo dejan usar".

### Modo legacy

```lua
Config.MultiJob.SelfAssignDefault = true
```

Cualquier trabajo con `whitelisted = 0` aparece automaticamente; `/publicjob <job> off` para ocultar uno concreto.

### Desactivar toda la pestaña

```lua
Config.MultiJob.AllowSelfAssign = false
```

El payload del NUI incluye `available = {}` y la seccion desaparece. Los admins siguen teniendo `/assignjob` y `/publicjob`.

---

## Exports

Todos reciben `identifier` que en QBCore es **citizenid** y en ESX es **license**.

| Export | Descripcion |
|--------|-------------|
| `getPlayerJobs(identifier)` | Lista todas las filas de `nb_player_jobs` del jugador. |
| `getActiveJob(identifier)` | Devuelve el trabajo activo del jugador. |
| `addPlayerJob(identifier, jobName, grade, assignedBy)` | Agrega un trabajo sin activarlo. |
| `removePlayerJob(identifier, jobName)` | Quita un trabajo (gestiona fallback si era el activo). |
| `setActiveJob(identifier, jobName)` | Cambia el trabajo activo. |
| `setJobAutoSelect(jobName, enabled)` | Expone/oculta un trabajo en "Trabajos Disponibles". |

Detalles y firmas completas en [Exports](exports.md).

---

## FAQ

**Le asigne 2 trabajos pero F7 solo muestra el que tenia antes (QBCore).**
Casi siempre es un mismatch de identifier. Corre `/whichjobs <serverID>` — si las filas aparecen bajo un `license:...` o un citizenid viejo, repara con el `UPDATE nb_player_jobs` de arriba. Desde 1.2.1 `/assignjob` rechaza `license:...` para evitar que se repita.

**Reinicie el recurso y el jugador quedo en unemployed.**
La reconciliacion solo ocurre en `OnPlayerLoaded`. Si reinicias con el jugador online, reconectalo y vuelve a su trabajo activo.

**Quiero que no se puedan combinar ciertos trabajos.**
No hay exclusion nativa. Engancha un script externo a `addPlayerJob` y llama a `removePlayerJob` para el trabajo conflictivo antes.
