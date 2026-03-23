# Configuration

Configuration is loaded from the **project root** `.env` file (same directory as `package.json`). The backend resolves it relative to `backend/app/config.py`. If `.env` is missing, only process environment variables apply.

## Required variables

These are validated at backend startup; missing values cause the process to exit with an error.

| Variable | Description |
|----------|-------------|
| `LLM_API_KEY` | API key for any OpenAI-compatible LLM endpoint |
| `ZEP_API_KEY` | [Zep Cloud](https://app.getzep.com/) API key for graph memory and entity storage |

Example (see also `.env.example`):

```env
LLM_API_KEY=your_api_key_here
LLM_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
LLM_MODEL_NAME=qwen-plus
ZEP_API_KEY=your_zep_api_key_here
```

## LLM endpoint

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_BASE_URL` | `https://api.openai.com/v1` | OpenAI-compatible base URL |
| `LLM_MODEL_NAME` | `gpt-4o-mini` | Model name passed to the API |

## Flask server

| Variable | Default | Description |
|----------|---------|-------------|
| `FLASK_HOST` | `0.0.0.0` | Bind address |
| `FLASK_PORT` | `5001` | HTTP port |
| `FLASK_DEBUG` | `True` | String `true`/`false` (lowercase) toggles Flask debug |
| `SECRET_KEY` | `mirofish-secret-key` | Flask secret (change in production) |

## File uploads (graph / ontology step)

Defined in `Config` in `backend/app/config.py`:

| Setting | Value |
|---------|--------|
| Max body size | 50 MiB (`MAX_CONTENT_LENGTH`) |
| Allowed extensions | `pdf`, `md`, `txt`, `markdown` |

## Text chunking (graph build)

| Variable / setting | Default | Description |
|--------------------|---------|-------------|
| `DEFAULT_CHUNK_SIZE` | `500` | Characters per chunk |
| `DEFAULT_CHUNK_OVERLAP` | `50` | Overlap between chunks |
| Override per request | JSON body on `/api/graph/build` | `chunk_size`, `chunk_overlap` |

## OASIS simulation

| Variable | Default | Description |
|----------|---------|-------------|
| `OASIS_DEFAULT_MAX_ROUNDS` | `10` | Default cap for rounds when not overridden elsewhere |

Simulation data is stored under `backend/uploads/simulations/` (see `OASIS_SIMULATION_DATA_DIR` in code).

Platform action sets (for reference) include Twitter-style actions such as `CREATE_POST`, `LIKE_POST`, `REPOST`, `FOLLOW`, `QUOTE_POST`, `DO_NOTHING`, and Reddit-style actions such as `CREATE_COMMENT`, `SEARCH_POSTS`, `TREND`, etc.

## Report Agent

| Variable | Default | Description |
|----------|---------|-------------|
| `REPORT_AGENT_MAX_TOOL_CALLS` | `5` | Tool call budget per report generation |
| `REPORT_AGENT_MAX_REFLECTION_ROUNDS` | `2` | Reflection rounds |
| `REPORT_AGENT_TEMPERATURE` | `0.5` | Sampling temperature for the report agent |

## Optional LLM boost (parallel simulation scripts)

The root `.env.example` documents optional variables used by the parallel simulation runner (not the Flask `Config` class):

```env
LLM_BOOST_API_KEY=your_api_key_here
LLM_BOOST_BASE_URL=your_base_url_here
LLM_BOOST_MODEL_NAME=your_model_name_here
```

If you do not use boost acceleration, **omit** these keys entirely (see comments in `.env.example`). They are read in `backend/scripts/run_parallel_simulation.py` when present.

## Quick verification

- Backend health: `GET http://localhost:5001/health` should return `{"status":"ok","service":"MiroFish Backend"}`.
- After editing `.env`, restart the backend so `uv run python run.py` reloads environment variables.
