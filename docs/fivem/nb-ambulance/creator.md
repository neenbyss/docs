# Creator in-game

Editor NUI para crear, editar y eliminar hospitales sin tocar Lua. **Persiste en SQL** y sincroniza en vivo a todos los clientes — al guardar un cambio los blips y NPCs se rebuiltean al instante.

---

## Acceso

```
/creator              # alias directo
/nb-ambulance creator # equivalente umbrella
```

Solo admins (ver `Bridge.IsAdmin`). Los no-admins reciben toast `no_permission` y la NUI no abre.

---

## Flujo

### Crear hospital

1. Abre el creator.
2. Click **`+ Nuevo hospital`** (esquina inferior izquierda).
3. Pestaña **Info** → rellena:
   - **Nombre interno**: snake_case unico (ej. `pillbox`, `paleto`, `sandy`). No se cambia despues sin riesgo de desync.
   - **Label visible**: el que vera el jugador en el blip / dispatch.
   - **Coste de respawn**: $ que se cobran al respawnear aqui (0 = gratis). Sobrescribe `Config.Death.RespawnCost`.
   - **Activo** (check): si esta off, el hospital no recibe respawns ni muestra blip.
   - **Centro del hospital**: click `Capturar mis coordenadas` → el NUI se oculta, te mueves al sitio en el juego, presionas `[E]` (o `[BACKSPACE]` para cancelar).
   - **Blip**: activo, sprite (61 = hospital por defecto), color, escala.
4. Click **Guardar**. El hospital aparece en la lista de la izquierda y el blip surge en el mapa al instante.

### Añadir camas (respawn points)

1. Selecciona el hospital → pestaña **Camas**.
2. Click **`Capturar cama`** → captura coords. Cada cama es un punto de respawn.
3. La cama mas cercana al lugar de muerte del jugador es a la que respawnea (server-side).
4. Boton GPS por cama para teleportarte y verificar la posicion.

> **Nota:** la coord capturada va al campo `bed_*`. Para Phase 5+ se podra capturar tambien un `spawn_*` separado (donde se levanta el ped despues de despertar). Por ahora `bed = spawn`.

### Añadir NPCs

1. Pestaña **NPCs** → elige rol/modelo/scenario antes de capturar.
2. **Roles disponibles**:
   - `doctor` — tratamiento del paciente
   - `receptionist` — interaccion social (placeholder Phase 6+)
   - `duty` — toggle on/off (placeholder Phase 6+)
   - `paramedic` — vehiculos / reanimacion (placeholder)
3. **Modelo**: cualquier hash valido (`s_m_m_doctor_01` por defecto).
4. **Scenario** (opcional): GTA scenario name para que el NPC haga animaciones (`WORLD_HUMAN_CLIPBOARD`, `WORLD_HUMAN_AA_COFFEE`, etc).
5. Click **Capturar posicion del NPC** → se mueve al sitio + presiona `[E]`.

### Whitelist de jobs

1. Pestaña **Jobs** → input nombre del job + grado minimo.
2. Click **Añadir**.
3. Lista vacia = el hospital acepta a CUALQUIER job en `Config.MedicalJobs`. Añadir filas restringe.

### Editar / eliminar

- Cualquier campo del Info tab → click Guardar y persiste.
- Boton papelera junto al titulo elimina el hospital + todas sus camas/NPCs/jobs (CASCADE).
- Camas / NPCs individuales tienen su propio boton papelera con confirmacion.

---

## UX detalles

- **Captura de coordenadas**: el NUI se oculta sin perder estado mientras estas in-game. Vuelve al mismo formulario despues de capturar o cancelar.
- **Multi-hospital simultaneo**: tener N hospitales `enabled = true` es lo normal. El respawn busca la cama mas cercana entre todos.
- **Live reload**: cualquier guardado dispara un broadcast `:client:hospitalsSync` que rebuiltea blips y NPCs al instante en todos los clientes conectados, sin reiniciar el recurso.
- **Sin /reload necesario**: solo si modificas la DB manualmente (insert/update por SQL) tienes que ejecutar `/nbamb-reload` para refrescar el cache server.

---

## Esquema SQL

```sql
nb_ambulance_hospitals  → 1 fila por hospital
nb_ambulance_beds       → N camas, FK ON DELETE CASCADE
nb_ambulance_npcs       → N NPCs, FK ON DELETE CASCADE
nb_ambulance_jobs       → whitelist, UNIQUE(hospital_id, job_name)
```

Eliminar un hospital borra automaticamente todo lo relacionado.
