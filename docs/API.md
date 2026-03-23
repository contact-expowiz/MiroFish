# REST API reference

Base URL for local development:

```bash
export BASE=http://localhost:5001
```

All API routes below are prefixed with `/api` unless noted. Responses are JSON unless a download endpoint returns a file.

---

## Health (no `/api` prefix)

**GET** `/health`

```bash
curl -s "$BASE/health"
```

Example response:

```json
{"status": "ok", "service": "MiroFish Backend"}
```

---

## Graph API (`/api/graph`)

### Generate ontology (upload seeds)

**POST** `/api/graph/ontology/generate`  
Content-Type: `multipart/form-data`

| Field | Required | Description |
|-------|----------|-------------|
| `files` | Yes | One or more files: `.pdf`, `.md`, `.txt`, `.markdown` |
| `simulation_requirement` | Yes | Natural-language prediction question |
| `project_name` | No | Display name (default: `Unnamed Project`) |
| `additional_context` | No | Extra instructions for the LLM |

```bash
curl -s -X POST "$BASE/api/graph/ontology/generate" \
  -F "files=@./seed-report.pdf" \
  -F "simulation_requirement=If event X happens next week, how might online discussion evolve?" \
  -F "project_name=Example Project"
```

Save `project_id` from `data.project_id` (referred to as `proj_xxx` below).

### Build Zep graph

**POST** `/api/graph/build`  
Content-Type: `application/json`

```json
{
  "project_id": "proj_xxx",
  "graph_name": "My graph",
  "chunk_size": 500,
  "chunk_overlap": 50,
  "force": false
}
```

```bash
curl -s -X POST "$BASE/api/graph/build" \
  -H "Content-Type: application/json" \
  -d '{"project_id":"proj_xxx","graph_name":"My graph"}'
```

Returns `task_id`. Poll until complete.

### Poll graph build task

**GET** `/api/graph/task/<task_id>`

```bash
curl -s "$BASE/api/graph/task/task_xxx"
```

### Projects and tasks (CRUD-style)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/graph/project/<project_id>` | Project details |
| GET | `/api/graph/project/list?limit=50` | List projects |
| DELETE | `/api/graph/project/<project_id>` | Delete project |
| POST | `/api/graph/project/<project_id>/reset` | Reset for rebuild |
| GET | `/api/graph/tasks` | List tasks |

### Graph data and Zep cleanup

**GET** `/api/graph/data/<graph_id>` — nodes/edges summary from Zep.

```bash
curl -s "$BASE/api/graph/data/graph_xxx"
```

**DELETE** `/api/graph/delete/<graph_id>` — delete graph in Zep.

```bash
curl -s -X DELETE "$BASE/api/graph/delete/graph_xxx"
```

---

## Simulation API (`/api/simulation`)

### Create simulation

**POST** `/api/simulation/create`

```json
{
  "project_id": "proj_xxx",
  "graph_id": "graph_xxx",
  "enable_twitter": true,
  "enable_reddit": true
}
```

`graph_id` is optional if already stored on the project.

```bash
curl -s -X POST "$BASE/api/simulation/create" \
  -H "Content-Type: application/json" \
  -d '{"project_id":"proj_xxx"}'
```

Returns `simulation_id` (`sim_xxx`).

### Prepare simulation (async)

**POST** `/api/simulation/prepare`

```json
{
  "simulation_id": "sim_xxx",
  "entity_types": ["Student", "Professor"],
  "use_llm_for_profiles": true,
  "parallel_profile_count": 5,
  "force_regenerate": false
}
```

```bash
curl -s -X POST "$BASE/api/simulation/prepare" \
  -H "Content-Type: application/json" \
  -d '{"simulation_id":"sim_xxx"}'
```

### Prepare status

**POST** `/api/simulation/prepare/status`

```json
{
  "task_id": "task_xxx",
  "simulation_id": "sim_xxx"
}
```

Either field helps disambiguate; `simulation_id` can short-circuit if already prepared.

```bash
curl -s -X POST "$BASE/api/simulation/prepare/status" \
  -H "Content-Type: application/json" \
  -d '{"simulation_id":"sim_xxx","task_id":"task_xxx"}'
```

### Get simulation

**GET** `/api/simulation/<simulation_id>`

```bash
curl -s "$BASE/api/simulation/sim_xxx"
```

### List and history

**GET** `/api/simulation/list?project_id=proj_xxx`  
**GET** `/api/simulation/history?limit=20` — enriched list for dashboards.

### Start simulation

**POST** `/api/simulation/start`

```json
{
  "simulation_id": "sim_xxx",
  "platform": "parallel",
  "max_rounds": 40,
  "enable_graph_memory_update": false,
  "force": false
}
```

`platform`: `twitter`, `reddit`, or `parallel`. Use a modest `max_rounds` to control cost.

```bash
curl -s -X POST "$BASE/api/simulation/start" \
  -H "Content-Type: application/json" \
  -d '{"simulation_id":"sim_xxx","platform":"parallel","max_rounds":40}'
```

### Stop simulation

**POST** `/api/simulation/stop`

```json
{"simulation_id": "sim_xxx"}
```

### Run status and actions

**GET** `/api/simulation/<simulation_id>/run-status`  
**GET** `/api/simulation/<simulation_id>/run-status/detail?platform=twitter`  
**GET** `/api/simulation/<simulation_id>/actions?limit=100&offset=0&platform=reddit`  
**GET** `/api/simulation/<simulation_id>/timeline?start_round=0`  
**GET** `/api/simulation/<simulation_id>/agent-stats`

```bash
curl -s "$BASE/api/simulation/sim_xxx/run-status"
```

### Entities (from Zep)

**GET** `/api/simulation/entities/<graph_id>?entity_types=Student,Professor&enrich=true`  
**GET** `/api/simulation/entities/<graph_id>/<entity_uuid>`  
**GET** `/api/simulation/entities/<graph_id>/by-type/<entity_type>`

```bash
curl -s "$BASE/api/simulation/entities/graph_xxx?enrich=true"
```

### Profiles and config

**GET** `/api/simulation/<simulation_id>/profiles?platform=reddit`  
**GET** `/api/simulation/<simulation_id>/profiles/realtime?platform=twitter`  
**GET** `/api/simulation/<simulation_id>/config`  
**GET** `/api/simulation/<simulation_id>/config/realtime`  
**GET** `/api/simulation/<simulation_id>/config/download` — download `simulation_config.json`

### Standalone profile generation

**POST** `/api/simulation/generate-profiles`

```json
{
  "graph_id": "graph_xxx",
  "entity_types": ["Student"],
  "use_llm": true,
  "platform": "reddit"
}
```

### Script downloads

**GET** `/api/simulation/script/<script_name>/download`  
Allowed `script_name`: `run_twitter_simulation.py`, `run_reddit_simulation.py`, `run_parallel_simulation.py`, `action_logger.py`.

```bash
curl -s -O -J "$BASE/api/simulation/script/run_parallel_simulation.py/download"
```

### Posts and comments (SQLite)

**GET** `/api/simulation/<simulation_id>/posts?platform=reddit&limit=50`  
**GET** `/api/simulation/<simulation_id>/comments?post_id=123&limit=50`

### Interview (requires live env)

Interview routes need the simulation environment to accept commands. Check first:

**POST** `/api/simulation/env-status`

```json
{"simulation_id": "sim_xxx"}
```

```bash
curl -s -X POST "$BASE/api/simulation/env-status" \
  -H "Content-Type: application/json" \
  -d '{"simulation_id":"sim_xxx"}'
```

**POST** `/api/simulation/interview`

```json
{
  "simulation_id": "sim_xxx",
  "agent_id": 0,
  "prompt": "What is your view on the main controversy?",
  "platform": "twitter",
  "timeout": 60
}
```

Omit `platform` to query both platforms in dual-platform mode (see server response shape).

**POST** `/api/simulation/interview/batch` — body includes `interviews: [{ "agent_id", "prompt", "platform"? }, ...]`.  
**POST** `/api/simulation/interview/all` — same `prompt` for every agent.

**POST** `/api/simulation/interview/history`

```json
{
  "simulation_id": "sim_xxx",
  "platform": "reddit",
  "agent_id": 0,
  "limit": 100
}
```

### Close environment (graceful)

**POST** `/api/simulation/close-env`

```json
{"simulation_id": "sim_xxx", "timeout": 30}
```

---

## Report API (`/api/report`)

### Generate report (async)

**POST** `/api/report/generate`

```json
{
  "simulation_id": "sim_xxx",
  "force_regenerate": false
}
```

```bash
curl -s -X POST "$BASE/api/report/generate" \
  -H "Content-Type: application/json" \
  -d '{"simulation_id":"sim_xxx"}'
```

Returns `task_id` and `report_id`.

### Report generation status

**POST** `/api/report/generate/status`

```json
{
  "task_id": "task_xxx",
  "simulation_id": "sim_xxx"
}
```

### Get report

**GET** `/api/report/<report_id>`  
**GET** `/api/report/by-simulation/<simulation_id>`  
**GET** `/api/report/list?simulation_id=sim_xxx&limit=50`  
**GET** `/api/report/<report_id>/download` — Markdown file  
**DELETE** `/api/report/<report_id>`

### Progress and sections (streaming-style)

**GET** `/api/report/<report_id>/progress`  
**GET** `/api/report/<report_id>/sections`  
**GET** `/api/report/<report_id>/section/<section_index>` — e.g. `1` for first section

### Report Agent chat

**POST** `/api/report/chat`

```json
{
  "simulation_id": "sim_xxx",
  "message": "Summarize the strongest counter-arguments.",
  "chat_history": []
}
```

```bash
curl -s -X POST "$BASE/api/report/chat" \
  -H "Content-Type: application/json" \
  -d '{"simulation_id":"sim_xxx","message":"What were the key risks identified?"}'
```

### Report / interview gating

**GET** `/api/report/check/<simulation_id>` — returns `interview_unlocked` when a completed report exists (used by the UI).

### Agent and console logs (debugging)

**GET** `/api/report/<report_id>/agent-log?from_line=0`  
**GET** `/api/report/<report_id>/agent-log/stream`  
**GET** `/api/report/<report_id>/console-log?from_line=0`  
**GET** `/api/report/<report_id>/console-log/stream`

### Debug tools (Zep)

**POST** `/api/report/tools/search`

```json
{
  "graph_id": "graph_xxx",
  "query": "narrative themes",
  "limit": 10
}
```

**POST** `/api/report/tools/statistics`

```json
{"graph_id": "graph_xxx"}
```

---

## Error handling

Typical error payload:

```json
{
  "success": false,
  "error": "Human-readable message"
}
```

Some 500 responses include `traceback` in debug mode. Check `success` before reading `data`.
