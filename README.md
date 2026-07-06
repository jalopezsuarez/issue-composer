# Issue Composer

Single-page web app (a single `index.html`, **no dependencies or build**) that supercharges the issue tracker you already have on GitHub: **write and publish code-aware issues with the help of an LLM**, manage them on a **Kanban board built on plain `status:` labels** (no GitHub Projects setup), comment on them, and run the whole tracker from a **conversational AI assistant** with real tools. Designed as an **iPhone-style mobile app** (iOS + GitHub aesthetic, monochrome, *glass* effect, light/dark/auto), meant to be used from the phone but fully functional on the desktop.

Everything it creates is a **plain GitHub issue, label or comment** — your data never leaves GitHub, there's no second tracker to sync and no lock-in.

🔗 **Demo (GitHub Pages):** https://jalopezsuarez.github.io/issue-composer/
📣 **Landing page:** https://jalopezsuarez.github.io/issue-composer/web/

> All the logic lives in the browser. **There is no backend**: requests go directly to the GitHub API and to your LLM provider. Your credentials are stored only in your browser's `localStorage`.

---

## Contents

- [Views](#views)
- [Writing modes](#writing-modes)
- [AI assistant](#-ai-assistant)
- [Search](#search)
- [Getting started](#getting-started)
  - [1. GitHub token](#1-github-token)
  - [2. LLM provider](#2-llm-provider)
  - [3. Language](#3-language)
  - [4. Theme](#4-theme)
- [Gestures and interaction](#gestures-and-interaction)
- [How it works under the hood](#how-it-works-under-the-hood)
  - [GitHub integration](#github-integration)
  - [LLM integration](#llm-integration)
  - [Kanban statuses and order](#kanban-statuses-and-order)
  - [Persistence (localStorage)](#persistence-localstorage)
- [Privacy and security](#privacy-and-security)
- [Deployment](#deployment)
- [UI and compatibility notes](#ui-and-compatibility-notes)
- [Known limitations](#known-limitations)
- [Project structure](#project-structure)

---

## Views

Navigation is a **floating glass capsule** (bottom right) with four destinations — **Kanban**, **Issues**, **AI assistant** and **Settings** — plus a **+** button to create and a 🔍 button to search.

### 🗂️ Kanban
Board by **status**, one column per fixed status. Each card shows the title, an excerpt of the body (2 lines), the number, comments and the last-updated date. It allows:
- **Dragging between columns** to change the issue's status.
- **Reordering within a column** by dragging vertically (persistent manual order).
- Tapping a card to open its **detail**.

Order within each column: first the ones with a manual order (`order:n` ascending), then the rest by date (most recent on top); closed ones at the bottom.

### 📋 Issues
List with **infinite scroll** (25 per page) and navigation to the detail. Each row shows on two lines: the `open/closed` capsule + the status chip, and `#no · user · comments`. A **sort** toggle (Newest / Oldest) lives inside the search bar.

### ✏️ Create (+ button)
AI issue composer:
1. You write a **note** describing what you want.
2. You choose the writing **mode**.
3. **Generate with AI**: if the mode requires it, the AI **reviews the codebase** (README + up to 5 files it picks as relevant) and writes a **title + body** in Markdown.
4. You review/edit the result and **Publish** the issue.

### 🔍 Issue detail
- Description rendered in **Markdown** (headings, lists, checklists, code, quotes, links…).
- The user's **labels** (the internal `status:`/`order:` ones stay hidden).
- **Status**: a row of options to change it instantly (stored as a label).
- **Close / Reopen** the issue.
- Link to **GitHub**.
- **Comments**: listed, can be **created with AI** (with their own mode selector) and your own can be **edited/deleted** (per permissions).
- Tapping the **title** opens the editor to rename it.

### ✨ AI assistant
A **ChatGPT-style** conversational view (max 720px) scoped to the **active repository's issues and code**. Using the configured LLM with **tool-calling**, it can:
- read and search issues, read the source files and the README;
- create issues, add and edit comments;
- change an issue's status and edit its title/description.

Each tool call is shown as a small chip while it runs. Conversations are **not persisted** (they reset when you leave). It refuses topics unrelated to this repo.

### ⚙️ Settings
- **Active repo banner**, pinned at the top: **yellow** for public repos, **green with a lock** for private ones (privacy is detected on selection and cached).
- **Repositories**: a **Recent / Favorites** segmented list. Tap a row to select it, the ✕ removes it, and private repos show a lock. Mark the active repo as a favorite with the **heart** button (top right).
- **GitHub**: token + repository search/selection (or manual `owner/repo`).
- **LLM provider**: Base URL, API key and Model.
- **Generation language**: Spanish / English.
- **Appearance**: Light / Dark / Auto theme.
- **Settings export / import**: 100% of the settings (token, LLM, active repo, recents, favorites, mode, language and theme), exported **encrypted** (AES-256-GCM via Web Crypto) as an opaque blob plus a randomly generated **encryption key** — copy both with their buttons, and paste both on the other device to import (with paste-from-clipboard buttons).

### 🖊️ Full-screen editor
A reusable view (a single borderless, full-screen `textarea`) to edit **one element at a time**: the issue **title**, its **body** or a **comment**. **Back** button (cancel) at the top left and **save** (↑) at the top right. It adapts dynamically to the visible height and the keyboard.

---

## Writing modes

Available when creating issues and when writing comments:

| Mode | Reviews the code | What it produces |
|------|:---:|-------------|
| **Simple** | No (instant) | Transforms your text as little as possible: fixes it and applies light Markdown formatting, keeping your words and intent. Doesn't add sections or new information. |
| **Feature** | Yes | Develops a **new feature** as a User Story / PBI: *User story, Description and context, Status in the code, Scope, Areas/files involved, Acceptance criteria, Technical notes and dependencies*. |
| **Bug** | Yes | **Bug** report: *Description, Reproduction steps, Expected vs actual behavior, Status in the code, Files involved, Probable root cause, Impact, Edge cases, Proposed fix, Acceptance criteria*. |

In the **Feature** and **Bug** modes, the AI first analyzes the code and the README to assess whether what you describe makes sense and what **state** it's in (already implemented, partial or pending), and reflects it in the *Status in the code* section citing **real** paths and symbols (it doesn't make things up). In comments it also takes into account the **issue description and the thread's previous comments** to respond consistently.

---

## Search

Both the **Kanban** and the **Issues** list have a 🔍 button that toggles a search bar above the board/list. It queries the **GitHub Search API** server-side and filters what's shown; the ✕ clears it. The open/closed state of the bar is remembered per view. In the Issues list, the **sort** toggle (Newest / Oldest) sits inside the bar and disappears with it.

Empty, loading and error states are **uniform** across both views (e.g. *"Enter your token in Settings."*, *"Select a repository in Settings."*, *"No issues in this repository."*).

---

## Getting started

Open the [demo](https://jalopezsuarez.github.io/issue-composer/) (or your deployment) and go to **Settings**.

### 1. GitHub token

Create a **Personal Access Token** on GitHub and paste it into the *Personal Access Token* field. Press **Connect** to load your repositories (your own, as a collaborator, and from your organizations), or type `owner/repo` by hand.

Recommended permissions:
- **Fine-grained token**: access to the desired repos with *Issues* (read and write) and *Metadata* (read) permissions.
- **Classic token**: `repo` scope.
- If your organization uses **SAML SSO**, authorize the token for that organization.

> A **404** when adding a repo almost always means the token can't see it (insufficient permissions, SSO not authorized, or `owner/repo` misspelled).

### 2. LLM provider

Compatible with any **OpenAI-style** API:

| Field | Example |
|-------|---------|
| **Base URL** | `https://api.openai.com/v1` |
| **API key** | `sk-…` |
| **Model** | `gpt-4o-mini` |

It does `POST {baseURL}/chat/completions` with `Authorization: Bearer`, `max_tokens: 1500`, `temperature: 0.3`, and reads `data.choices[0].message.content`.

> **CORS**: if the LLM fails with "Failed to fetch", enable CORS for the app's origin in your provider/proxy.

### 3. Language

Choose **Spanish** or **English** for the language of the generated issues and comments (the interface itself is in English).

### 4. Theme

Choose **Light**, **Dark** or **Auto** (follows the system preference and updates live when it changes). The choice is saved, applied before the first paint (no flash), and travels with the settings export/import.

---

## Gestures and interaction

- **Tap card / row** → opens the detail.
- **Drag card** (Kanban): mouse, starts on move; touch, **long press** (~180 ms) to "pick up" the card. Auto-scroll near the edges.
- **Horizontal drag** → changes column (status). **Vertical drag** → reorders within the column.
- **+ button** → opens the issue composer. **🔍 button** → toggles the search bar.
- **Reload** (icon on the left in list/kanban) → reloads from the network.
- **Back** (‹, top left) → goes back; **Save** (↑, top right) in the editor.
- Zoom disabled (no pinch or double-tap) to behave like a native app.

---

## How it works under the hood

### GitHub integration

REST API v3 (`https://api.github.com`) with headers:

```
Authorization: Bearer <token>
Accept: application/vnd.github+json
X-GitHub-Api-Version: 2022-11-28
```

Endpoints used: `GET /user`, `GET /user/repos`, `GET /user/orgs`, `GET /orgs/{org}/repos`, `GET /repos/{repo}`, `GET/POST /repos/{repo}/issues`, `GET /search/issues` (search), `PATCH /repos/{repo}/issues/{n}`, `PUT /repos/{repo}/issues/{n}/labels`, `POST /repos/{repo}/labels`, `GET/POST/PATCH/DELETE .../issues/comments`, `GET .../readme`, `GET .../git/trees/{branch}?recursive=1`, `GET .../contents/{path}`.

Reads use `cache: "no-store"` so changes are reflected on reload (GitHub responds with `Cache-Control: private, max-age=60`).

### LLM integration

- **Code selection** (Feature/Bug modes): the default branch, the README and the file tree are cached per session; the LLM picks up to **5 relevant paths** that are downloaded and passed as context.
- **Issue writing**: responds ONLY with a JSON `{"title","body"}` (tolerant parsing that strips ```` ```json ```` fences).
- **Comment writing**: responds ONLY with the comment's Markdown text.

### Kanban statuses and order

**Fixed** statuses, stored as `status:<key>` labels:

`pending` · `progress` · `review` · `done` · `archive` · `resources`

Each status has a color that adapts to the theme (soft pastels in light mode, a muted Notion-style dark palette in dark mode). The **manual order** within a column is stored as an `order:<n>` label. Both label families are internal and are **not shown** as user chips. The status change is done in a single `PUT .../labels` call (creating the label if it doesn't exist).

### Persistence (localStorage)

| Key | Content |
|-------|-----------|
| `ic_gh_token` | GitHub Personal Access Token |
| `ic_llm_base_url` / `ic_llm_api_key` / `ic_llm_model` | LLM configuration |
| `ic_repo` | Active repository |
| `ic_recent` | Recent repositories (max. 10) |
| `ic_favs` | Favorite repositories (max. 10) |
| `ic_tone` | Writing mode |
| `ic_lang` | Generation language |
| `ic_theme` | UI theme (light/dark/auto) |
| `ic_tab` | Last used tab |
| `ic_repo_tab` | Recent/Favorites segment |

Additional internal helpers not included in export/import: `ic_repo_priv` (per-repo public/private cache) and `ic_srch_kanban` / `ic_srch_issues` (search-bar open state per view).

---

## Privacy and security

- The **GitHub token** and the **LLM credentials** are stored in the browser's `localStorage` (at the user's request): they stay on the browser's disk and are accessible to same-origin scripts.
- Use it on a **trusted device** and with a **minimal-permissions** token.
- No backend or telemetry: the calls go directly from the browser to GitHub and to the LLM.

---

## Deployment

It's a **static** site (a single `index.html`). It's published on **GitHub Pages** with the [`.github/workflows/static.yml`](.github/workflows/static.yml) workflow, which uploads the repo root on every push to `main`. The marketing landing page under [`web/`](web/) is published alongside it.

To deploy it to your own repo:
1. Copy `index.html` (and optionally `web/index.html` and `.github/workflows/static.yml`).
2. In **Settings → Pages**, choose **GitHub Actions** as the source.
3. Push to `main`.

You can also open `index.html` directly in the browser (no server) to use it locally.

---

## UI and compatibility notes

- **Vanilla**: HTML + CSS + JS embedded in a single file, no frameworks or build process.
- **iOS/glass aesthetic**: `backdrop-filter` (blur), translucent floating buttons, system typography, monochrome SVG icons (octicons). A **borderless** design where sections are separated only by their title, and lines appear only as row separators inside lists (issues, comments, recents/favorites, status selector).
- **Light & dark**: a full dark theme, chosen in Settings and honored for the iOS status-bar tint.
- **Responsive width**: on the desktop the **Kanban and Issues** views use the **full browser width**, while **Create, Detail, Settings and the editor** are capped at **720 px, centered**. On the phone it's a single full-width column.
- **Safe areas**: respects `env(safe-area-inset-*)`; the top strip of the *safe area* and the background behind the keyboard follow the theme.
- **Dynamic viewport**: the height adapts to the visible viewport (`visualViewport`) so the editor and its textarea fit with the keyboard open.
- Designed for recent **Safari/Chrome** on mobile; works well on desktop too.

---

## Known limitations

- **LLM CORS**: depends on your provider/proxy allowing the app's origin.
- No real-time sync: use **Reload** to bring in changes made outside the app.
- Search relies on the GitHub Search API, which is rate-limited and slightly delayed vs. the plain issues listing.

---

## Project structure

```
.
├── index.html                     # The whole app (HTML + CSS + JS)
├── web/
│   └── index.html                 # Marketing landing page (self-contained)
├── favicon.png                    # App icons
├── apple-touch-icon.png
├── README.md
└── .github/
    └── workflows/
        └── static.yml             # Deployment to GitHub Pages
```
