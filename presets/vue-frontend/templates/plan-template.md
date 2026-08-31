# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `__SPECKIT_COMMAND_PLAN__` command; its definition describes the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  Stack-level prefill (vue-frontend preset): the fields below are the locked frontend stack.
  Rewrite a field only when this feature genuinely deviates, and justify the deviation in
  Complexity Tracking. Do NOT mark prefilled values as NEEDS CLARIFICATION.
-->

**Language/Version**: TypeScript (Vue 3.5+ Composition API), Node ≥ 24.10

**Primary Dependencies**: Vite (rolldown-vite), Pinia 3 (+persistedstate), Vue Router 5, Element Plus, TailwindCSS 4, vue-i18n 11, axios, MQTT.js; visualization on demand: ECharts 6 / Konva / Three / AntV G6

**Storage**: N/A (frontend; local persistence via pinia-plugin-persistedstate / localforage)

**Testing**: Vitest 4 (unit, colocated `__tests__/`) + Playwright (e2e, `e2e/` directory, chromium locally by default)

**Target Platform**: modern browsers (Docker + Nginx deployment, environments via Vite modes)

**Project Type**: web-frontend (AI-Filter platform frontend)

**Performance Goals**: [If no render hot path is hit, write "stack defaults (lazy-loaded routes, on-demand imports)"; if hit, state an explicit budget, e.g. "10k-point chart first frame < 1s, list scrolling at 60fps"]

**Constraints**: Mandatory i18n (Chinese and English); lint + type-check must pass before commit; [add feature-specific constraints or N/A]

**Scale/Scope**: [Pages / components / data volume this feature touches, or N/A]

### Render Hot-Path Assessment (required)

Assess each category against Constitution Principle X:

| Hot-path category | Hit? | Rationale |
|---|---|---|
| Large list / table | [yes/no] | [evidence] |
| Large-scale chart (ECharts/G6/Three/Konva) | [yes/no] | [evidence] |
| High-frequency MQTT real-time rendering | [yes/no] | [evidence] |
| First-screen critical path change | [yes/no] | [evidence] |

**Conclusion**: [No hit → stack defaults, no performance work; hit → put a budget in Performance Goals, add a Phase 0 performance research task (virtual scrolling / chunked rendering / frame sampling / lazy loading), add a verification scenario to quickstart.md]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file]

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (__SPECKIT_COMMAND_PLAN__ command output)
├── research.md          # Phase 0 output (__SPECKIT_COMMAND_PLAN__ command)
├── data-model.md        # Phase 1 output (__SPECKIT_COMMAND_PLAN__ command)
├── quickstart.md        # Phase 1 output (__SPECKIT_COMMAND_PLAN__ command)
├── contracts/           # Phase 1 output (__SPECKIT_COMMAND_PLAN__ command)
└── tasks.md             # Phase 2 output (__SPECKIT_COMMAND_TASKS__ command - NOT created by __SPECKIT_COMMAND_PLAN__)
```

### Source Code (repository root)

<!--
  The tree below is the real aifilter-frontend repository layout (prefilled). Keep only the
  directories this feature touches, annotate the concrete files to add/modify, delete irrelevant
  branches, and never invent directories that do not exist.
-->

```text
src/
├── @types/            # TypeScript type definitions
├── assets/            # Static assets
├── hooks/             # Composables (stateful reusable logic)
├── i18n/              # i18n configuration
├── locales/           # Chinese & English locale packs (new copy must land in both)
├── jsonToForm/        # JSON-to-form components
├── layout/            # Layout components
├── mqtt/              # MQTT wrapper (real-time data entry point)
├── nodeEditing/       # Node editing feature
├── pages/             # Page components (routed pages)
├── router/            # Router configuration (lazy-loaded)
├── stores/            # Pinia setup stores
├── tools/             # Stateless utilities (business logic forbidden)
├── view/              # View components
├── __tests__/         # Unit tests (may also live in per-source __tests__/ folders)
├── App.vue
└── main.ts

e2e/                   # Playwright e2e tests
```

**Structure Decision**: [List the directories this feature actually touches and full paths of new files; reusable-logic placement must satisfy Constitution Principle VII: stateful → hooks/, stateless → tools/, no page-to-page imports]

## Design Deliverables (Phase 1)

### OOD Tier Assessment (required)

**Tier**: [simple / complex] (criteria in Constitution Principle V)

- Simple tier (single-page CRUD, no cross-module collaboration): data-model.md only (for the
  frontend this means data shapes and store state shape).
- Complex tier (cross-page collaboration / new visualization scene / MQTT real-time link / complex
  form flow): in addition to data-model.md, MUST produce —
  - **Component/composable responsibility table**: | Name | Single responsibility | Collaborators (props/emits/store) |
  - **Key-interaction sequence diagram** (mermaid): covering page → hooks/stores → axios/mqtt data flow.

### E2E Acceptance Mapping (required for UI features)

Map the spec's acceptance scenarios to Playwright cases (Constitution Principle XI):

| Spec acceptance scenario | e2e case (e2e/*.spec.ts) | Key selectors (getByRole/data-testid) |
|---|---|---|
| [Scenario 1] | [case file & name] | [selectors] |

**Run command**: `npm run test:e2e -- --project=chromium --reporter=list`

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., deviating from the locked stack] | [current need] | [why the locked stack is insufficient] |
| [e.g., exceeding size limits without splitting] | [specific problem] | [why splitting now is worse] |
