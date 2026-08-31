# monalert-backend Preset

Stack-standards preset for the monAlert monitoring & alerting backend: FastAPI + Celery + asyncpg + InfluxDB + Redis + MQTT + MinIO.

## What It Provides

| File | Strategy | Content |
|---|---|---|
| `templates/constitution-template.md` | replace | Approved constitution: typing / async-first / unified response / data layering / tiered OOD / coupling / size limits / hot paths / distributed exceptions / YAGNI, plus the locked stack and quality bars |
| `templates/plan-template.md` | replace | Prefilled Technical Context and the real repository tree; adds hot-path assessment, OOD tiering, and distributed exception analysis sections; preserves the Constitution Check and Complexity Tracking skeleton |
| `templates/checklist-addendum.md` | append | Stack-level mandatory items: reuse/coupling, SQL/storage, endpoint docs, concurrency/distribution |

## Install

```bash
# Development (install straight from the fork working tree)
specify preset add --dev /path/to/spec-kit/presets/monalert-backend

# Regular install (requires catalog registration)
specify preset add monalert-backend
```

Projects that want the constitution materialized in sync with the preset should install the
`constitution-sync` preset first (a hand-edited constitution is never overwritten).

## Sources

- "Monitoring & Alerting Backend" chapter of the team coding standards (`docs/项目编码规范文档.md`)
- Spec Kit customization plan (`myprojectdocs/speckit-融合规划.md`), engineering-quality clauses 2.2–2.7

## Amendments

Numeric changes (size limits, coverage, performance defaults) bump MINOR; adding/removing principles bumps MAJOR. Re-run `preset add` in each project after releasing a new version.
