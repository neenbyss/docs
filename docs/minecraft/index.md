# Minecraft Mods

Documentación de mods para Minecraft. Cada enlace lleva a la guía del proyecto correspondiente.

## Listado de mods

| Proyecto | Descripción |
|----------|-------------|
| *(Añade aquí cada mod con enlace a su página)* | *(Breve descripción)* |

## Cómo añadir un nuevo mod

1. Crea un archivo `.md` en esta carpeta, por ejemplo `mi-mod.md`.
2. En `mkdocs.yml`, dentro de `Minecraft Mods`, añade una línea como:
   ```yaml
   - Nombre del Mod: minecraft/mi-mod.md
   ```
3. En la tabla de este índice, añade la fila con el nombre y enlace al nuevo archivo.

## Plantilla para una guía de mod

Puedes copiar la estructura de `_plantilla-mod.md` (si existe) o usar:

- **Requisitos** (versión de Minecraft, Forge/Fabric, etc.)
- **Instalación** (pasos para instalar el mod)
- **Configuración** (archivos de config, opciones)
- **Uso** (cómo se usa en juego)
- **FAQ / Problemas conocidos** (opcional)
