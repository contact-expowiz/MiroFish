# MiroFish documentation

This directory holds detailed guides that go beyond the project overview in [README-EN.md](../README-EN.md).

## Guides

| Document | Description |
|----------|-------------|
| [WORKFLOW.md](./WORKFLOW.md) | End-to-end pipeline from seed upload to report and optional agent interview |
| [CONFIGURATION.md](./CONFIGURATION.md) | Environment variables, Flask settings, and tuning knobs |
| [API.md](./API.md) | REST API reference with `curl` and JSON examples |

A Chinese archive of this index is kept at [archive/README.zh.md](./archive/README.zh.md).

## Conventions

**Base URL**

Unless noted otherwise, examples assume the backend is running locally:

- `BASE=http://localhost:5001`

If you set `FLASK_PORT` in `.env`, replace `5001` with that value.

**Placeholder IDs**

Examples use consistent placeholders; substitute real values from API responses:

| Placeholder | Meaning |
|-------------|---------|
| `proj_xxx` | Project ID (from ontology generation) |
| `task_xxx` | Async task ID (graph build, simulation prepare, report generation) |
| `graph_xxx` | Zep graph ID |
| `sim_xxx` | Simulation ID |
| `report_xxx` | Report ID |

**Response shape**

Most endpoints return JSON with a top-level `success` boolean and either `data` or `error`. Check individual routes in [API.md](./API.md) for details.

**Async operations**

Graph build, simulation preparation, and report generation run in the background and return a `task_id`. Poll the corresponding task or status endpoints until completion (see [WORKFLOW.md](./WORKFLOW.md)).
