# Neenbyss Docs

Documentación centralizada en **MkDocs + Material Theme** para mods de Minecraft, scripts de FiveM y otras guías. Publicada en **https://docs.neenbyss.com**.

## Estructura

```
docs/
├── mkdocs.yml          # Configuración y navegación
├── requirements.txt    # Dependencias Python
├── README.md           # Este archivo
└── docs/               # Contenido en Markdown
    ├── index.md        # Portada
    ├── minecraft/      # Mods Minecraft
    │   ├── index.md
    │   └── _plantilla-mod.md
    └── fivem/          # Scripts FiveM
        ├── index.md
        └── _plantilla-script.md
```

## Requisitos

- Python 3.8+

## Instalación y uso local

```bash
# Crear entorno virtual (recomendado)
python -m venv .venv
.venv\Scripts\activate   # Windows
# source .venv/bin/activate   # Linux/macOS

# Instalar dependencias
pip install -r requirements.txt

# Servidor de desarrollo (recarga al guardar)
mkdocs serve
```

Abre **http://127.0.0.1:8000** en el navegador.

## Generar sitio estático

```bash
mkdocs build
```

El sitio queda en la carpeta `site/`.

## Despliegue en GitHub Pages

El repo incluye un workflow que publica la documentación en **GitHub Pages** en cada push a `main` (o `master`).

### 1. Activar GitHub Pages

1. En tu repositorio: **Settings** → **Pages**.
2. En **Build and deployment** → **Source**, elige **GitHub Actions**.

### 2. (Opcional) Dominio personalizado docs.neenbyss.com

El archivo `docs/CNAME` contiene `docs.neenbyss.com`. Al hacer build, se incluye en la raíz del sitio y GitHub Pages lo usa para el dominio personalizado.

En **Settings** → **Pages** → **Custom domain**, escribe `docs.neenbyss.com` y guarda. Luego en tu proveedor de DNS (donde está neenbyss.com) añade:

| Tipo  | Nombre | Valor           |
|-------|--------|-----------------|
| CNAME | docs   | `tu-usuario.github.io` |

*(Sustituye `tu-usuario` por tu usuario o organización de GitHub.)*

Tras el primer deploy, la documentación quedará en:

- **GitHub:** `https://tu-usuario.github.io/nombre-repo/`
- **Con dominio:** `https://docs.neenbyss.com` (si configuraste CNAME y DNS)

Si publicas solo con la URL de GitHub (sin dominio propio), en `mkdocs.yml` cambia `site_url` a la URL de tu sitio (p. ej. `https://tu-usuario.github.io/docs/`) para que los enlaces y recursos carguen bien.

### 3. Ejecutar el deploy

- **Automático:** cada push a `main` (o `master`) ejecuta el workflow y actualiza la web.
- **Manual:** en la pestaña **Actions** → **Deploy Docs to GitHub Pages** → **Run workflow**.

## Añadir un nuevo proyecto

1. **Crear la página:** nuevo `.md` en `docs/minecraft/` o `docs/fivem/` (usa las plantillas `_plantilla-*.md`).
2. **Enlazar en la navegación:** edita `mkdocs.yml` y añade la entrada en la sección `nav` correspondiente (Minecraft Mods o FiveM Scripts).
3. **Actualizar el índice:** en `docs/minecraft/index.md` o `docs/fivem/index.md`, añade una fila en la tabla con el nombre y enlace al nuevo archivo.

Los archivos que empiezan por `_` (como las plantillas) no hace falta listarlos en `nav`; no se incluyen en el menú si no los referencias.
