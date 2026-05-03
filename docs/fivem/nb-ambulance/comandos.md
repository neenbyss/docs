# Comandos

---

## Comandos directos (alias por defecto)

| Comando | Quien | Que hace |
|---------|-------|----------|
| `/911` | cualquiera | Crea una llamada 911 desde tu posicion. Util para civiles testigos o para llamar sin estar en laststand. |
| `/dispatch` | medico/admin | Abre el panel de llamadas activas. |
| `/treat [id]` | medico/admin | Abre el NUI de tratamiento del paciente. Sin id captura el mas cercano a 4m. |
| `/revive [id]` | medico/admin | Reanima al objetivo. Sin id solo admin (revive a si mismo). |
| `/creator` | admin | Abre el editor de hospitales in-game. |
| `/hospitals` | cualquiera | Imprime la lista de hospitales activos en la consola del cliente. |
| `/nbamb-reload` | admin | Recarga hospitales desde la DB sin reiniciar el recurso. |
| `/nb-ambulance-revive <id>` | consola/admin | Equivalente a `/revive` desde rcon o consola del server. |
| `/nb-ambulance-pvp <id> <0\|1>` | consola/admin | Toggle del flag PVP-safe en un jugador. |

Todos los nombres son configurables en `Config.Commands` (`shared/config.lua`).

---

## Comando umbrella (fallback)

```
/nb-ambulance <subcommand>
```

Subcomandos: `creator`, `hospitals`, `reload`, `revive [id]`, `treat [id]`, `dispatch`, `911`, `help`.

Util si renombras los alias y prefieres un solo punto de entrada.

---

## Key bindings

| Tecla por defecto | Accion | Equivalente |
|------------------|--------|-------------|
| **F6** | Abrir dispatch panel | `/dispatch` |
| `[G]` (in laststand) | Llamar 911 | — |
| `[E]` (in laststand) | Rendirse / acelerar muerte | — |
| `[E]` (cerca de paciente downed) | RCP / desfibrilador (medicos) | — |
| `[E]` (in dead state) | Respawn al hospital mas cercano | — |

Cada keybind se puede rebindar desde el menu de FiveM (`Settings → Key Bindings → FiveM`).

---

## Permisos

- **Admin**: cualquier comando (auto-otorgado por `Bridge.IsAdmin`, depende del framework — ESX `group`, QBCore `permissions`).
- **Medico**: jugador con `job.name` en `Config.MedicalJobs`. Puede tratar/reanimar a otros, abrir dispatch.
- **Civil**: solo `/911` y `/hospitals`.

Cuando un comando se ejecuta sin permisos suficientes, sale el toast `no_permission` y la accion se ignora silenciosamente del lado del server.
