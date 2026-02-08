# FiveM Scripts

Documentación de scripts y recursos para servidores FiveM. Cada enlace lleva a la guía del recurso.

## Listado de recursos

| Recurso | Descripción |
|---------|-------------|
| *(Añade aquí cada script/recurso con enlace a su página)* | *(Breve descripción)* |

## Cómo añadir un nuevo script/recurso

1. Crea un archivo `.md` en esta carpeta, por ejemplo `mi-script.md`.
2. En `mkdocs.yml`, dentro de `FiveM Scripts`, añade una línea como:
   ```yaml
   - Nombre del Script: fivem/mi-script.md
   ```
3. En la tabla de este índice, añade la fila con el nombre y enlace al nuevo archivo.

## Plantilla para una guía de script

Puedes usar esta estructura:

- **Requisitos** (OneSync, dependencias, versión del servidor)
- **Instalación** (dónde colocar el recurso, `ensure`)
- **Configuración** (archivo config, convars)
- **Uso** (comandos, eventos, permisos)
- **API / Eventos** (si expone eventos para otros scripts)
- **FAQ / Problemas conocidos** (opcional)
