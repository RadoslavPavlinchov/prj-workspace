# prj-workspace

A small, production‑quality React app that showcases frontend patterns: **TanStack Router**, **TanStack Query** for server state, **Redux Toolkit** for UI state, **MSW** for local API mocking, **Vitest** for tests & coverage, **Storybook** for a component catalog, **Tailwind CSS** for styling, and **Azure Static Web Apps** for deployment.

---

## Badges

[![CI](https://github.com/RadoslavPavlinchov/prj-workspace/actions/workflows/ci.yml/badge.svg)](https://github.com/RadoslavPavlinchov/prj-workspace/actions/workflows/ci.yml)
[![Deploy SWA](https://github.com/RadoslavPavlinchov/prj-workspace/actions/workflows/azure-swa.yml/badge.svg)](https://github.com/RadoslavPavlinchov/prj-workspace/actions/workflows/azure-swa.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](#license)

---

## Demo

- **Production** (Azure Static Web Apps): `https://<your-swa-domain>.azurestaticapps.net`
- **Storybook** (optional SWA or GitHub Pages): `https://<your-swa-domain>.azurestaticapps.net/storybook/`

---

## Table of Contents

- [Motivation & Goals](#motivation--goals)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Architecture Decisions](#architecture-decisions)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment & Configuration](#environment--configuration)
- [Available Scripts](#available-scripts)
- [Testing & Coverage](#testing--coverage)
- [Storybook](#storybook)
- [Accessibility & i18n](#accessibility--i18n)
- [Performance](#performance)
- [CI/CD](#cicd)
- [Security & Repo Hygiene](#security--repo-hygiene)
- [Data Sources (Dev vs Prod)](#data-sources-dev-vs-prod)
- [Troubleshooting](#troubleshooting)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Motivation & Goals

**prj-workspace** is a realistic, public example that mirrors patterns you’ll use on an enterprise project:

- Go through the full cycle: **scaffold → build features → test → document → deploy**.
- Demonstrate the separation of concerns between **server state** (Query) and **UI/client state** (Redux).
- Show production touches: **routing boundaries, error states, a11y**, coverage thresholds, CI, and Azure deployment.

---

## Key Features

- **People directory** with list, detail, filter/search.
- **TanStack Router** with lazy loading, not‑found & error boundaries.
- **TanStack Query** with loading, error, empty, and optimistic updates.
- **Redux Toolkit** for global UI state (filters, theme, locale).
- **MSW** for dev/test API; **static JSON** in production.
- **Vitest + Testing Library** & **coverage thresholds**.
- **Storybook** with a11y checks and interaction tests.
- **Azure Static Web Apps** deployment + PR preview environments.

---

## Tech Stack

- **Runtime:** React 18/19 + TypeScript, Vite
- **Routing:** @tanstack/react-router
- **Server state:** @tanstack/react-query
- **UI state:** @reduxjs/toolkit, react-redux
- **Mocking:** msw
- **Styling:** Tailwind CSS
- **Forms/Validation:** react-hook-form, zod
- **Testing:** Vitest, @testing-library/react, @testing-library/user-event, jsdom, @vitest/coverage-v8
- **Docs:** Storybook (Vite builder)
- **CI/CD:** GitHub Actions → Azure Static Web Apps

---

## Architecture Decisions

### 1) Server State vs UI State

- **React Query** owns **server cache**, request lifecycles, retries, and deduping.
- **Redux Toolkit** owns **UI state** and preferences (search query, filters, theme, locale). This avoids overusing Redux for data that Query already solves.

### 2) Mocking Strategy

- **Dev/Test:** MSW intercepts `/api/*`, returning deterministic seed data.
- **Production:** fetch from a static file: `public/data/users.json`.

```ts
// src/config.ts
export const USERS_URL = import.meta.env.DEV ? '/api/users' : '/data/users.json'
```

### 3) Routing

- File‑like route organization with **TanStack Router**.
- Per‑route **error** and **pending** boundaries.
- **Lazy** route components + **prefetch** for details on hover/focus.

### 4) Forms & Mutations

- **react-hook-form + zod** for UX and safety.
- **Optimistic updates** for create/update (dev/test via MSW; in prod we no‑op or demonstrate the pattern without persisting).

### 5) Testing

- **Unit** (slices, utils) + **component** (pure UI) + **integration** (Query + MSW) with **coverage thresholds** in CI.

### 6) Storybook

- Living component catalog with **A11y addon** and **interaction tests**.

### 7) Deployment

- **Static Web App**: SPA fallback to `/index.html`, strict CSP, and cache headers via `staticwebapp.config.json`.

#### Diagram

```mermaid
flowchart LR
  UI[React + Tailwind Components] --> Router[TanStack Router]
  UI --> Store[Redux Toolkit (UI state)]
  UI --> Query[React Query (server state)]
  Query -->|DEV/TEST| MSW[(MSW Handlers)]
  Query -->|PROD| StaticJSON[(public/data/users.json)]
  Router --> Pages[Routes & Boundaries]
  CI[GitHub Actions] --> Build[Vite build]
  Build --> Deploy[Azure Static Web Apps]
```

---

## Project Structure

```
src/
  api/
    users.ts           # query hooks, fetchers
  components/
    Button.tsx
    UserCard.tsx
  features/
    members/
      MembersList.tsx
      MemberDetail.tsx
      AddMemberModal.tsx
  hooks/
  pages/
    Home.tsx
    Members.tsx
    About.tsx
  router/
    index.tsx          # router + route tree
  store/
    filtersSlice.ts
    index.ts
  mocks/
    browser.ts         # MSW worker
    handlers.ts        # GET/POST handlers
    seed.ts            # deterministic fake users
  styles/
    index.css
  config.ts            # URLs, flags
main.tsx
AppLayout.tsx

public/
  data/users.json      # prod data (static)
  favicon.svg

staticwebapp.config.json
```

---

## Getting Started

### Prerequisites

- **Node.js 20+**
- **npm** (or pnpm/yarn; scripts assume npm)

### Install & Run

```bash
# 1) Install deps
npm ci

# 2) Start dev (MSW auto-starts in dev)
npm run dev

# 3) Run Storybook
npm run storybook

# 4) Tests + coverage
npm run test:ci
```

> Note: In dev/test, requests to `/api/users` are intercepted by MSW. In production, the app reads `public/data/users.json`.

---

## Environment & Configuration

This project avoids secrets; most settings are static. Optional flags:

```
# .env (optional)
# Vite exposes vars as import.meta.env.VITE_*
VITE_APP_NAME=prj-workspace
VITE_LOG_LEVEL=info
```

> No API keys required. For Azure SWA you’ll need the deployment token as a GitHub secret.

---

## Available Scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .ts,.tsx",
    "format": "prettier --write .",
    "test": "vitest",
    "test:ci": "vitest run --coverage",
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build -o dist-storybook"
  }
}
```

**Coverage thresholds** are configured via `@vitest/coverage-v8` (adjust in `vitest.config.ts`).

---

## Testing & Coverage

- **Unit tests:** reducers, utils, pure components.
- **Integration tests:** components using Query with MSW handlers.
- **JSDOM environment** for DOM rendering.

Run locally:

```bash
npm run test
npm run test:ci   # includes coverage
```

Example MSW setup in tests:

```ts
// test/setup.ts
import { setupServer } from 'msw/node'
import { handlers } from '@/mocks/handlers'
const server = setupServer(...handlers)
beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

---

## Storybook

- Uses **Vite builder**; Tailwind is loaded via preview setup.
- Addon suggestions: `@storybook/addon-a11y`, `@storybook/addon-interactions`.

Run:

```bash
npm run storybook
npm run build-storybook
```

Consider publishing Storybook alongside the app (e.g., at `/storybook/`).

---

## Accessibility & i18n

- Keyboard and screen‑reader friendly components (focus ring, skip‑link, ARIA labels).
- Basic **i18n** with `i18next` for **en** and **bg** UI labels; persisted locale in Redux.
- Storybook’s **A11y addon** lights up color contrast and aria issues.

---

## Performance

- **Code splitting** via TanStack Router lazy routes.
- **Prefetch** member details on hover/focus.
- **Vite + rollup-plugin-visualizer** for bundle insight.
- Memoized lists and stable keys.

---

## CI/CD

### Continuous Integration (GitHub Actions)

- `.github/workflows/ci.yml` runs lint + tests + coverage on push/PR.

### Deploy to Azure Static Web Apps

- `.github/workflows/azure-swa.yml` builds `dist/` and deploys.
- Create an **Azure Static Web App** in the portal and connect the repo, or add the **SWA deploy** action with a token.
- Required secret: `AZURE_STATIC_WEB_APPS_API_TOKEN`.
- SPA config & headers via `staticwebapp.config.json`:

```json
{
  "navigationFallback": { "rewrite": "/index.html" },
  "mimeTypes": { ".json": "application/json" },
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; img-src 'self' https://api.dicebear.com data:; style-src 'self' 'unsafe-inline';"
  }
}
```

> Pull Requests receive **preview environments** automatically when using SWA.

---

## Security & Repo Hygiene

- No secrets in the repo; production uses static JSON.
- Strong **CSP** (adjust as you add integrations).
- **Husky + lint-staged** to enforce lint/format on commit.
- Adopt **Conventional Commits**; consider adding `CHANGELOG.md`.
- Suggested: branch protection rules (require status checks, code review).

---

## Data Sources (Dev vs Prod)

- **Dev/Test**: `/api/users` via **MSW**. Deterministic seed data lives in `src/mocks/seed.ts`.
- **Prod**: `/data/users.json` (static). Replace or regenerate as needed.

To (re)generate `public/data/users.json` from the seed (optional utility):

```ts
// scripts/generate-users.ts (example)
import { seededUsers } from '../src/mocks/seed'
import { writeFileSync, mkdirSync } from 'fs'
mkdirSync('public/data', { recursive: true })
writeFileSync('public/data/users.json', JSON.stringify(seededUsers, null, 2))
console.log('Wrote public/data/users.json')
```

Run with:

```bash
node --loader ts-node/esm scripts/generate-users.ts
```

---

## Troubleshooting

- **MSW not intercepting?** Ensure the worker starts only in dev and you are requesting `/api/...` paths.
- **CORS or CSP errors in prod?** Adjust `staticwebapp.config.json` headers.
- **Router 404 on refresh?** Confirm `navigationFallback` rewrite is present.
- **Coverage < thresholds?** Add tests or exclude generated code.
- **Storybook styles missing?** Verify Tailwind is included in `.storybook/preview.ts`.

---

## Roadmap

- [x] App shell & routes
- [x] Query for users (list/detail)
- [x] Redux filters & theme
- [x] MSW + deterministic seed data
- [x] Storybook with a11y
- [x] Vitest + coverage ≥ 90% lines
- [x] Lazy routes + prefetch
- [x] i18n (en/bg)
- [x] Azure SWA deployment & PR previews
- [ ] E2E smoke tests (Playwright)
- [ ] Accessibility test in CI
- [ ] Visual regression for critical components

---

## Contributing

PRs welcome! Please follow Conventional Commits and keep PRs focused. Run `npm run lint` and `npm run test:ci` before pushing.

---

## License

MIT © prj-workspace contributors
