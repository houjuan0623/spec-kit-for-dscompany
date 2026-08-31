## Stack-Level Mandatory Items (injected by the monalert-backend preset)

> When generating a checklist, items below MUST be included (numbering continues) whenever the
> feature touches the corresponding area; untouched areas may be omitted, but the SQL / reuse /
> coupling groups are mandatory for any code-producing feature.

### Reuse & Coupling

- [ ] Searched `tools/` and the owning `*Handle` before writing new functions; confirmed nothing reusable exists
- [ ] New shared functions landed in `tools/` (not privately inside an api sub-domain)
- [ ] No cross-sub-domain private imports; dependency direction follows routers → api → *Handle → tools
- [ ] New third-party dependencies justified in research.md (Decision/Rationale/Alternatives)

### SQL & Storage

- [ ] SQL-touching tasks confirmed index availability and pagination for list queries
- [ ] All SQL uses asyncpg parameter binding; no string concatenation
- [ ] New data placed per the layering discipline (relational→PG, time-series→Influx, cache→Redis, files→MinIO)
- [ ] Redis operations assessed for key volume, TTL policy, and correct db selection
- [ ] InfluxDB queries bound the time range and use aggregation functions

### Endpoints & Documentation

- [ ] Every new/modified route carries summary, description, and responses
- [ ] Response bodies follow the unified `{code, data, message}` shape; error codes are registered (never invented ad hoc)

### Concurrency & Distribution

- [ ] Hot-path tasks carry a performance budget and load-test scenario (otherwise stack defaults confirmed)
- [ ] Tasks touching cross-process shared state designed for idempotency and retries (matching the plan's distributed exception table)
- [ ] New Celery tasks named `module.function` with explicit retry policy
