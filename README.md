# Portfolio — Technical Overview

A single-file personal portfolio site built with React, delivered as a plain `index.html` with no build tooling required.

---

## Architecture

Everything lives in one file: `index.html`. It is structured in three layers:

1. **HTML shell** — sets the page title, loads external fonts, and mounts the React root
2. **CSS** — a `<style>` block using custom properties for the entire design system
3. **React app** — a `<script type="text/babel">` block containing all components and data

React and Babel are loaded via CDN, so the file runs directly in any modern browser without a build step.

---

## Dependencies (all via CDN)

| Package | Version | Purpose |
|---|---|---|
| React | 18 | UI component model |
| ReactDOM | 18 | DOM rendering |
| Babel Standalone | latest | JSX transpilation in-browser |
| Google Fonts | — | DM Serif Display, DM Mono, DM Sans |

---

## Design System

All design tokens are defined as CSS custom properties on `:root`:

**Colors**
- `--bg`, `--bg2`, `--bg3` — layered dark backgrounds
- `--surface` — elevated surface color
- `--border`, `--border2` — subtle and mid-weight borders
- `--text`, `--text2`, `--text3` — three levels of text hierarchy
- `--accent` — lime green, used for primary highlights and active states
- `--accent2` — sky blue, secondary accent
- `--accent3` — amber, tertiary accent

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
│  · Nav links    │  Sections stack vertically,       │
│  · Social links │  separated by border-bottom       │
└─────────────────┴──────────────────────────────────┘
```

On viewports under 900px the sidebar is hidden and the main column fills full width.

---

## Navigation

The sidebar nav uses the browser's `IntersectionObserver` API to track which section is currently in view and applies an `active` class to the corresponding nav link. Clicking a nav link calls `scrollIntoView({ behavior: 'smooth' })` on the target section element.

Each section is identified by a plain HTML `id` attribute that matches an entry in the `NAV` array.

---

## Component Structure

All components are plain React functional components defined in a single script block.

```
App
├── Sidebar
│   ├── Brand
│   ├── NavList (maps over NAV array)
│   └── SocialLinks
└── Main
    ├── HomeSection
    ├── WorkSection
    ├── CaseStudiesSection
    ├── ExperimentsSection
    ├── AboutSection
    ├── ResumeSection
    └── ContactSection
```

`SectionHeader` is a shared sub-component used at the top of every section. It renders the section label, a horizontal rule, and the section index number.

---

## Data Layer

Content is separated from markup via plain JavaScript arrays defined at the top of the script block. Each section that renders a list of items has a corresponding array:

| Array | Used by |
|---|---|
| `NAV` | Sidebar nav |
| `PROJECTS` | WorkSection |
| `CASE_STUDIES` | CaseStudiesSection |
| `EXPERIMENTS` | ExperimentsSection |
| `RESUME_EXPERIENCE` | ResumeSection |
| `RESUME_EDUCATION` | ResumeSection |

Skills clusters and contact links are hardcoded inline in their respective components.

---

## Animations

Section entry uses a CSS `@keyframes fadeUp` animation (opacity + translateY) applied to every `.section` element. No JavaScript animation library is used.

Hover transitions on cards, nav links, and buttons use CSS `transition` on `background`, `color`, `border-color`, and `transform` properties.

---

## Extending the Site

**Adding a section**
1. Add an entry to the `NAV` array
2. Create a new functional component (e.g. `function WritingSection() { ... }`)
3. Render it inside `<main>` in the `App` component
4. Give the root `<section>` the matching `id`

**Adding a build pipeline**
The component structure maps cleanly to a Vite + React + TypeScript setup. Each section component can be extracted into its own `.tsx` file, the data arrays moved to a `data/` directory, and CSS custom properties migrated to a global stylesheet or CSS Modules.

**Deploying**
No build step means deployment is just serving the file. It works with GitHub Pages, Netlify drag-and-drop, Vercel, or any static host.
