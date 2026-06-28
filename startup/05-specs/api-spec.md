---
type: Spec
title: "Conductor REST API Specification v0.1"
description: Formal specification for the Conductor HTTP API — authentication, rate-limit tiers, compile and run endpoints, pipeline management, SSE event stream, OpenAI-compatible inference proxy, and unified error response format.
resource: /startup/05-specs/api-spec.md
tags: [api, rest, sse, websocket, endpoints, authentication, spec, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor REST API Specification v0.1

**Status**: Draft  
**Applies to**: Conductor API v0.1.x  
**Base URL**: `https://api.conductorlabs.io/v1`  
**Conformance suite**: `tests/conformance/api/`

---

## 1. Authentication

### 1.1 Bearer JWT

All endpoints (except `GET /v1/healthz`) require an `Authorization` header with a valid Bearer JWT:

```
Authorization: Bearer <jwt>
```

JWTs are issued by the Conductor identity provider (Clerk) on user login or via the API key management page. Tokens contain the following standard claims plus Conductor-specific claims:

| Claim | Type | Description |
|---|---|---|
| `sub` | String | User ID (`user_<id>`) |
| `org_id` | String | Tenant/organization ID (`org_<id>`) |
| `tier` | String | `"free"`, `"builder"`, `"team"`, or `"enterprise"` |
| `exp` | Number | Expiry timestamp (Unix epoch) |
| `iat` | Number | Issued-at timestamp |

Tokens expire after 1 hour. Clients must re-authenticate and obtain a new token. The API never returns a refreshed token; refresh must be performed via the Clerk SDK.

**API Keys** (long-lived service tokens) follow the same bearer scheme. They are prefixed with `cond_sk_` and do not expire until revoked. API keys carry the same claims as JWTs plus `"key_id"` for audit.

### 1.2 Rate Limits

Rate limits are enforced per `org_id` (tenant), not per user.

| Tier | Compile requests/min | Concurrent runs | LLM calls/min |
|---|---|---|---|
| Free | 10 | 2 | 50 |
| Builder | 60 | 10 | 300 |
| Team | 200 | 50 | 1,000 |
| Enterprise | Custom (negotiated) | Custom | Custom |

Rate limit state is tracked in Redis with a sliding window algorithm. When a limit is exceeded, the API returns HTTP 429 with a `Retry-After` header (seconds until the window resets) and an error body per §7.

### 1.3 Content Negotiation

All request and response bodies are `application/json` unless otherwise noted (SSE endpoints use `text/event-stream`). The server ignores unknown top-level request fields to allow forward compatibility.

---

## 2. Compile Endpoint

Compiles a `.agent` source string to IR. Does not execute. Stateless — no run record is created.

### `POST /v1/compile`

**Request body**:

```json
{
  "source": "pipeline RefactorDeadCode:\n  model default: claude-3-5-sonnet\n  ...",
  "options": {
    "strict": true,
    "target": "bedrock"
  }
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `source` | String | Yes | Full `.agent` source text |
| `options.strict` | Boolean | No (default `false`) | If `true`, warnings are promoted to errors and compilation fails on any warning |
| `options.target` | String | No (default `"bedrock"`) | IR target: `"bedrock"` (AWS Bedrock) or `"vllm"` (self-hosted). Affects model routing metadata in the IR |

**Response 200 (success)**:

```json
{
  "ir": [
    {"op": "CALL", "tool": "fs.glob", "args": ["./src/**/*.ts"], "output": "$source_files"},
    "..."
  ],
  "warnings": [
    {
      "code": "E007",
      "message": "Back-edge detected on step Validate→FindDeadCode. This may cause an infinite loop.",
      "step": "Validate",
      "line": 28,
      "column": 7
    }
  ],
  "errors": [],
  "stats": {
    "tokens": 112,
    "steps": 6,
    "edges": 7,
    "back_edges": 1,
    "compile_ms": 14
  }
}
```

**Response 422 (compile error)**:

```json
{
  "errors": [
    {
      "code": "E001",
      "message": "Undefined variable $source_files referenced before assignment.",
      "step": "FindDeadCode",
      "line": 15,
      "column": 14
    },
    {
      "code": "E005",
      "message": "Unknown model alias 'turbo'. Declare it with 'model turbo: <model-id>' in the pipeline header.",
      "step": "RefactorEach",
      "line": 22,
      "column": 11
    }
  ],
  "warnings": [],
  "ir": null,
  "stats": null
}
```

**Response fields**:

| Field | Type | Description |
|---|---|---|
| `ir` | Array or null | Compiled IR instruction array. `null` if compilation failed |
| `warnings` | Array | Warning objects (may be empty) |
| `errors` | Array | Error objects (empty on success) |
| `stats.tokens` | Number | Estimated token count for the compiled pipeline (static estimate) |
| `stats.steps` | Number | Number of `step` declarations in the pipeline |
| `stats.edges` | Number | Number of directed edges in the step graph |
| `stats.back_edges` | Number | Number of back-edges (potential loops) |
| `stats.compile_ms` | Number | Compile latency in milliseconds |

Error/warning objects share the same shape:

| Field | Type | Description |
|---|---|---|
| `code` | String | Error code from Language Spec §4.4 (e.g., `"E001"`) |
| `message` | String | Human-readable description |
| `step` | String or null | Name of the step where the error occurred |
| `line` | Number or null | 1-based line number in the source |
| `column` | Number or null | 1-based column number in the source |

---

## 3. Run Endpoints

### `POST /v1/runs`

Creates and enqueues a new pipeline run from compiled IR.

**Request body**:

```json
{
  "ir": [
    {"op": "CALL", "tool": "fs.glob", "args": ["./src/**/*.ts"], "output": "$source_files"},
    "..."
  ],
  "config": {
    "models": {
      "default": "claude-3-5-sonnet",
      "codegen": "gpt-4o"
    },
    "budget_tokens": 32000,
    "timeout_seconds": 3600,
    "allow_network": false,
    "cache_llm_calls": false
  },
  "pipeline_id": "pipe_xyz"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `ir` | Array | Yes | Compiled IR array from `/v1/compile` |
| `config.models` | Object | Yes | Map of model alias → model ID for all aliases used in the IR |
| `config.budget_tokens` | Number | No (default 32768) | Token budget for context window (must match `context budget` in source) |
| `config.timeout_seconds` | Number | No (default 3600) | Wall-clock timeout for the entire run |
| `config.allow_network` | Boolean | No (default `false`) | Enable outbound network in `shell.run` containers |
| `config.cache_llm_calls` | Boolean | No (default `false`) | Cache LLM responses by content hash (see Runtime Spec §6.3) |
| `pipeline_id` | String | No | Associate run with a saved pipeline for history tracking |

**Response 201**:

```json
{
  "run_id": "run_abc123",
  "status": "pending",
  "created_at": "2026-06-28T10:00:00Z"
}
```

---

### `GET /v1/runs/{run_id}`

Returns the current state and step-level detail for a run.

**Path parameters**: `run_id` — the run identifier returned by `POST /v1/runs`.

**Response 200**:

```json
{
  "run_id": "run_abc123",
  "pipeline_id": "pipe_xyz",
  "status": "running",
  "current_step": "FindDeadCode",
  "ir_index": 4,
  "steps": [
    {
      "name": "Ingest",
      "status": "completed",
      "started_at": "2026-06-28T10:00:05Z",
      "completed_at": "2026-06-28T10:00:05Z",
      "duration_ms": 234,
      "tokens_used": 0,
      "tool_calls": 1,
      "llm_calls": 0
    },
    {
      "name": "FindDeadCode",
      "status": "running",
      "started_at": "2026-06-28T10:00:06Z",
      "completed_at": null,
      "duration_ms": null,
      "tokens_used": 12400,
      "tool_calls": 0,
      "llm_calls": 1
    }
  ],
  "tokens_total": 12400,
  "budget_tokens": 32000,
  "cost_usd": 0.047,
  "started_at": "2026-06-28T10:00:05Z",
  "completed_at": null,
  "output": null
}
```

**Response fields**:

| Field | Type | Description |
|---|---|---|
| `run_id` | String | Run identifier |
| `pipeline_id` | String or null | Associated pipeline ID |
| `status` | String | Current state: `pending`, `running`, `paused`, `completed`, `failed`, `aborted` |
| `current_step` | String or null | Name of the currently-executing step |
| `ir_index` | Number | IR instruction index currently being executed (or about to execute) |
| `steps` | Array | Per-step detail objects (one entry per step defined in the pipeline, in declaration order) |
| `tokens_total` | Number | Cumulative tokens consumed so far |
| `budget_tokens` | Number | Token budget for this run |
| `cost_usd` | Number | Cumulative LLM cost in USD |
| `started_at` | String or null | ISO 8601 UTC timestamp |
| `completed_at` | String or null | ISO 8601 UTC timestamp (null if not yet completed) |
| `output` | String or null | Final output string (non-null only when `status == "completed"`) |

---

### `GET /v1/runs`

Lists runs for the authenticated tenant.

**Query parameters**:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `status` | String | (all) | Filter by status: `pending`, `running`, `paused`, `completed`, `failed`, `aborted` |
| `pipeline_id` | String | (all) | Filter by pipeline |
| `limit` | Number | 20 | Page size (max 100) |
| `offset` | Number | 0 | Pagination offset |
| `order` | String | `"desc"` | `"asc"` or `"desc"` by `created_at` |

**Response 200**:

```json
{
  "runs": [
    {
      "run_id": "run_abc123",
      "pipeline_id": "pipe_xyz",
      "status": "completed",
      "created_at": "2026-06-28T10:00:00Z",
      "completed_at": "2026-06-28T10:02:41Z",
      "cost_usd": 0.11
    }
  ],
  "total": 42,
  "limit": 20,
  "offset": 0
}
```

---

### `POST /v1/runs/{run_id}/approve`

Resolves a `PAUSED` run that is suspended at a `human.checkpoint` instruction.

**Request body**:

```json
{
  "decision": "approved",
  "notes": "Looks good, the dead code items are correct. Proceed."
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `decision` | String | Yes | `"approved"` or `"rejected"` |
| `notes` | String | No (default `""`) | Human-provided comment; stored in the `ApprovalResult.notes` field |

**Response 200**:

```json
{
  "run_id": "run_abc123",
  "status": "running",
  "resumed_at": "2026-06-28T10:01:15Z"
}
```

**Error 409** (run not in `PAUSED` state):

```json
{
  "error": {
    "code": "INVALID_STATE",
    "message": "Run run_abc123 is in state 'running', not 'paused'. Only paused runs can be approved.",
    "details": {"current_status": "running"}
  }
}
```

---

### `POST /v1/runs/{run_id}/retry`

Retries a `FAILED` run from its last committed checkpoint.

**Request body**: empty (`{}`)

**Response 200**:

```json
{
  "run_id": "run_abc123",
  "status": "running",
  "retry_from_ir_index": 7,
  "retried_at": "2026-06-28T10:05:00Z"
}
```

---

### `DELETE /v1/runs/{run_id}`

Aborts a `PENDING`, `RUNNING`, or `PAUSED` run. Idempotent: aborting an already-`ABORTED` or `COMPLETED` run returns 200 with the current status.

**Response 200**:

```json
{
  "run_id": "run_abc123",
  "status": "aborted",
  "aborted_at": "2026-06-28T10:03:00Z"
}
```

---

## 4. Pipeline Management

### `POST /v1/pipelines`

Saves a named pipeline (source + metadata). Does not compile or run.

**Request body**:

```json
{
  "name": "refactor-dead-code",
  "source": "pipeline RefactorDeadCode:\n  ...",
  "description": "Finds and removes dead TypeScript code using Claude.",
  "tags": ["refactor", "typescript", "claude"]
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | String | Yes | URL-slug-safe name (pattern: `[a-z0-9-]+`, max 64 chars) |
| `source` | String | Yes | Full `.agent` source text |
| `description` | String | No | Human-readable description |
| `tags` | Array of String | No | Freeform tags for search |

**Response 201**:

```json
{
  "pipeline_id": "pipe_xyz789",
  "name": "refactor-dead-code",
  "description": "Finds and removes dead TypeScript code using Claude.",
  "tags": ["refactor", "typescript", "claude"],
  "created_at": "2026-06-28T09:00:00Z",
  "updated_at": "2026-06-28T09:00:00Z"
}
```

---

### `GET /v1/pipelines`

Lists pipelines for the authenticated tenant.

**Query parameters**: `limit` (default 20, max 100), `offset`, `tag` (filter by tag), `q` (full-text search on name + description).

**Response 200**:

```json
{
  "pipelines": [
    {
      "pipeline_id": "pipe_xyz789",
      "name": "refactor-dead-code",
      "description": "Finds and removes dead TypeScript code using Claude.",
      "tags": ["refactor", "typescript"],
      "run_count": 7,
      "last_run_at": "2026-06-28T10:00:00Z",
      "created_at": "2026-06-28T09:00:00Z"
    }
  ],
  "total": 3,
  "limit": 20,
  "offset": 0
}
```

---

### `GET /v1/pipelines/{pipeline_id}`

Returns a single pipeline with source and run history summary.

**Response 200**:

```json
{
  "pipeline_id": "pipe_xyz789",
  "name": "refactor-dead-code",
  "source": "pipeline RefactorDeadCode:\n  ...",
  "description": "Finds and removes dead TypeScript code using Claude.",
  "tags": ["refactor", "typescript"],
  "run_count": 7,
  "recent_runs": [
    {"run_id": "run_abc123", "status": "completed", "cost_usd": 0.11, "created_at": "2026-06-28T10:00:00Z"},
    {"run_id": "run_def456", "status": "failed", "cost_usd": 0.02, "created_at": "2026-06-27T14:30:00Z"}
  ],
  "created_at": "2026-06-28T09:00:00Z",
  "updated_at": "2026-06-28T09:00:00Z"
}
```

---

### `PUT /v1/pipelines/{pipeline_id}`

Updates an existing pipeline (source, description, or tags). All fields are optional; omitted fields are unchanged.

**Request body**: Same shape as `POST /v1/pipelines` but all fields optional.

**Response 200**: Same shape as `GET /v1/pipelines/{pipeline_id}`.

---

### `DELETE /v1/pipelines/{pipeline_id}`

Deletes a pipeline. Associated run records are retained (not deleted) with `pipeline_id` set to null.

**Response 200**:

```json
{
  "pipeline_id": "pipe_xyz789",
  "deleted": true
}
```

---

## 5. Server-Sent Events Stream

### `GET /v1/runs/{run_id}/events`

Streams real-time events for a run using the [Server-Sent Events](https://html.spec.whatwg.org/multipage/server-sent-events.html) protocol.

**Request headers**:

```
Accept: text/event-stream
Authorization: Bearer <jwt>
Cache-Control: no-cache
```

**Response**: HTTP 200 with `Content-Type: text/event-stream`. The connection is held open until the run reaches a terminal state (`COMPLETED`, `FAILED`, or `ABORTED`), after which the server sends a `run_completed` or `run_failed` event and closes the stream.

**Reconnection**: The server sends `id:` fields with each event. Clients may reconnect with `Last-Event-ID` to replay missed events from the last received ID. Events are retained for 1 hour after the run completes.

**Event format** (SSE):

```
id: 1
event: step_started
data: {"step": "Ingest", "ir_index": 0, "timestamp": "2026-06-28T10:00:05.123Z"}

id: 2
event: tool_call
data: {"step": "Ingest", "tool": "fs.glob", "args": ["./src/**/*.ts"], "result_count": 43, "duration_ms": 187, "timestamp": "2026-06-28T10:00:05.310Z"}

id: 3
event: step_completed
data: {"step": "Ingest", "duration_ms": 234, "timestamp": "2026-06-28T10:00:05.357Z"}

id: 4
event: step_started
data: {"step": "FindDeadCode", "ir_index": 2, "timestamp": "2026-06-28T10:00:05.400Z"}

id: 5
event: llm_call
data: {"step": "FindDeadCode", "model": "claude-3-5-sonnet", "prompt_tokens": 12400, "timestamp": "2026-06-28T10:00:05.450Z"}

id: 6
event: llm_response
data: {"step": "FindDeadCode", "model": "claude-3-5-sonnet", "output_tokens": 847, "cost_usd": 0.047, "duration_ms": 2341, "timestamp": "2026-06-28T10:00:07.791Z"}

id: 7
event: step_completed
data: {"step": "FindDeadCode", "duration_ms": 2391, "timestamp": "2026-06-28T10:00:07.841Z"}

id: 8
event: checkpoint
data: {"step": "ConfirmWithUser", "display_data": [{"file": "src/utils.ts", "line": 42, "reason": "Unused export"}], "timestamp": "2026-06-28T10:00:08.001Z"}

id: 9
event: run_completed
data: {"run_id": "run_abc123", "output": "Refactor complete. 3 items removed.", "total_cost_usd": 0.11, "total_tokens": 18400, "duration_ms": 156000, "timestamp": "2026-06-28T10:02:41.000Z"}
```

**Complete event type catalog**:

| Event Type | Payload Fields | Description |
|---|---|---|
| `step_started` | `step`, `ir_index`, `timestamp` | Emitted when the IR walker enters a new step |
| `llm_call` | `step`, `model`, `prompt_tokens`, `timestamp` | Emitted immediately before an LLM API request is dispatched |
| `llm_response` | `step`, `model`, `output_tokens`, `cost_usd`, `duration_ms`, `timestamp` | Emitted when the LLM response is received and parsed |
| `tool_call` | `step`, `tool`, `args`, `result_count` (for list results), `duration_ms`, `timestamp` | Emitted after a tool call completes successfully |
| `checkpoint` | `step`, `display_data`, `timestamp` | Emitted when execution is suspended at `human.checkpoint` |
| `context_eviction` | `step`, `evicted_frames`, `tokens_freed`, `timestamp` | Emitted when context frames are dropped due to budget pressure |
| `step_completed` | `step`, `duration_ms`, `timestamp` | Emitted when a step finishes all its IR instructions |
| `step_failed` | `step`, `error`, `timestamp` | Emitted when a step encounters an unrecoverable error |
| `run_completed` | `run_id`, `output`, `total_cost_usd`, `total_tokens`, `duration_ms`, `timestamp` | Terminal event: run completed successfully |
| `run_failed` | `run_id`, `error` (`{code, message}`), `total_cost_usd`, `total_tokens`, `timestamp` | Terminal event: run failed |
| `run_aborted` | `run_id`, `timestamp` | Terminal event: run aborted by user |
| `heartbeat` | `timestamp` | Emitted every 15 s when no other events are produced, to keep the connection alive |

---

## 6. OpenAI-Compatible Inference Endpoint

For users who want to route requests to Conductor's self-hosted GPU tier (Gemma4 models) using any OpenAI SDK client, Conductor exposes an OpenAI-compatible completions endpoint.

### `POST /v1/chat/completions`

**Behavior**: Proxies the request to the vLLM backend hosting Gemma4 models. The `model` field in the request must be one of the Conductor GPU-tier model IDs:
- `gemma4-26b-private` — Tier 2, A10G GPU, self-hosted
- `gemma4-12b-fast` — Tier 3, g5.xlarge GPU, self-hosted

**Request**: OpenAI Chat Completions API format. Conductor passes through all standard fields (`messages`, `temperature`, `max_tokens`, `stream`, `top_p`, etc.) without modification.

**Response**: OpenAI Chat Completions API format. Conductor adds a non-standard `x-conductor-model-tier` header to the response indicating which tier served the request.

**Streaming**: Supported. When `stream: true`, the endpoint returns a `text/event-stream` response using the OpenAI streaming format (data-only SSE lines containing JSON delta objects, terminated by `data: [DONE]`).

**Authentication**: Same Bearer JWT scheme as all other endpoints. Tenant tier must be `"builder"` or higher to access GPU-tier models; Free tier returns HTTP 403.

**Rate limits**: Counted against the tenant's LLM calls/min limit (shared with Conductor pipeline LLM calls).

**Error**: If all GPU instances are unavailable (e.g., during maintenance), returns HTTP 503 with the recommendation to fall back to a Bedrock model.

---

## 7. Error Response Format

All error responses (4xx and 5xx) follow a unified envelope:

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "You have exceeded 10 compile requests per minute on the Free tier.",
    "details": {
      "limit": 10,
      "window": "1m",
      "reset_at": "2026-06-28T10:01:00Z",
      "tier": "free"
    }
  }
}
```

**Envelope fields**:

| Field | Type | Description |
|---|---|---|
| `error.code` | String | Machine-readable error code (snake_case, uppercase) |
| `error.message` | String | Human-readable description |
| `error.details` | Object or null | Optional structured context for the error |

### 7.1 HTTP Status Code Semantics

| Status | Meaning | Common `error.code` values |
|---|---|---|
| 400 | Invalid request body (malformed JSON, missing required fields, invalid field types) | `INVALID_REQUEST`, `MISSING_FIELD`, `INVALID_FIELD_TYPE` |
| 401 | Missing or invalid JWT (expired, malformed, revoked) | `UNAUTHORIZED`, `TOKEN_EXPIRED`, `TOKEN_INVALID` |
| 403 | Valid authentication but insufficient permissions (e.g., accessing a resource owned by another tenant, using a GPU model on Free tier) | `FORBIDDEN`, `TIER_REQUIRED` |
| 404 | Resource not found | `NOT_FOUND` |
| 409 | State conflict (e.g., approving a non-paused run) | `INVALID_STATE` |
| 422 | Compile errors (from `/v1/compile`) — see §2 for the compile-specific response shape | `COMPILE_ERRORS` |
| 429 | Rate limit exceeded | `RATE_LIMIT_EXCEEDED` |
| 500 | Internal server error (unexpected exception) | `INTERNAL_ERROR` |
| 503 | Service temporarily unavailable (GPU tier down, database overloaded) | `SERVICE_UNAVAILABLE`, `GPU_UNAVAILABLE` |

### 7.2 Error Code Catalog

| HTTP | `error.code` | Description |
|---|---|---|
| 400 | `INVALID_REQUEST` | Request body is not valid JSON or violates the schema |
| 400 | `MISSING_FIELD` | A required field is absent from the request body |
| 400 | `INVALID_FIELD_TYPE` | A field has the wrong JSON type |
| 401 | `UNAUTHORIZED` | No `Authorization` header provided |
| 401 | `TOKEN_EXPIRED` | JWT has passed its `exp` claim |
| 401 | `TOKEN_INVALID` | JWT signature verification failed or token is malformed |
| 403 | `FORBIDDEN` | Authenticated user does not have access to this resource |
| 403 | `TIER_REQUIRED` | This feature requires a higher subscription tier |
| 404 | `NOT_FOUND` | The requested run, pipeline, or resource does not exist |
| 409 | `INVALID_STATE` | Operation cannot be performed in the resource's current state |
| 422 | `COMPILE_ERRORS` | Source code failed to compile; see `errors` array in response body |
| 429 | `RATE_LIMIT_EXCEEDED` | Tenant has exceeded the rate limit for this endpoint |
| 500 | `INTERNAL_ERROR` | Unexpected server-side exception; automatically reported to Sentry |
| 503 | `SERVICE_UNAVAILABLE` | Conductor backend is temporarily unavailable |
| 503 | `GPU_UNAVAILABLE` | All GPU instances are offline; fall back to Bedrock models |

---

## 8. Auxiliary Endpoints

### `GET /v1/healthz`

No authentication required. Returns HTTP 200 with `{"status": "ok"}` if the API is healthy, or HTTP 503 if any critical dependency (Postgres, Redis) is unavailable.

### `GET /v1/models`

Returns the list of available model IDs and their tier requirements.

**Response 200**:

```json
{
  "models": [
    {"id": "claude-3-5-sonnet", "tier": "bedrock", "provider": "anthropic", "min_tier": "free", "context_window": 200000},
    {"id": "llama3-70b", "tier": "bedrock", "provider": "meta", "min_tier": "free", "context_window": 128000},
    {"id": "mistral-large", "tier": "bedrock", "provider": "mistral", "min_tier": "free", "context_window": 128000},
    {"id": "gpt-4o", "tier": "openai", "provider": "openai", "min_tier": "builder", "context_window": 128000},
    {"id": "gemma4-26b-private", "tier": "gpu_private", "provider": "conductor", "min_tier": "builder", "context_window": 32768},
    {"id": "gemma4-12b-fast", "tier": "gpu_fast", "provider": "conductor", "min_tier": "builder", "context_window": 32768}
  ]
}
```

### `GET /v1/usage`

Returns token usage and cost summary for the authenticated tenant for the current billing period.

**Response 200**:

```json
{
  "period_start": "2026-06-01T00:00:00Z",
  "period_end": "2026-07-01T00:00:00Z",
  "tokens_input": 1240000,
  "tokens_output": 184000,
  "cost_usd": 4.72,
  "run_count": 93,
  "compile_count": 211
}
```
