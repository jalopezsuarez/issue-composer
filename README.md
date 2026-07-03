# Issue Composer

Aplicación web de una sola página (un único `index.html`, sin dependencias ni build) para **redactar y publicar issues de GitHub con ayuda de IA**, gestionarlos en un **tablero Kanban** y comentarlos. Diseñada como una **app móvil estilo iPhone** (estética iOS/GitHub, monocromo, glass), pensada para usarse desde el móvil.

🔗 **Demo:** https://jalopezsuarez.github.io/issue-composer/

## Características

- **Crear issues con IA**: escribes una nota y el LLM redacta título y cuerpo en Markdown. Antes de redactar, la IA puede **revisar el código base** del repo (README + ficheros relevantes) para ser precisa.
- **Modos de redacción** (tono), tanto para issues como para comentarios:
  - **Simple** — cambios mínimos, conserva tus palabras (no analiza el código, instantáneo).
  - **Estructurado** — informe claro y organizado (contexto, pasos, esperado vs actual, archivos implicados, criterios de aceptación).
  - **Técnico** — análisis de ingeniería que diagnostica (causa raíz, impacto, casos límite) y propone solución + plan de pruebas.
- **Tablero Kanban** por estado con **arrastrar y soltar** entre columnas y **orden manual** dentro de una columna (arrastre vertical). Estados fijos: `pending`, `progress`, `review`, `done`, `archive`, `resources`.
- **Listado de incidencias** con scroll infinito, orden por fecha (recientes/antiguas) y navegación al detalle.
- **Detalle de incidencia**: descripción en Markdown, etiquetas, cambio de estado, cerrar/reabrir, y **comentarios** (crear con IA, editar y borrar los propios).
- **Editor a pantalla completa** reutilizable para editar el título, el cuerpo o un comentario (un elemento cada vez).
- **Idioma de generación**: español o inglés.
- **Repositorios recientes** y selección/persistencia del repo activo.
- Persistencia en `localStorage` (token, ajustes del LLM, repo, tono, idioma, última pestaña y recientes).

## Uso

1. Abre la app (la demo o tu propio despliegue).
2. En **Configuración**:
   - Introduce tu **GitHub Personal Access Token** (con permiso sobre los repos que quieras usar) y pulsa **Conectar** para cargar tus repositorios; o escribe `owner/repo` a mano.
   - Configura el **Proveedor LLM** (compatible con OpenAI): **Base URL**, **API key** y **Model**.
   - Elige el **idioma** de generación.
3. Usa las pestañas: **Kanban**, **Incidencias** y **Configuración**; y el botón **+** para crear una incidencia.

### Proveedor LLM

Compatible con cualquier API estilo OpenAI. Se hace `POST {baseURL}/chat/completions` con autenticación `Bearer` y se lee `data.choices[0].message.content`.

- **Base URL**: p. ej. `https://api.openai.com/v1`
- **API key**: `sk-…`
- **Model**: p. ej. `gpt-4o-mini`

> Si el endpoint del LLM falla con «Failed to fetch», suele ser un problema de **CORS**: habilita CORS para el origen de la app.

## Privacidad y seguridad

- El **GitHub token** y las **credenciales del LLM** se guardan en el `localStorage` del navegador (a petición del usuario). Quedan en el disco del navegador y son accesibles a scripts del mismo origen.
- Úsalo en un equipo de confianza y con un token de **permisos mínimos**.
- La app no tiene backend: todas las llamadas salen del navegador directamente a la API de GitHub y a tu proveedor LLM.

## Despliegue

Es un sitio estático (un solo `index.html`). Se publica en **GitHub Pages** mediante el workflow [`.github/workflows/static.yml`](.github/workflows/static.yml), que despliega la raíz del repo en cada push a `main`.

Para usarlo en otro repo: copia `index.html`, activa GitHub Pages con GitHub Actions y haz push a `main`.

## Detalles técnicos

- **Vanilla**: HTML + CSS + JS embebidos en un único archivo, sin frameworks ni build.
- **API de GitHub**: REST v3 con cabeceras `Authorization: Bearer`, `Accept: application/vnd.github+json`, `X-GitHub-Api-Version: 2022-11-28`. Las lecturas usan `cache: no-store` para reflejar los cambios al instante.
- **Estados del Kanban** guardados como etiquetas `status:<clave>`; el **orden manual** como `order:<n>` (ambas ocultas como chips de usuario).
- **UI iOS**: `backdrop-filter` (glass), `env(safe-area-inset-*)`, altura ajustada al viewport visible (`visualViewport`) para que el editor y el teclado encajen, y safe-area superior transparente.
