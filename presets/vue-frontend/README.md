# vue-frontend Preset

Stack-standards preset for the AI-Filter frontend: Vue3 + TypeScript + Pinia + Naive UI + Tailwind + ECharts/Konva/Three + MQTT.

## What It Provides

| File | Strategy | Content |
|---|---|---|
| `templates/constitution-template.md` | replace | Approved constitution: typing / Composition API / state & data flow / async & errors / design-before-code tiered OOD / mandatory i18n / coupling / size limits / styling / render hot paths / e2e acceptance / YAGNI, plus the locked stack and engineering conventions |
| `templates/plan-template.md` | replace | Prefilled Technical Context and the real repository tree; adds render hot-path assessment, OOD tiering, and E2E acceptance mapping sections; preserves the Constitution Check and Complexity Tracking skeleton |
| `templates/checklist-addendum.md` | append | Stack-level mandatory items: reuse/typing/i18n/state/testing/commits |

## Install

```bash
# Development (install straight from the fork working tree)
specify preset add --dev /path/to/spec-kit/presets/vue-frontend

# Regular install (requires catalog registration)
specify preset add vue-frontend
```

Projects that want the constitution materialized in sync with the preset should install the
`constitution-sync` preset first (a hand-edited constitution is never overwritten).

## Sources

- `CODE_STYLE.md` in the frontend repository (including repo-level requirements such as mandatory i18n and Conventional Commits)
- Spec Kit customization plan (`myprojectdocs/speckit-融合规划.md`), engineering-quality clauses 2.2–2.7

## Amendments

Numeric changes (size limits, coverage) bump MINOR; adding/removing principles bumps MAJOR. Re-run `preset add` in each project after releasing a new version.
