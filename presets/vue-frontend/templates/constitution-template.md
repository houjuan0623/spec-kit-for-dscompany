# AI-Filter Frontend Constitution

> This is an approved constitution distributed with the `vue-frontend` preset. Every article is a
> machine-checkable hard rule: the Constitution Check gate in `/speckit.plan` and the
> `/speckit.analyze` audit use this document as their sole basis. The full style guide lives in the
> repository's `CODE_STYLE.md`; this document only carries the hard rules the gate evaluates.

## Core Principles

### I. Mandatory Typing & Naming

Every function parameter, return value, and variable MUST carry a TypeScript type; interfaces / type
aliases / enums use PascalCase; component file names match the component name in PascalCase
(`UserProfile.vue` → `<UserProfile />`).
**Check**: any untyped signature or naming violation fails the gate.

### II. Composition API & Component Contract

Components MUST use `<script setup lang="ts">` (Options API forbidden); props MUST be declared via
typed `defineProps<{...}>()`, events via typed `defineEmits<{...}>()`; SFC section order is fixed:
template → script setup → style scoped.
**Check**: a new component using the Options API or untyped props/emits fails the gate.

### III. State & Data-Flow Discipline

Cross-component shared state MUST live in Pinia setup stores (`stores/`), persisted via
`pinia-plugin-persistedstate` when needed; component-local state MUST NOT enter a global store.
HTTP requests MUST go through the shared axios wrappers in `hooks/` (`useAxios`,
`useAxiosMonAlert`, `useAxiosDataHub`, `useAuth` — one instance per backend service, each with
request/response interceptors) — creating bare axios instances inside components is forbidden;
real-time data MUST go through the `mqtt/` wrapper.
**Check**: bypassing the shared wrappers or pushing component-local state into a store fails the gate.

### IV. Async & Error Handling

Async operations MUST use async/await; concurrent requests MUST use `Promise.all` /
`Promise.allSettled`; async calls MUST be wrapped in try/catch with user feedback through
naive-ui's `useMessage()` (backed by the `n-message-provider` in App.vue); HTTP status semantics
are fixed: 401 → redirect to login, 403 → no-permission notice, 500 → server-error notice.
**Check**: an async chain without error handling, or a home-grown notification channel, fails the gate.

### V. Design Before Code, Tiered OOD

Implementation MUST NOT precede design artifacts; tasks MUST be traceable to design elements.
Tiered OOD applies:
- **Simple tier** (single-page CRUD, no cross-module collaboration): data-model.md is sufficient
  (for the frontend this means data shapes and store state shape).
- **Complex tier** (cross-page collaboration / new visualization scene / MQTT real-time link /
  complex form flow): Phase 1 MUST additionally produce a component/composable responsibility
  table (name, single responsibility, collaborators — props/emits/store) and a key-interaction
  sequence diagram of the key flow (page → hooks/stores → axios/mqtt data flow).
**Check**: a complex-tier feature missing the responsibility table or sequence diagram fails the
gate; forcing full OOD onto a simple-tier feature counts as over-engineering and equally fails.

### VI. Mandatory Internationalization (NON-NEGOTIABLE)

This is an international product. Every user-visible text (UI, errors, notices) MUST go through
`vue-i18n`. Component-local copy lives in the component's SFC `<i18n>` block (the repository's
preferred form); copy shared across components or referenced outside components (menus, route
titles, …) lives in `locales/index.ts` under the `zh`/`en` keys. Either way both Chinese and
English MUST ship together; hard-coding user-visible copy inside components is forbidden.
**Check**: any hard-coded copy, or new text shipped in only one language, fails the gate.

### VII. High Cohesion, Low Coupling

Reusable logic has fixed homes: stateful composables → `hooks/`; stateless pure functions → `tools/`;
business logic MUST NOT enter `tools/`. Pages (`pages/`, `view/`) MUST NOT import each other;
cross-page sharing goes only through hooks / stores / tools / shared components. Zero tolerance for
circular imports.
**Check**: page-to-page imports, business logic in tools, or circular imports fail the gate.

### VIII. File Size Limits

Vue SFC ≤ 400 lines; TS modules ≤ 300 lines; functions ≤ 50 lines. Anything over the limit MUST be
split by responsibility (extract child components / composables / stores); mechanical slicing is
forbidden.
**Check**: a file/function exceeding the limit after the change without a split task fails the gate.
(Numbers are the team's recommended starting point; changes go through Governance.)

### IX. Styling Discipline

Styling MUST prefer TailwindCSS utility classes; component-own styles MUST be `scoped`; global
variables are defined only in `main.scss`.
**Check**: unscoped component styles, or large hand-written reusable styles bypassing Tailwind, fail
the gate.

### X. Render Hot Paths (conditional)

A feature hitting any item on this list MUST provide a performance budget and strategy in the plan
(virtual scrolling / chunked rendering / frame sampling / lazy loading, etc.): large lists or tables,
large-scale charts (ECharts/G6/Three/Konva scenes), high-frequency MQTT real-time render updates,
first-screen critical-path changes.
Features not on the list use the stack defaults (lazy-loaded routes, on-demand imports) with no
dedicated performance work.
**Check**: hitting the list without a budget, or fabricating metrics without hitting it, both fail
the gate.

### XI. Testing & Acceptance

Core business logic MUST reach ≥ 80% Vitest unit-test coverage, with test files in a `__tests__/`
folder next to the source; every UI feature's acceptance MUST include a Playwright e2e smoke case
(`e2e/`); e2e selectors MUST prefer `getByRole` / `data-testid` — fragile CSS hierarchy chains are
forbidden.
**Check**: a UI feature without an e2e case, or selector violations, fails the gate.

### XII. Minimal Implementation & Dependency Discipline

MUST NOT introduce abstraction layers or configuration options for hypothetical needs (YAGNI); a new
npm dependency MUST be justified in research.md using the Decision / Rationale /
Alternatives-considered format; before writing a new function, `hooks/` and `tools/` MUST be
searched — reuse what exists.
**Check**: an unjustified new dependency, or an implementation duplicating existing hooks/tools,
fails the gate.

## Locked Technology Stack

| Dimension | Locked choice |
|---|---|
| Framework | Vue 3.5+ (Composition API) + TypeScript (vue-tsc type checking) |
| Build | Vite (rolldown-vite), Node ≥ 24.10 |
| State | Pinia 3 + pinia-plugin-persistedstate |
| Routing | Vue Router 5 (lazy-loaded) |
| UI library | naive-ui (explicit imports; locale bound to vue-i18n). Element Plus is frozen legacy (App.vue shell + userManage/userInfo) — new code MUST NOT introduce it |
| Styling | TailwindCSS 4 + SCSS global variables |
| Visualization | ECharts 6 (vue-echarts), Konva, Three, AntV G6 |
| Real-time | MQTT.js (wrapped in `src/mqtt/`) |
| i18n | vue-i18n 11 (SFC `<i18n>` blocks preferred; shared copy in `src/locales/index.ts` zh/en keys) |
| HTTP | axios (shared wrappers in hooks/ — useAxios / useAxiosMonAlert / useAxiosDataHub / useAuth) |
| Testing | Vitest 4 (unit) + Playwright (e2e) |
| Lint | eslint + oxlint + prettier |

Adding or replacing a stack component is a constitutional amendment and goes through Governance.

## Engineering Conventions

- Commit messages MUST follow Conventional Commits: `<type>: <short description>` (scope optional:
  `<type>(<scope>): …`), with types limited to feat / fix / docs / style / refactor / test / chore.
- `npm run lint` and `npm run type-check` MUST pass before committing.
- Dependencies are installed with `npm i --legacy-peer-deps` (repository convention).
- Deployment is Docker-based (registry.deepsight.ai private registry); environments switch via Vite
  modes.

## Governance

- This constitution supersedes other practices; `/speckit.plan` runs the Constitution Check once
  before Phase 0 and again after Phase 1; unavoidable violations MUST be justified in the plan's
  Complexity Tracking table, otherwise the feature does not proceed to tasks.
- Amendment flow: proposal → team review → semantic version bump → distribute as a new preset release.
- Adjusting numeric articles (size limits, coverage) is a MINOR amendment; adding/removing principles
  is MAJOR.

**Version**: 1.0.0 | **Ratified**: 2026-08-27 | **Last Amended**: 2026-08-31
