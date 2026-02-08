# Nombre del Script/Recurso

Breve descripción del recurso (1-2 frases).

## Requisitos

- **OneSync:** Infinity (o el que necesites)
- **Dependencias:** (otros recursos que deban estar antes)
- **Servidor:** Build recomendado

## Instalación

1. Descarga o clona el recurso en `resources/[categoria]/nombre-recurso`.
2. Añade en `server.cfg`:
   ```cfg
   ensure nombre-recurso
   ```
3. Reinicia el servidor o el recurso.

## Configuración

Indica el archivo de config (p. ej. `config.lua` o convars).

| Parámetro | Descripción | Por defecto |
|-----------|-------------|-------------|
| `Config.Ejemplo` | Qué hace | `true` |

## Uso

- **Comandos:** `/comando` – descripción.
- **Permisos/ACE:** si aplica.

## API / Eventos

Si otros recursos pueden integrarse:

```lua
-- Ejemplo: disparar evento
TriggerEvent('nombreRecurso:nombreEvento', datos)
```

## FAQ / Problemas conocidos

- **P: Error X al iniciar**  
  R: Comprueba que...

*(Elimina esta plantilla o renómbrala para que no aparezca en la navegación. En `mkdocs.yml` no añadas `_plantilla-script.md` al nav.)*
