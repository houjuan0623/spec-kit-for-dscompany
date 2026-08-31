# monAlert Monitoring & Alerting Backend Constitution

> This is an approved constitution distributed with the `monalert-backend` preset. Every article is a
> machine-checkable hard rule: the Constitution Check gate in `/speckit.plan` and the `/speckit.analyze`
> audit use this document as their sole basis. Operational handbook content (deployment, log mapping,
> etc.) does not belong here — see the repository README and AGENTS.md.

## Core Principles

### I. Mandatory Typing & Naming

Every function parameter and return value MUST carry a type annotation; variables/functions use
snake_case, classes use PascalCase, constants use UPPER_SNAKE_CASE.
**Check**: any untyped function signature or naming violation fails the gate.

### II. Async First

Functions performing I/O (PostgreSQL, Redis, InfluxDB, HTTP, MQTT, MinIO) MUST be `async def` and use
async libraries; pure CPU work uses `def` (offloaded to the FastAPI thread pool); unavoidable
synchronous I/O MUST be wrapped via `loop.run_in_executor`.
Concurrent subtasks MUST use `asyncio.gather()` or `anyio.create_task_group()`.
**Check**: calling blocking I/O directly on an async path fails the gate.

### III. Unified Response & Exceptions

Public HTTP endpoints MUST return the unified response body `{code, data, message}`; business
exceptions MUST inherit from the shared base (AppException) and be caught by the global
`exception_handler`; HTTP status codes MUST follow RESTful semantics (201/204/401/403/404, etc.).
**Check**: endpoints returning bare payloads, exceptions swallowed by scattered try/except blocks, or
misused status codes fail the gate.

### IV. Data Layering Discipline

Data MUST land in its designated layer: relational data → PostgreSQL (asyncpg, connections managed
with `async with`); time-series metrics → InfluxDB; cache and queues → Redis (db0 = Celery broker,
db1 = Celery backend, db6 = RedBeat, db7 = application cache); file objects → MinIO (bucket names in
lowercase-with-hyphens, file names carrying timestamp and device ID).
**Check**: new data placed in the wrong store, a wrong Redis db, or direct connections bypassing the
unified `pgsqlHandle`/`influxHandle` (and peers) fail the gate.

### V. Design Before Code, Tiered OOD

Implementation MUST NOT precede design artifacts; tasks MUST be traceable to design elements.
Tiered OOD applies:
- **Simple tier** (single-module CRUD, no cross-module collaboration): data-model.md is sufficient.
- **Complex tier** (cross-module collaboration / new alerting strategy / new storage medium / Celery
  orchestration): Phase 1 MUST additionally produce a class design table (class, single
  responsibility, collaborators) and a sequence diagram of the key flow (router → api → *Handle
  cross-layer calls).
**Check**: a complex-tier feature missing the class table or sequence diagram fails the gate; forcing
full OOD onto a simple-tier feature counts as over-engineering and equally fails.

### VI. High Cohesion, Low Coupling

Dependencies MUST point one way: `routers → api/<domain> → *Handle (pgsql/influx/redis/mqtt/minio/http)
→ tools`; reverse references are forbidden; importing another api sub-domain's private modules is
forbidden (cross-domain access goes through public interfaces only); zero tolerance for circular
imports; shared logic MUST sink into `tools/`, business logic MUST NOT enter `tools/`.
**Check**: any reverse dependency, cross-domain private import, or circular import fails the gate.

### VII. File Size Limits

Python modules ≤ 500 lines, functions ≤ 50 lines. Anything over the limit MUST be split by
responsibility (extract sub-modules / collaborator classes); mechanical slicing is forbidden.
**Check**: a file/function exceeding the limit after the change without a split task fails the gate.
(Numbers are the team's recommended starting point; changes go through Governance.)

### VIII. Performance Hot Paths (conditional)

A feature hitting any item on the **hot-path list** MUST provide an explicit performance budget, a
Phase 0 performance research task, and a load-test scenario in the plan: high-frequency endpoints,
batch/scheduled Celery tasks, large-volume SQL queries, Redis hot-key or bulk operations, InfluxDB
writes and aggregation queries, MQTT real-time links.
Features not on the list MUST adopt the stack defaults (API p95 < 200ms; list queries always
paginated; Celery tasks finish within their schedule period) and skip the performance work.
**Check**: hitting the list without a budget, or fabricating metrics without hitting it, both fail
the gate.

### IX. Distributed Exception Analysis (conditional)

This system is high-concurrency and deploys distributed (horizontally scaled multi-worker). Any
feature hitting a hot path **or** touching cross-process shared state MUST include a distributed
exception analysis in the plan, with a mitigation per category: node crash / network partition,
duplicate message consumption and idempotency, distributed lock failure, Celery task retry and loss,
cache inconsistency. Features not affected MUST state "no distributed state involved" explicitly.
**Check**: an affected feature missing any category fails the gate.

### X. Minimal Implementation & Dependency Discipline

MUST NOT introduce abstraction layers, design patterns, or configuration options for hypothetical
needs (YAGNI); a new third-party dependency MUST be justified in research.md using the
Decision / Rationale / Alternatives-considered format before adoption; before writing a new function,
`tools/` MUST be searched — reuse what exists, and new shared functions MUST land in `tools/`.
**Check**: an unjustified new dependency, or a new function duplicating `tools/`, fails the gate.

## Locked Technology Stack

| Dimension | Locked choice |
|---|---|
| Language | Python 3.13 |
| Web framework | FastAPI (uvicorn) |
| Task queue | Celery + RedBeat dynamic scheduling (task names `module.function`, retries configured explicitly via `bind=True, max_retries`) |
| Relational store | PostgreSQL (asyncpg async driver) |
| Time-series store | InfluxDB (queries must bound the time range and use aggregation functions) |
| Cache/queue | Redis (db allocation per Principle IV; colon-separated keys, e.g. `device:{id}:status`) |
| Object store | MinIO |
| Messaging | MQTT (hierarchical topics `device/{device_id}/status`, QoS 0/1/2 by importance) |
| Testing | pytest + pytest-asyncio + pytest-mock |
| Deployment | Docker (secrets via environment variables only) |

Adding or replacing a stack component is a constitutional amendment and goes through Governance.

## Quality & Security Bars

- Unit-test coverage of core business logic (api sub-domains and celeryHandle tasks) MUST be ≥ 80%.
- SQL MUST use parameter binding (asyncpg placeholders); string-concatenated SQL is forbidden.
- Secrets (keys, connection strings) MUST be injected via environment variables only; `.env` MUST NOT
  be committed or shipped in images.
- Alert levels MUST use the unified four tiers: Info / Warning / Error / Tip.
- Logs MUST include timestamp, level, module, and message, rotated by time or size.

## Governance

- This constitution supersedes other practices; `/speckit.plan` runs the Constitution Check once
  before Phase 0 and again after Phase 1; unavoidable violations MUST be justified in the plan's
  Complexity Tracking table, otherwise the feature does not proceed to tasks.
- Amendment flow: proposal → team review → semantic version bump → distribute as a new preset release.
- Adjusting numeric articles (size limits, coverage, performance defaults) is a MINOR amendment;
  adding/removing principles is MAJOR.

**Version**: 1.0.0 | **Ratified**: 2026-08-27 | **Last Amended**: 2026-08-31
