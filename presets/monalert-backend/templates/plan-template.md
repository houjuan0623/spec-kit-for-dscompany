# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `__SPECKIT_COMMAND_PLAN__` command; its definition describes the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  Stack-level prefill (monalert-backend preset): the fields below are the locked stack for the
  monitoring & alerting backend. Rewrite a field only when this feature genuinely deviates, and
  justify the deviation in Complexity Tracking. Do NOT mark prefilled values as NEEDS CLARIFICATION.
-->

**Language/Version**: Python 3.13

**Primary Dependencies**: FastAPI (uvicorn), Celery + RedBeat, asyncpg, influxdb-client, async Redis client, MQTT client, MinIO SDK

**Storage**: PostgreSQL (relational, asyncpg) + InfluxDB (time-series metrics) + Redis (cache/queues, db allocation per Constitution Principle IV) + MinIO (file objects)

**Testing**: pytest + pytest-asyncio + pytest-mock

**Target Platform**: Linux server (Docker containers, distributed multi-worker deployment)

**Project Type**: web-service (monitoring & alerting backend, active polling + passive listening dual mode)

**Performance Goals**: [If no hot path is hit, write "stack defaults"; if hit, state an explicit budget, e.g. "alert evaluation chain p95 < 500ms, N metrics/minute per worker"]

**Constraints**: Stack baseline — API p95 < 200ms; list queries always paginated; Celery tasks finish within their schedule period; [add feature-specific constraints or N/A]

**Scale/Scope**: [Devices / metrics / task volume this feature touches, e.g. "500 devices × 1 metric/s", or N/A]

### Performance Hot-Path Assessment (required)

Assess each category against the hot-path list in Constitution Principle VIII:

| Hot-path category | Hit? | Rationale |
|---|---|---|
| High-frequency endpoint | [yes/no] | [evidence] |
| Batch / scheduled Celery task | [yes/no] | [evidence] |
| Large-volume SQL query | [yes/no] | [evidence] |
| Redis hot-key / bulk operation | [yes/no] | [evidence] |
| InfluxDB write / aggregation query | [yes/no] | [evidence] |
| MQTT real-time link | [yes/no] | [evidence] |

**Conclusion**: [No hit → adopt stack defaults, no performance work; hit → put an explicit budget in Performance Goals, add a Phase 0 performance research task, add a load-test scenario to quickstart.md]

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
  The tree below is the real monAlert repository layout (prefilled). Keep only the directories this
  feature touches, annotate the concrete files to add/modify, delete irrelevant branches, and never
  invent directories that do not exist.
-->

```text
src/
├── api/                            # API modules by monitoring/alerting domain (business logic layer)
│   ├── deviceAlarm/                # Device alarms
│   ├── metricAlarm/                # Metric alarms
│   ├── serviceAlarm/               # Service alarms
│   ├── basicServiceMonitoring/     # Basic service monitoring
│   ├── inferenceMonitoring/        # Inference monitoring
│   ├── productionLineMonitoring/   # Production line monitoring
│   ├── serviceStatusOverview/      # Service status overview
│   ├── thirdPartyServiceMonitoring/# Third-party service monitoring
│   └── aiMiddlePlatformMonitoring/ # AI middle-platform monitoring
├── celeryHandle/                   # Celery tasks (directories by schedule period)
│   ├── celery.py                   # Celery app & configuration
│   ├── second/ minute/ hour/ day/ week/ year/
│   └── fix/                        # One-off remediation tasks
├── httpHandle/                     # Outbound HTTP wrapper
├── influxHandle/                   # InfluxDB unified entry
├── minioHandle/                    # MinIO unified entry
├── mqttHandle/                     # MQTT unified entry
├── pgsqlHandle/                    # PostgreSQL unified entry (asyncpg pool)
├── redisHandle/                    # Redis unified entry
├── routers/                        # FastAPI route registration (index.py)
├── setting/                        # Configuration (index.py, logManage.py)
├── sockets/                        # WebSocket handling
├── tools/                          # Shared utilities (business logic forbidden)
└── main.py                         # Application entry point

tests/                              # pytest tests (unit / integration, mirroring src)
```

**Structure Decision**: [List the directories this feature actually touches and full paths of new files; dependency direction must satisfy Constitution Principle VI: routers → api/<domain> → *Handle → tools]

## Design Deliverables (Phase 1)

### OOD Tier Assessment (required)

**Tier**: [simple / complex] (criteria in Constitution Principle V)

- Simple tier (single-module CRUD): data-model.md only.
- Complex tier (cross-module collaboration / new alerting strategy / new storage medium / Celery
  orchestration): in addition to data-model.md, MUST produce —
  - **Class design table**: | Class | Single responsibility | Collaborators |
  - **Key-flow sequence diagram** (mermaid): covering router → api/<domain> → *Handle cross-layer
    calls and any Celery/MQTT links.

### Distributed Exception Analysis (conditionally required)

**Trigger**: [Hot path hit OR cross-process shared state involved → fill the table; otherwise write "no distributed state involved, skipped" with rationale]

| Exception category | Involved? | Mitigation |
|---|---|---|
| Node crash / network partition | [yes/no] | [mitigation or N/A] |
| Duplicate consumption & idempotency | [yes/no] | [mitigation or N/A] |
| Distributed lock failure | [yes/no] | [mitigation or N/A] |
| Celery task retry & loss | [yes/no] | [mitigation or N/A] |
| Cache inconsistency | [yes/no] | [mitigation or N/A] |

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., deviating from the locked stack] | [current need] | [why the locked stack is insufficient] |
| [e.g., exceeding size limits without splitting] | [specific problem] | [why splitting now is worse] |
