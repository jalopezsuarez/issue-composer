# Issue Composer

**The AI issue tracker that lives inside GitHub.**

Turn rough notes into professional, code-aware GitHub issues. Run your board on plain issue labels. Manage the whole tracker by chatting with an AI. All from a single HTML file that runs in your browser — no backend, no accounts, no lock-in.

🔗 **App (GitHub Pages):** https://jalopezsuarez.github.io/issue-composer/
📣 **Landing page:** https://jalopezsuarez.github.io/issue-composer/web/

---

## Why Issue Composer

Writing good issues is slow. Keeping a board tidy is tedious. Most tools solve this by pulling your work *out* of GitHub into yet another tracker you have to sync, pay for and migrate away from someday.

Issue Composer takes the opposite bet: **GitHub is already your tracker — it just needs superpowers.**

- ✍️ **AI that writes issues like a senior engineer.** Describe what you want in one rough sentence. The AI reads your README and the relevant source files first, then writes a structured issue — user story or bug report — citing *real* paths and symbols, never inventions.
- 🗂️ **A Kanban board with zero setup.** Statuses are plain `status:` labels. No GitHub Projects to configure, no permissions dance — the board works on any repo you can open issues on, in seconds.
- ✨ **A conversational assistant with real tools.** Ask "what's still pending?", "summarize the thread on #42", "move it to review and open a follow-up" — the assistant reads and searches issues, digs into the code, creates issues, comments and changes statuses for you.
- 📱 **Feels like a native iPhone app.** Monochrome iOS + GitHub aesthetic, glass floating buttons, light/dark/auto theme, adjustable type size — installable on your home screen, great on the desktop too.
- 🔒 **Private by architecture.** There is no server. Every request goes straight from your browser to the GitHub API and to your LLM provider. Your credentials never leave your device, and settings travel between devices as an AES-256-GCM encrypted blob.

**1** static HTML file · **0** servers & databases · **0** setup on your repo · **100%** GitHub-native data

Everything Issue Composer creates is a plain GitHub issue, label or comment. Your teammates see the same tracker on github.com. Close the tab or delete the app whenever you want — nothing was ever held hostage.

---

## Quick start (60 seconds)

1. **Open the app** — https://jalopezsuarez.github.io/issue-composer/ (or add it to your home screen as a web app).
2. **Connect GitHub** — in *Settings*, paste a Personal Access Token (the app links you to [github.com/settings/personal-access-tokens](https://github.com/settings/personal-access-tokens/new) and lists the exact permissions: fine-grained with *Issues read & write* + *Metadata read*, or classic with the `repo` scope) and press **Connect GitHub**.
3. **Configure your LLM** — any OpenAI-compatible API: Base URL, API key and model (e.g. `https://api.openai.com/v1` + `gpt-4o-mini`).
4. **Ship your first issue** — press **+**, write a note, tap a mode (Simple / Feature / Bug) and the AI writes it; review and **Publish**. It lands at the top of your *pending* column.

---

## Contents

- [Views](#views)
- [Writing modes](#writing-modes)
- [Search](#search)
- [Gestures and interaction](#gestures-and-interaction)
- [Architecture](#architecture)
  - [Source-code map](#source-code-map)
  - [Data model: GitHub as the database](#data-model-github-as-the-database)
  - [Kanban ordering algorithm](#kanban-ordering-algorithm)
  - [Issues list behavior](#issues-list-behavior)
  - [GitHub integration](#github-integration)
  - [LLM integration](#llm-integration)
  - [AI assistant (tool-calling)](#ai-assistant-tool-calling)
  - [Theming and typography](#theming-and-typography)
  - [State and persistence](#state-and-persistence)
  - [Encrypted settings export/import](#encrypted-settings-exportimport)
  - [UI conventions](#ui-conventions)
- [Privacy and security](#privacy-and-security)
- [Deployment](#deployment)
- [Known limitations](#known-limitations)
- [Project structure](#project-structure)

---

## Views

Navigation is a **floating glass capsule** (bottom right) with four destinations — **Kanban**, **Issues**, **AI assistant** and **Settings** — plus a **+** button to create and a 🔍 button to search. The floating repo label at the top acts as a shortcut: tapping it triggers the left button's action (back or reload).

### 🗂️ Kanban
Board by **status**, one column per fixed status. Each card shows the title, a 2-line excerpt, the number, comments and the last-updated date. It allows:
- **Dragging between columns** to change the issue's status.
- **Reordering within a column** by dragging vertically (persistent manual order).
- Tapping a card to open its **detail**.

Issues published from the app land at the **top of the *pending* column** (created with `status:pending` + `order:0` in the same request). The created issue is inserted **optimistically** into the local board and list caches straight from the POST response — GitHub's listing can lag a few seconds behind a just-created issue, so no refetch is needed for it to appear instantly.

The board loads **page by page** (25 issues per page, the same standard size as the list): scrolling near the bottom fetches the next page for **all columns at once** (the vertical scroll is shared), and while it loads each column shows a **transparent pagination row with a centered loader** at its bottom. If the loaded pages don't fill the screen, the next page is requested automatically until it does (or there's nothing left). Pagination works the same while searching (Search API pages).

### 📋 Issues
List with **infinite scroll** (25 per page). Issues render as **two groups — open first, closed always at the end** — each ordered by date. Each row shows the `open/closed` capsule + the status chip, then `#no · user · comments · date`. The **sort** toggle (Newest / Oldest) lives inside the search bar.

### ✏️ Create (+ button)
1. Write a **note** describing what you want.
2. Tap a writing **mode** (Simple / Feature / Bug) — tapping **generates with AI** directly; if the mode requires it, the AI reviews the codebase first.
3. Review/edit the generated title + body and **Publish**. The composer closes, you return to the board/list, and a **toast** under the nav bar confirms the new issue (tap it or wait to dismiss). Progress and validation messages appear below the Publish button.

### 🔍 Issue detail
- Description rendered in **Markdown** (headings, lists, checklists, code, quotes, links…), via a dependency-free, escape-first renderer.
- The user's **labels** (internal `status:`/`order:` labels stay hidden).
- **Status**: a row of options to change it instantly (single labels call).
- **Edit details** (opens the full-screen editor for the body) and a solid **Close / Reopen** button, stacked full-width.
- A **share fab** (top right, box-with-arrow icon) opens the issue on github.com.
- **Comments**: listed, created with AI (tap a mode to generate), and your own can be edited/deleted.
- Tapping the **title** opens the editor to rename it.

### ✨ AI assistant
A **ChatGPT-style** conversational view scoped to the **active repository's issues and code**. Using your configured LLM with **function/tool calling**, it can read and search issues, read source files, create issues, add/edit comments, change statuses and edit titles/descriptions. Each tool call shows as a small chip while it runs. Conversations are **not persisted**. A round **(X)** closes it and returns to the previous view.

### ⚙️ Settings
- **Active repo banner** — a capsule pinned at the top: **yellow** for public repos, **green with a lock** for private ones (privacy detected on selection and cached).
- **Repositories** — a **Recent / Favorites** segmented list (max 25 each, app-local). Recents are ordered **most-recently-used first**: activating a repo from anywhere (the select, a recent, a favorite) moves it to the top. Tap a row to select (the active repo shows a green/yellow dot — private/public), ✕ removes, private repos show a lock; the **star fab** (top right) toggles the active repo as a favorite.
- **100% local, 100% yours** — an informational section (title + text) above GitHub making the local-first design explicit: no backend, everything stored in the browser, requests go only to GitHub and the LLM provider.
- **GitHub** — token (with inline guidance and a direct link to create one), repository search + select, and the **Connect GitHub** button below them.
- **LLM provider** — Base URL, API key, Model (OpenAI-compatible), plus a **Test LLM** button that makes one tiny round-trip and reports the result (with latency) in the section's message zone.
- **Generation language** — Spanish / English (for AI output; the UI is English).
- **Appearance** — **Light / Dark / Auto** theme and a **font size** control (− / offset / +) that scales the whole app.
- **Export &amp; import settings** — take your setup to another device or keep a safe copy, everything encrypted (see [below](#encrypted-settings-exportimport)).

### 🖊️ Full-screen editor
A reusable, borderless full-screen `textarea` to edit **one element at a time**: the issue title, its body or a comment. Back (cancel) top left, save (↑) top right. It tracks the visible viewport so it fits exactly above the keyboard.

---

## Writing modes

Available when creating issues and when writing comments — **tapping the mode is the generate action**:

| Mode | Context used | What it produces |
|------|:---:|-------------|
| **Simple** | Project context only (README) | Cleans, structures and polishes your text so it reads well, adopting the project's own terminology. Never references code (no paths, symbols or snippets) and never invents details. Keeps your meaning and intent. |
| **Feature** | README + code review | A **new feature** as a User Story / PBI: *User story, Description and context, Status in the code, Scope, Areas/files involved, Acceptance criteria, Technical notes and dependencies*. |
| **Bug** | README + code review | A **bug report**: *Description, Reproduction steps, Expected vs actual behavior, Status in the code, Files involved, Probable root cause, Impact, Edge cases, Proposed fix, Acceptance criteria*. |

In **Feature** and **Bug**, the AI first analyzes the code to assess whether what you describe is *already implemented, partial or pending*, and reflects it in a **Status in the code** section citing **real** paths and symbols. Comments also take the issue description and the previous thread into account.

---

## Search

Both the **Kanban** and the **Issues** list have a 🔍 fab that toggles a pill search bar above the board/list. It queries the **GitHub Search API** server-side; ✕ clears it. The bar's open/closed state is remembered per view. Empty, loading and error states are **uniform** across both views.

---

## Gestures and interaction

- **Tap card / row** → opens the detail. **Tap the floating repo label** → triggers the left button (back/reload).
- **Drag card** (Kanban): mouse starts on move; touch uses a **long press** (~180 ms) to pick up. Auto-scroll near the edges. Horizontal drag changes column; vertical drag reorders.
- **+** → issue composer · **🔍** → search bar · **Reload** (left fab) → refresh from network.
- Zoom is disabled (pinch and double-tap) to behave like a native app.

---

## Architecture

> This section is written for both humans and AI agents that need a precise conceptual model of the system.

**One file, no build.** The entire application is `index.html`: a `<style>` block, the markup for every view, and one IIFE `<script>` (`(function () { "use strict"; ... })()`) in ES5-style vanilla JavaScript. There are no dependencies, no framework, no build step and no module system. A tiny inline `<head>` script applies the persisted theme/font-size before first paint.

**Rendering model.** Views are `<section class="view">` elements toggled with an `.active` class by `showView(name)` / `markActiveView(name)` (`VIEWS = ["kanban", "issues", "composer", "config", "edit", "assistant"]`). Dynamic content (lists, board, detail, chat) is rendered by building HTML strings from escaped data (`esc()`) and assigning `innerHTML`, or via `document.createElement` for rows/cards. The **editor** and the **assistant** are fixed, full-viewport overlays; the others live inside the scrollable `<main id="content">`.

**Responsive width.** The scroll containers span the **full viewport** (so scrollbars sit at the screen edge). Content width is capped per view through the `--appmax` CSS variable set by `markActiveView`: `100%` for kanban/issues, `720px` (centered) for create/detail/settings; the fixed overlays center their ~720px column with side padding.

**Concurrency.** In-flight request races (fast typing in search, switching repos) are handled with **monotonic tokens**: `issuesState.token` and `KAN.token` are bumped on each new load; stale responses compare their captured token and bail.

### Source-code map

`index.html` is organized top-to-bottom:

1. `<head>` — meta/viewport/PWA tags, favicon, **no-flash theme + font script**, the full `<style>` sheet (CSS variables → dark overrides → fabs/nav → groups/fields → tables/banners/search → lists → detail/markdown → status rows → kanban → assistant chat → toast → export rows).
2. `<body>` — fixed chrome (fabs: back, reload, repo tag, add, search, favorite star, save, assistant close, share; toast), `<main id="content">` with the kanban / issues(+detail) / composer / settings sections, the nav capsule, and the fixed editor + assistant sections.
3. `<script>` IIFE, in order: octicon `PATHS` + `svg()` helper · constants (`GH_API`, `LS_KEYS`, `ARRAY_KEYS`, tones, theme/font state) · DOM helpers (`$`, `showMsg`, `showBusy`, **toast**, `esc`, `fmtShort`) · view switching (`markActiveView`, `showView`) · settings load/persist · segmented controls (language, theme, font size, io mode, tones) · prompts (`issueSystemPrompt`, `commentSystemPrompt`, `GROUNDING`, `CODE_ASSESS`) · repo banners/tag/star fab · recents & favorites · encrypted export/import · repo selection + privacy cache · `ghFetch` / `parseLooseJSON` / `llmComplete` · repos connect · issues list (state, rows, search, sort, infinite scroll) · markdown renderer (`renderMd`) · detail + comments (+ AI comment generation) · full-screen editor · statuses/labels machinery · kanban (load, render, drag) · composer (code review pipeline + generate + publish) · **AI assistant** (tool schemas, executors, chat loop, UI) · viewport sync · init (restore settings, controls, tab).

### Data model: GitHub as the database

There is **no app database**. All durable state is either GitHub itself or `localStorage`:

- **Issues, comments, titles, bodies, open/closed state** — plain GitHub objects, manipulated through the REST API.
- **Kanban status** — a label `status:<key>` with fixed keys: `pending · progress · review · done · archive · resources`. An issue with no status label is *displayed* as `pending` (pure in-memory default; nothing is written until the user acts).
- **Manual order within a column** — a label `order:<n>`. Both label families are "internal": hidden from the label chips the UI shows.
- Each status has a **theme-aware palette**: light pastel background + dark dot for light mode, and a muted Notion-style dark set (`dbg`/`dtext`/`dcol`) for dark mode, resolved at render time by `statusColors()`.

### Kanban ordering algorithm

Within each column (`renderKanban` sort):
1. **Closed issues always sink to the bottom.**
2. Issues **with** `order:n` come first, ascending.
3. Issues **without** order follow, newest first.

New issues are created with `order:0` (manual orders start at 1), so they appear at the very top of *pending*; ties at 0 resolve newest-first. Dragging calls `applyColumnOrder(repo, status, seq)`, which renumbers the destination column `1..N` and writes **only the cards that changed** (status + order in one `PUT` per card). Changing status from the detail (`applyStatus`) writes the new status and discards the old order. The label writer (`putIssueLabels`) replaces the full label set in one call and, on a `422` (labels missing in the repo), creates them and retries once.

### Issues list behavior

- Loaded 25/page (`per_page`), sorted by `updated` in the chosen direction; **pull requests are filtered out**.
- Rendering always paints **two stable groups**: open first, closed last (a stable sort by state preserves the date order inside each group, even across pages).
- With a query, the list switches to the **Search API** (`GET /search/issues`, `q=repo:{repo} type:issue {query}`) which returns `{items}` instead of an array.
- First page shows a centered loader; later pages show the **standard pagination row** below the list. No-token / no-repo / empty / error states render as uniform centered blocks shared with the kanban.

### GitHub integration

REST v3 (`https://api.github.com`), all requests through `ghFetch` with:

```
Authorization: Bearer <token>
Accept: application/vnd.github+json
X-GitHub-Api-Version: 2022-11-28
```

Reads use `cache: "no-store"` (GitHub serves `max-age=60`, which otherwise made label changes look reverted). Endpoints used: `GET /user`, `GET /user/repos`, `GET /repos/{repo}`, `GET/POST /repos/{repo}/issues` (creation includes `labels`), `GET /search/issues`, `PATCH .../issues/{n}`, `PUT .../issues/{n}/labels`, `POST /repos/{repo}/labels`, `GET/POST/PATCH/DELETE .../comments`, `GET .../readme`, `GET .../git/trees/{branch}?recursive=1`, `GET .../contents/{path}`.

### LLM integration

Any **OpenAI-compatible** `POST {baseURL}/chat/completions` with `Authorization: Bearer`. Two thin client functions sit on one shared transport, `llmPost(baseUrl, apiKey, body)`:

- `llmComplete(messages)` — plain text/JSON completions for issue & comment writing (`max_tokens: 1500`, `temperature: 0.3`).
- `llmChatRaw(messages, tools)` — the assistant's calls, with `tools` + `tool_choice: "auto"` (`temperature: 0.2`).

`llmPost` provides two compatibility layers:

- **Headers** (`llmHeaders`): when the provider is **Anthropic (Claude)** — base URL on `anthropic.com` or an `sk-ant-` key — it adds `anthropic-dangerous-direct-browser-access: true`, which Anthropic requires before it will serve CORS headers to a browser (without it every call fails as "Failed to fetch" even with valid credentials). Anthropic's OpenAI-compatible endpoint (`https://api.anthropic.com/v1`) then works directly, including assistant tool-calling.
- **Adaptive parameters**: optional parameters vary across models — some reject `temperature`, others demand `max_completion_tokens` instead of `max_tokens`. On a `400` whose message names one of them, `llmPost` drops it (`temperature`, `tool_choice`) or renames it (`max_tokens` → `max_completion_tokens`) and retries; each retry removes a distinct parameter so chains terminate, and genuine 400s surface unchanged.

The Settings **Test LLM** button makes one tiny round-trip through this same path and reports the result (with latency) in the section's message zone.

**Code-review pipeline** (Feature/Bug modes), in `reviewCodebase(owner, name, query)`:
1. `getRepoBase` (cached per repo/session): default branch → README (first 5000 chars) → full recursive file tree, filtered by `IGNORE_DIRS` (node_modules, dist, …) and `BINARY_EXT`.
2. One LLM call picks **up to 5 relevant paths** from the tree (strict JSON array output, validated against the real tree).
3. Those files are downloaded (≤5000 chars each) and passed as grounded context.

**Prompt design:** system prompts per mode enforce structure and grounding — `GROUNDING` (cite only real paths/symbols; omit empty sections) and `CODE_ASSESS` (assess implemented/partial/pending before writing, reflected in a *Status in the code* section). **Simple** mode uses `projectContext()` (README only) and explicitly forbids code references. Issue writing must return strict JSON `{"title","body"}` (tolerant parser strips fences); comment writing returns raw Markdown. A language instruction (English/Spanish) is appended from settings.

### AI assistant (tool-calling)

- The conversation is a standard OpenAI message list; the system prompt scopes the assistant to the **active repo's issues and code**, lists the valid statuses and the reply language, and instructs it to refuse unrelated topics.
- `ASSISTANT_TOOLS` (JSON-schema function declarations) ↔ `ASSISTANT_IMPL` executors, all resolving against the active repo:

| Tool | Action |
|------|--------|
| `list_issues` | List issues (state filter), PRs excluded, brief shape |
| `search_issues` | Search API query |
| `get_issue` | Full issue + comments |
| `create_issue` | Create (title, body) |
| `update_issue` | Patch title / body / open-closed state |
| `set_issue_status` | Set the `status:` label (kanban machinery) |
| `add_comment` / `update_comment` | Create / edit a comment |
| `list_files` / `read_file` | Repo tree (cached) / file contents (truncated) |

- The loop (`runAssistantLoop`): call the LLM → if `tool_calls`, render label chips, execute all calls (errors return `{error}` payloads and tint the chip), append `role:"tool"` results, and iterate — up to **8 steps** — until a plain text answer arrives (rendered as Markdown). Mutating tools invalidate the list/board caches so they reload next visit.

### Theming and typography

- **Themes:** `light`, `dark`, `auto`. Dark is a CSS-variable override under `:root[data-theme="dark"]` (GitHub-dark palette) plus a few component tweaks. **Auto** resolves against `prefers-color-scheme` and re-applies live when the OS changes. The resolved theme also drives the status palettes and the `theme-color`/`color-scheme` meta tags. A head script applies the stored theme **before first paint**.
- **Typography:** everything is sized in `rem` over a single root rule — `html { font-size: calc(19px + var(--fofs, 0px)) }`. The Settings font control moves `--fofs` between −4 and +6 px; one variable scales the entire app uniformly.

### State and persistence

`localStorage`, via the `LS_KEYS` map (these keys are what export/import covers):

| Key | Content |
|-------|-----------|
| `ic_gh_token` | GitHub Personal Access Token |
| `ic_llm_base_url` / `ic_llm_api_key` / `ic_llm_model` | LLM configuration |
| `ic_repo` | Active repository |
| `ic_recent` / `ic_favs` | Recent / favorite repositories (JSON arrays, max 25, self-healing; recents MRU-first) |
| `ic_tone` | Writing mode |
| `ic_lang` | Generation language |
| `ic_theme` | UI theme (light/dark/auto) |
| `ic_font` | Font-size offset over the base |
| `ic_tab` | Last used tab |
| `ic_repo_tab` | Recent/Favorites segment |

Internal helpers outside export/import: `ic_repo_priv` (per-repo public/private cache) and `ic_srch_kanban` / `ic_srch_issues` (search-bar open state).

### Encrypted settings export/import

Web Crypto, **AES-256-GCM**:
- **Export my settings** generates a random 256-bit key, encrypts `JSON.stringify(settings)` with a random 12-byte IV, and outputs two base64 strings: the **payload** (`IV ‖ ciphertext`) and the **key** (raw 32 bytes) — shown in two fields with copy buttons, the action button below them.
- **Import my settings** takes both (each field has a paste-from-clipboard button), decrypts, validates and writes only known `LS_KEYS`, then reloads. A wrong key fails with a clear message; nothing plaintext ever leaves the field. All progress/results report in the section's message zone (no JS alerts).

### UI conventions

- **Borderless design:** no divider or container borders anywhere; sections are separated only by their uppercase title (with top margin). The *only* lines are the per-row `border-bottom` inside genuine lists — issues, comments, recents/favorites and the status selector.
- **Capsules:** the settings repo banner, search inputs, chips and the toast are fully-rounded; the fabs and the nav are frosted glass (`backdrop-filter`).
- **Toast:** a single fixed capsule below the top bar (`showToast(html, ms)`), auto-hides in ~7 s, tap to dismiss — used for "issue created".
- **Dates:** one format everywhere (`fmtShort`): `04 jul 11:38`, adding the year only when it differs from the current one.
- **Empty/loading/error states:** one shared centered block (`stateBlock`) with identical wording across the board and the list.
- **Infinite-scroll pagination:** one standard everywhere — a transparent `.pageloader` row with a centered spinner, built by `pageLoader()`, shown below the issues list and at the bottom of every kanban column while the next page loads; one shared scroll trigger on the content scroller paginates whichever view is active.
- **Icons:** inline monochrome octicons from a `PATHS` map rendered by `svg(name, size)` (16×16 viewBox, `currentColor`).

### Section message zones (feedback standard)

This is the **fixed standard** for user feedback on anything that talks to a third-party service (GitHub API, LLM endpoint, clipboard, Web Crypto). Every new feature MUST follow it.

**Placement.** Each section that interacts with an external service has one reserved message zone — a `<div class="msg">` — placed **directly below the section's title + description** and above its first field. It is invisible (`display:none`) until a message is shown.

**Severities.** Exactly three states, each with a fixed meaning and color (light/dark variables in `:root`):

| State | Class | Color | Used for |
|---|---|---|---|
| Info / OK | `.msg.info` / `.msg.success` | **blue** (`--info-bg`/`--info-fg`) | progress (with `<span class="spinner">`), confirmations, successful results |
| Warning | `.msg.warn` | **yellow** (`--warn-bg`/`--warn-fg`) | missing input / user action needed before the call (guards), non-fatal issues (clipboard unavailable, 0 results) |
| Error | `.msg.error` | **red** (`--danger-bg`/`--danger`) | failed API calls, crypto failures, network errors |

**Helpers** (always use these, never set classes by hand): `msgInfo(el, text)`, `msgWork(el, text)` (info + spinner, for in-flight calls), `msgWarn(el, text)`, `msgError(el, text)`, `hideMsg(el)`.

**Zones.** `repoMsg` (Settings › GitHub), `llmMsg` (Settings › LLM provider), `ioMsg` (Settings › export/import), `genMsg` (Create › note & generation), `publishMsg` (Create › review and publish), `statusMsg` (Detail › status + close/reopen), `commentsMsg` (Detail › comments list), `commentMsg` (Detail › new comment). Board/list-wide errors use the view-level `kanbanMsg` / `issuesMsg` banners.

**Rules.**
- Be maximally informative with **short, simple, descriptive** messages: show a spinner message while a call is in flight, a blue confirmation when it succeeds, and a mapped, actionable red message when it fails (`ghErrorText` translates 401/403/404/network into plain language).
- Guard checks (missing token, repo, note, LLM config…) are **yellow**, not red — nothing failed yet.
- GitHub errors go through `ghErrorText(err)`; LLM errors surface the provider message.
- The **only exception**: successful issue creation is confirmed with the **toast** under the nav bar (the composer closes, so there is no section left to report into).
- Debounced "saved on this device" notes (token / LLM fields) must never stomp an action's progress or result: any explicit `showMsg`/`hideMsg` on the zone cancels the pending note (`cancelSavedNote`).
- No `alert()` for API outcomes in sections that have a zone; the only remaining alerts are in the full-screen editor, which has no section header.

---

## Privacy and security

- The **GitHub token** and **LLM credentials** live only in the browser's `localStorage` (by design, at the user's request): they stay on the device and are accessible to same-origin scripts only.
- Use a **minimal-permissions token** on a trusted device. Private-repo status is detected and shown (green banner + lock).
- **No backend, no telemetry, no middleman**: browser → GitHub, browser → LLM. Settings transfer is end-to-end encrypted (AES-256-GCM) with a key that's displayed only to you.

---

## Deployment

A **static site**. GitHub Pages publishes the repo root on every push to `main` via [`.github/workflows/static.yml`](.github/workflows/static.yml); the landing page under [`web/`](web/) ships alongside.

To self-host: copy `index.html` (optionally `web/index.html` and the workflow), enable Pages → GitHub Actions, push to `main`. It also works opened directly from disk (encryption features require a secure context, i.e. HTTPS or localhost).

---

## Known limitations

- **LLM CORS**: your provider/proxy must allow the app's origin.
- The **assistant requires tool/function-calling support** in the provider.
- No real-time sync — use **Reload** to pick up outside changes.
- Search relies on the GitHub Search API (rate-limited, slightly delayed vs the plain listing).

---

## Project structure

```
.
├── index.html                     # The whole app (HTML + CSS + JS, no build)
├── web/
│   └── index.html                 # Marketing landing page (self-contained)
├── favicon.png                    # App icons
├── apple-touch-icon.png
├── README.md
└── .github/
    └── workflows/
        └── static.yml             # Deployment to GitHub Pages
```
