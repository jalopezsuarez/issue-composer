# Issue Composer

Aplicación web de una sola página (un único `index.html`, **sin dependencias ni build**) para **redactar y publicar issues de GitHub con ayuda de un LLM**, gestionarlos en un **tablero Kanban** y comentarlos. Diseñada como una **app móvil estilo iPhone** (estética iOS + GitHub, monocromo, efecto *glass*), pensada para usarse desde el teléfono pero funcional en cualquier navegador moderno.

🔗 **Demo (GitHub Pages):** https://jalopezsuarez.github.io/issue-composer/

> Toda la lógica vive en el navegador. **No hay backend**: las peticiones salen directamente a la API de GitHub y a tu proveedor LLM. Tus credenciales se guardan solo en el `localStorage` de tu navegador.

---

## Índice

- [Vistas](#vistas)
- [Modos de redacción (tono)](#modos-de-redacción-tono)
- [Puesta en marcha](#puesta-en-marcha)
  - [1. Token de GitHub](#1-token-de-github)
  - [2. Proveedor LLM](#2-proveedor-llm)
  - [3. Idioma](#3-idioma)
- [Gestos e interacción](#gestos-e-interacción)
- [Cómo funciona por dentro](#cómo-funciona-por-dentro)
  - [Integración con GitHub](#integración-con-github)
  - [Integración con el LLM](#integración-con-el-llm)
  - [Estados y orden del Kanban](#estados-y-orden-del-kanban)
  - [Persistencia (localStorage)](#persistencia-localstorage)
- [Privacidad y seguridad](#privacidad-y-seguridad)
- [Despliegue](#despliegue)
- [Notas de UI y compatibilidad](#notas-de-ui-y-compatibilidad)
- [Limitaciones conocidas](#limitaciones-conocidas)
- [Estructura del proyecto](#estructura-del-proyecto)

---

## Vistas

La navegación es una **cápsula flotante** (abajo a la derecha) con tres destinos, más un botón **+** para crear:

### 🗂️ Kanban
Tablero por **estado** con una columna por cada estado fijo. Cada tarjeta muestra título, un extracto del cuerpo (2 líneas), número, comentarios y fecha de actualización. Permite:
- **Arrastrar entre columnas** para cambiar el estado de la incidencia.
- **Reordenar dentro de una columna** arrastrando verticalmente (orden manual persistente).
- Tocar una tarjeta para abrir su **detalle**.

Orden dentro de cada columna: primero las que tienen orden manual (`order:n` ascendente), luego el resto por fecha (más reciente arriba); las cerradas al fondo.

### 📋 Incidencias
Listado con **scroll infinito** (25 por página), orden **Recientes/Antiguas** conmutable, y navegación al detalle. Cada fila muestra en dos líneas: cápsula `open/closed` + estado, y `#nº · usuario · comentarios · fecha`.

### ✏️ Crear (botón +)
Compositor de issues con IA:
1. Escribes una **nota** describiendo lo que quieres.
2. Eliges el **modo** de redacción.
3. **Generar con IA**: si el modo lo requiere, la IA **revisa el código base** (README + hasta 5 ficheros que ella elige como relevantes) y redacta **título + cuerpo** en Markdown.
4. Revisas/editas el resultado y **Publicas** el issue.

### 🔍 Detalle de incidencia
- Descripción renderizada en **Markdown** (encabezados, listas, checklists, código, citas, enlaces…).
- **Etiquetas** del usuario (las internas `status:`/`order:` quedan ocultas).
- **Estado**: fila para cambiarlo al instante (se guarda como etiqueta).
- **Cerrar / Reabrir** la incidencia.
- Enlace a **GitHub**.
- **Comentarios**: se listan, se pueden **crear con IA** (con su propio selector de modo) y **editar/borrar** los propios (según permisos).
- Tocar el **título** abre el editor para renombrarlo.

### ⚙️ Configuración
- **Repositorios recientes** (los últimos 10 usados) para seleccionar con un toque.
- **Repo activo** (banner amarillo).
- **GitHub**: token + búsqueda/selección de repositorio (o `owner/repo` manual).
- **Proveedor LLM**: Base URL, API key y Model.
- **Idioma de generación**: Español / English.

### 🖊️ Editor a pantalla completa
Vista reutilizable (un único `textarea` sin bordes, a pantalla completa) para editar **un elemento a la vez**: el **título** de la incidencia, su **cuerpo** o un **comentario**. Botón **atrás** (cancelar) arriba a la izquierda y **guardar** (↑) arriba a la derecha. Se ajusta dinámicamente al alto visible y al teclado.

---

## Modos de redacción (tono)

Disponibles al crear issues y al redactar comentarios:

| Modo | Revisa el código | Qué produce |
|------|:---:|-------------|
| **Simple** | No (instantáneo) | Transforma lo mínimo tu texto: corrige y da formato Markdown ligero, conservando tus palabras e intención. No añade secciones ni información nueva. |
| **Funcionalidad** | Sí | Desarrolla una **nueva funcionalidad** como Historia de Usuario / PBI: *Historia de usuario, Descripción y contexto, Estado en el código, Alcance, Áreas/archivos implicados, Criterios de aceptación, Notas técnicas y dependencias*. |
| **Incidencia** | Sí | Informe de **bug**: *Descripción, Pasos de reproducción, Comportamiento esperado vs actual, Estado en el código, Archivos implicados, Causa raíz probable, Impacto, Casos límite, Propuesta de solución, Criterios de aceptación*. |

En los modos **Funcionalidad** e **Incidencia**, la IA analiza primero el código y el README para valorar si lo que describes tiene sentido y en qué **estado** está (ya implementado, parcial o pendiente), y lo refleja en la sección *Estado en el código* citando rutas y símbolos **reales** (no inventa). En los comentarios también tiene en cuenta la **descripción de la incidencia y los comentarios previos** del hilo para responder de forma coherente.

---

## Puesta en marcha

Abre la [demo](https://jalopezsuarez.github.io/issue-composer/) (o tu despliegue) y ve a **Configuración**.

### 1. Token de GitHub

Crea un **Personal Access Token** en GitHub y pégalo en el campo *Personal Access Token*. Pulsa **Conectar** para cargar tus repositorios (propios, como colaborador y de tus organizaciones), o escribe `owner/repo` a mano.

Permisos recomendados:
- **Token fine-grained**: acceso a los repos deseados con permisos de *Issues* (lectura y escritura) y *Metadata* (lectura).
- **Token classic**: scope `repo`.
- Si tu organización usa **SAML SSO**, autoriza el token para esa organización.

> Un **404** al añadir un repo casi siempre significa que el token no puede verlo (permisos insuficientes, SSO sin autorizar o `owner/repo` mal escrito).

### 2. Proveedor LLM

Compatible con cualquier API **estilo OpenAI**:

| Campo | Ejemplo |
|-------|---------|
| **Base URL** | `https://api.openai.com/v1` |
| **API key** | `sk-…` |
| **Model** | `gpt-4o-mini` |

Se hace `POST {baseURL}/chat/completions` con `Authorization: Bearer`, `max_tokens: 1500`, `temperature: 0.3`, y se lee `data.choices[0].message.content`.

> **CORS**: si el LLM falla con «Failed to fetch», habilita CORS para el origen de la app en tu proveedor/proxy.

### 3. Idioma

Elige **Español** o **English** para el idioma de los issues y comentarios generados.

---

## Gestos e interacción

- **Tocar tarjeta / fila** → abre el detalle.
- **Arrastrar tarjeta** (Kanban): ratón, arranca al mover; táctil, **pulsación larga** (~180 ms) para «coger» la tarjeta. Auto-scroll cerca de los bordes.
- **Arrastre horizontal** → cambia de columna (estado). **Arrastre vertical** → reordena dentro de la columna.
- **Botón +** → abre el compositor de issues.
- **Recargar** (icono a la izquierda en listado/kanban) → recarga desde la red.
- **Atrás** (‹, arriba-izquierda) → vuelve; **Guardar** (↑, arriba-derecha) en el editor.
- Zoom desactivado (sin pinch ni doble-tap) para comportarse como app nativa.

---

## Cómo funciona por dentro

### Integración con GitHub

API REST v3 (`https://api.github.com`) con cabeceras:

```
Authorization: Bearer <token>
Accept: application/vnd.github+json
X-GitHub-Api-Version: 2022-11-28
```

Endpoints usados: `GET /user`, `GET /user/repos`, `GET /user/orgs`, `GET /orgs/{org}/repos`, `GET /repos/{repo}`, `GET/POST /repos/{repo}/issues`, `PATCH /repos/{repo}/issues/{n}`, `PUT /repos/{repo}/issues/{n}/labels`, `POST /repos/{repo}/labels`, `GET/POST/PATCH/DELETE .../issues/comments`, `GET .../readme`, `GET .../git/trees/{branch}?recursive=1`, `GET .../contents/{path}`.

Las lecturas usan `cache: "no-store"` para que los cambios se reflejen al recargar (GitHub responde con `Cache-Control: private, max-age=60`).

### Integración con el LLM

- **Selección de código** (modos Funcionalidad/Incidencia): se cachea por sesión la rama por defecto, el README y el árbol de ficheros; el LLM elige hasta **5 rutas** relevantes que se descargan y se le pasan como contexto.
- **Redacción de issue**: responde SOLO con un JSON `{"title","body"}` (parseo tolerante que quita fences ```` ```json ````).
- **Redacción de comentario**: responde SOLO con el texto Markdown del comentario.

### Estados y orden del Kanban

Estados **fijos**, guardados como etiquetas `status:<clave>`:

`pending` · `progress` · `review` · `done` · `archive` · `resources`

El **orden manual** dentro de una columna se guarda como etiqueta `order:<n>`. Ambas familias de etiquetas son internas y **no se muestran** como chips de usuario. El cambio de estado se hace en una sola llamada `PUT .../labels` (creando la etiqueta si no existe).

### Persistencia (localStorage)

| Clave | Contenido |
|-------|-----------|
| `ic_gh_token` | GitHub Personal Access Token |
| `ic_llm_base_url` / `ic_llm_api_key` / `ic_llm_model` | Configuración del LLM |
| `ic_repo` | Repositorio activo |
| `ic_recent` | Repositorios recientes (máx. 10) |
| `ic_favs` | Repositorios favoritos (máx. 10) |
| `ic_tone` | Modo de redacción |
| `ic_lang` | Idioma de generación |
| `ic_tab` | Última pestaña usada |

---

## Privacidad y seguridad

- El **GitHub token** y las **credenciales del LLM** se guardan en el `localStorage` del navegador (a petición del usuario): quedan en el disco del navegador y son accesibles a scripts del mismo origen.
- Úsalo en un **equipo de confianza** y con un token de **permisos mínimos**.
- Sin backend ni telemetría: las llamadas van directas del navegador a GitHub y al LLM.

---

## Despliegue

Es un sitio **estático** (un único `index.html`). Se publica en **GitHub Pages** con el workflow [`.github/workflows/static.yml`](.github/workflows/static.yml), que sube la raíz del repo en cada push a `main`.

Para desplegarlo en tu propio repo:
1. Copia `index.html` (y opcionalmente `.github/workflows/static.yml`).
2. En **Settings → Pages**, elige **GitHub Actions** como origen.
3. Haz push a `main`.

También puedes abrir `index.html` directamente en el navegador (sin servidor) para usarlo en local.

---

## Notas de UI y compatibilidad

- **Vanilla**: HTML + CSS + JS embebidos en un solo archivo, sin frameworks ni proceso de build.
- **Estética iOS/glass**: `backdrop-filter` (blur), botones flotantes translúcidos, tipografía del sistema, iconos SVG monocromos (octicons).
- **Safe areas**: respeta `env(safe-area-inset-*)`; la franja superior del *safe-area* y el fondo tras el teclado son transparentes.
- **Viewport dinámico**: la altura se ajusta al viewport visible (`visualViewport`) para que el editor y su textarea encajen con el teclado abierto.
- Pensado para **Safari/Chrome recientes** en móvil; funciona en escritorio (ancho máximo 480 px, centrado).

---

## Limitaciones conocidas

- **Transparencia del safe-area**: en una pestaña normal de Safari el lienzo del navegador es blanco, así que la zona transparente puede verse blanca; se aprovecha mejor como **PWA instalada** o según el fondo del sistema.
- **CORS del LLM**: depende de que tu proveedor/proxy permita el origen de la app.
- No hay sincronización en tiempo real: usa **Recargar** para traer cambios hechos fuera de la app.

---

## Estructura del proyecto

```
.
├── index.html                     # Toda la app (HTML + CSS + JS)
├── README.md
└── .github/
    └── workflows/
        └── static.yml             # Despliegue a GitHub Pages
```
