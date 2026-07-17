# Portfolio — Technical Overview

My personal portfolio site, built with React and delivered as a single plain `index.html` with no build tooling required.

---

## Architecture

Everything lives in one file: `index.html`. It is structured in three layers:

1. **HTML shell** — sets the page title (`Portfolio, C.J. Jones`), loads a favicon (`supporting-images/fav.png`), pulls in external fonts, and mounts the React root
2. **CSS** — a `<style>` block using custom properties for the entire design system
3. **React app** — a `<script type="text/babel">` block containing all data, components, and the app entry point

React and Babel are loaded via CDN, so the file runs directly in any modern browser without a build step. The site also references external supporting assets (images and video under `supporting-images/`) and links out to standalone project pages under `case-studies/`, `speculative-designs/`, and `strategic-items/`.

---

## Dependencies (all via CDN)

| Package | Version | Source | Purpose |
|---|---|---|---|
| React | 18 | unpkg | UI component model |
| ReactDOM | 18 | unpkg | DOM rendering |
| Babel Standalone | latest | unpkg | JSX transpilation in-browser |
| Google Fonts | — | fonts.googleapis.com | DM Serif Display, DM Mono, DM Sans |

React is loaded from the development build (`react.development.js` / `react-dom.development.js`).

---

## Design System

All design tokens are defined as CSS custom properties on `:root`:

**Colors**
- `--bg`, `--bg2`, `--bg3` — layered dark backgrounds
- `--surface` — elevated surface color
- `--border`, `--border2` — subtle and mid-weight borders
- `--text`, `--text2`, `--text3` — three levels of text hierarchy
- `--accent` — lime green (math / logic energy), used for primary highlights and active states
- `--accent2` — sky blue (CS / systematic), secondary accent
- `--accent3` — amber (warmth / craft), tertiary accent

**Typography**
- `--serif` — DM Serif Display, used for display headings and card titles
- `--mono` — DM Mono, used for labels, tags, metadata, and UI chrome
- `--sans` — DM Sans, used for body text and navigation

**Layout**
- `--nav-w: 220px` — fixed sidebar width
- `--max-w: 860px` — content column max width

---

## Layout

The page uses a two-column fixed layout:

```
┌─────────────────┬──────────────────────────────────┐
│  Sidebar (fixed)│  Main content (scrollable)        │
│  220px          │  margin-left: 220px               │
│                 │  max-width: 860px + padding       │
│  · Brand        │                                   │
│  · Nav links    │  Sections stack vertically        │
│  · Footer links │                                   │
└─────────────────┴──────────────────────────────────┘
```

On viewports under 900px the sidebar is hidden (`--nav-w` collapses to `0px`) and the main column fills full width, with several grids collapsing to fewer columns.

---

## Navigation

The sidebar nav uses the browser's `IntersectionObserver` API to track which section is currently in view and applies an `active` class to the corresponding nav link (with a `rootMargin` of `-30% 0px -60% 0px`). Clicking a nav link calls a `scrollTo` helper that runs `scrollIntoView({ behavior: 'smooth', block: 'start' })` on the target section element.

Each section is identified by a plain HTML `id` attribute that matches an entry in the `NAV` array. The current `NAV` entries are:

| id | Label | Index |
|---|---|---|
| `home` | Home | 00 |
| `work` | Featured Work | 01 |
| `about` | About Me | 02 |
| `case-studies` | Case Studies | 03 |
| `strategy` | Product Strategy | 04 |
| `experiments` | Concept Plans & Testing | 05 |
| `history` | Projects & Accomplishments | 06 |
| `contact` | Contact | 07 |

---

## Component Structure

All components are plain React functional components defined in a single script block. The sidebar is written as inline JSX inside `App` rather than as its own component.

```
App
├── nav.sidebar (inline JSX)
│   ├── Brand ("C.J. Jones" + "CS · UX · MTH · ACS")
│   ├── Nav list (maps over the NAV array)
│   └── Footer links (Email, LinkedIn, "Issues?" form)
└── main
    ├── HomeSection
    ├── WorkSection          (auto-rotating carousel)
    ├── AboutSection         (includes the Skills clusters)
    ├── CaseStudiesSection
    ├── StrategySection
    ├── ExperimentsSection
    ├── HistorySection
    └── ContactSection
```

`SectionHeader` is a shared sub-component used at the top of most sections. It renders the section label, a horizontal rule, and the section index number. `scrollTo(id)` is a small utility used by nav links and hero buttons.

**WorkSection** is an auto-rotating carousel: it cycles through `PROJECTS` on a 15-second interval (remounting the active card to replay a fade animation) and exposes a row of thumbnails for manual selection.

**CaseStudiesSection** cards can render either a `video` (e.g. `.mov`) or an `image`, depending on which field is present on each entry.

---

## Data Layer

Content is separated from markup via plain JavaScript arrays defined at the top of the script block. Each section that renders a list of items has a corresponding array:

| Array | Used by |
|---|---|
| `NAV` | Sidebar nav |
| `PROJECTS` | WorkSection (carousel) |
| `CASE_STUDIES` | CaseStudiesSection |
| `STRATEGY_ITEMS` | StrategySection |
| `EXPERIMENTS` | ExperimentsSection |
| `HISTORY` | HistorySection |

The Skills clusters (in `AboutSection`) and the contact links (in `ContactSection`) are hardcoded inline in their respective components rather than driven by arrays.

Most item objects carry fields such as `title`, `desc`/`summary`, `tags`, an `image` or `video` path, and an `external` flag with a `site` path pointing to a standalone project page.

---

## Animations

Section entry uses a CSS `@keyframes fadeUp` animation (opacity + translateY) applied to every `.section` element. The WorkSection carousel uses a separate `@keyframes fade-in` animation when the active slide changes. No JavaScript animation library is used.

Hover transitions on cards, nav links, and buttons use CSS `transition` on `background`, `color`, `border-color`, `transform`, and `gap` properties.

---

## Extending the Site

**Adding a section**
1. Add an entry to the `NAV` array
2. Create a new functional component (e.g. `function WritingSection() { ... }`)
3. Render it inside `<main>` in the `App` component
4. Give the root `<section>` the matching `id`
5. If it renders a list, add a corresponding data array at the top of the script block

**Adding a build pipeline**
The component structure maps cleanly to a Vite + React + TypeScript setup. Each section component can be extracted into its own `.tsx` file, the data arrays moved to a `data/` directory, and CSS custom properties migrated to a global stylesheet or CSS Modules.

**Deploying**
No build step means deployment is just serving the file (plus its `supporting-images/` and linked project directories). It works with GitHub Pages, Netlify drag-and-drop, Vercel, or any static host.
