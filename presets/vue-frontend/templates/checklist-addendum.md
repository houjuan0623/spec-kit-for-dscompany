## Stack-Level Mandatory Items (injected by the vue-frontend preset)

> When generating a checklist, items below MUST be included (numbering continues) whenever the
> feature touches the corresponding area; untouched areas may be omitted, but the reuse / typing /
> i18n groups are mandatory for any UI feature.

### Reuse & Coupling

- [ ] Searched `hooks/` and `tools/` before adding new logic; confirmed nothing reusable exists
- [ ] Stateful reusable logic landed in `hooks/`, stateless pure functions in `tools/`; no business logic entered `tools/`
- [ ] No direct imports between pages (pages/view); cross-page sharing goes through hooks/stores/shared components
- [ ] New npm dependencies justified in research.md (Decision/Rationale/Alternatives)

### Typing & Component Contract

- [ ] New components use `<script setup lang="ts">` with typed props/emits
- [ ] New UI built with naive-ui; no new Element Plus imports (frozen legacy)
- [ ] Component file names are PascalCase and match the component name
- [ ] SFCs over 400 lines / TS modules over 300 lines have split tasks listed

### Internationalization & Copy

- [ ] All new user-visible text goes through vue-i18n; no hard-coded copy
- [ ] Component copy in SFC `<i18n>` blocks with both zh and en; shared copy added to both zh/en keys in `locales/index.ts`

### State & Data Flow

- [ ] Cross-component shared state lives in Pinia stores; component-local state stays out of global stores
- [ ] HTTP requests use the shared hooks/ axios wrappers (useAxios*); real-time data uses the `mqtt/` wrapper
- [ ] Async chains have try/catch with naive-ui `useMessage` feedback; 401/403/500 branches follow the convention

### Testing & Acceptance

- [ ] Core logic covered by Vitest unit tests (colocated `__tests__/`)
- [ ] UI features have a Playwright e2e smoke case with getByRole/data-testid selectors (no fragile CSS chains)
- [ ] Render-hot-path tasks carry a performance budget and verification scenario (otherwise stack defaults confirmed)

### Commit & Build

- [ ] Commit messages follow Conventional Commits (feat/fix/docs/style/refactor/test/chore; scope optional)
- [ ] `npm run lint` and `npm run type-check` pass before commit
