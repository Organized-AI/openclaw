# HelixDB memory substrate branch

Repo: `Organized-AI/openclaw`  
Branch: `helix`  
Created: 2026-06-12  
Fit label: **OpenClaw private long-term memory backend**

## Why this branch exists

This branch marks `Organized-AI/openclaw` as one of the labeled Organized-AI HelixDB fits. The intent is not to replace Penumbra as a review/capture surface; it is to evaluate HelixDB as an owned, programmable backend for operational memory.

Clean framing:

> Penumbra = review/capture UI. HelixDB = owned operational memory substrate.

## Repo-specific HelixDB use

Use HelixDB for user facts, sessions, messages/events, channel relationships, skills, device/node memory, assistant history, semantic recall, and graph traversal over people/repos/sessions/tools.

## First integration sketch

- Add a small local `helix-memory` adapter/service behind this repo's existing event or persistence boundary.
- Ingest events as graph entities: `Agent`, `Session`, `Run`, `ToolCall`, `Artifact`, `Repo`, `File`, `Skill`, `Finding`, `Memory`, `UserFact`, `Video`, `Moment`, `Entity` as applicable.
- Link with edges such as `RAN`, `CALLED`, `PRODUCED`, `MODIFIED`, `MENTIONS`, `DERIVED_FROM`, `VALIDATED_BY`, `FAILED_WITH`, `FIXED_BY`, `RELATED_TO`.
- Use text fields for BM25, embeddings for vector recall, and graph traversal for provenance/explainability.
- Run local HelixDB with persistent disk mode for any serious pilot; default local mode is in-memory.

## Guardrails

- Keep private/NDA data local unless Helix Cloud/privacy posture is separately proven.
- Do not treat this as a Penumbra replacement for human review until this repo has a review/approval UX.
- Prefer a narrow pilot: event ingest + `search_memory`/`related_context`/`trace_run` before wider rewrites.

---

# HelixDB / Organized-AI fit evaluation

Source: https://github.com/HelixDB/helix-db
Inspected: repo README, Cargo workspace, docs llms-full, GitHub metadata, Organized-AI repo inventory via `gh repo list`.

## HelixDB summary

HelixDB is a Rust graph-vector database aimed at knowledge graphs, RAG, and AI memory. The public repo currently presents v3/Cloud-style Helix as an object-storage-backed graph database with integrated approximate vector search and BM25 full-text search. Queries are authored with TypeScript/Rust/Go/Python SDK DSLs or dynamic JSON, then sent to `POST /v1/query`.

Notable properties:
- Graph + vector + BM25 text in the same query/transaction.
- Dynamic query model; no separate deployment/migration step for queries.
- Local dev via CLI/container on port 6969; in-memory default, `--disk` for persistent local storage via MinIO/object-store path.
- Cloud architecture: gateway + single writer + autoscaling readers + object storage.
- ACID / serializable snapshot isolation claims for every query.
- SDKs: README examples for Rust and TypeScript; docs mention Go/Python too.
- Agent affordances: `helix chef`, `npx skills add HelixDB/skills`, docs MCP at `https://docs.helix-db.com/mcp`.
- License file is Apache-2.0.
- GitHub metadata at inspection: ~5,024 stars, 264 forks, release v3.0.5, primary language Rust.

## Better than Penumbra?

For Zu's stack: HelixDB is probably better than Penumbra when the job is an operational, code-owned memory substrate, not just a reviewed knowledge capture UI.

HelixDB advantages over Penumbra for our codebases:
- Local/self-hostable dev path; can keep private agent memory off third-party SaaS.
- Programmable API/SDKs; easier to wire into OpenClaw/Hermes/Jockey services.
- Unified graph + vector + text queries; fits repo/video/entity memory where relationships matter.
- ACID semantics; better fit for event traces, agent state, and build-factory memory than ad hoc captures.
- Agent-native setup (`chef`, skills, docs MCP) matches the Organized Harness / Codex / Claude Code workflow.

Penumbra still wins for:
- Immediate human review / staged deltas / visual workspace review.
- Quick demo/synthesis capture without standing up infra.
- Existing MCP capture flow already works for non-sensitive public notes.

Recommendation: treat HelixDB as the likely backend for Organized AI-owned memory/brain products; keep Penumbra as a review/demo/capture surface until Helix has a usable internal schema + UI.

## Best Organized-AI fits

### 1. `openclaw` / `openclaw-1` / `openclawdmolt`
Best fit. OpenClaw is the personal assistant/gateway across channels. HelixDB can become the local/private long-term memory store: user facts, sessions, skills, channel events, entities, and relationships. It could replace scattered SQLite/vector-store memory with a graph-native substrate.

### 2. `organized-harness`
Very strong fit. Organized Harness already has AutoAgent → traces → Meta-Harness. HelixDB can store traces, eval results, harness versions, failures, fixes, and dependency relationships as a graph. Vector/text search over trace snippets plus graph traversal over agent/run/tool/code entities is exactly the use case.

### 3. `jockey-hermes`
Strong fit. Jockey video intelligence produces moments, scenes, products/offers, ad variants, Stripe SKUs, and performance feedback. HelixDB could unify TwelveLabs moment IDs, catalog entities, creative variants, and downstream Meta buyer outcomes in one queryable graph.

### 4. `model-trader`
Strong fit, if strategy provenance matters. Store trader transcripts, extracted rules, gates, ticker setups, paper trades, and outcomes. Graph traversal maps content → thesis/rule → screener gate → trade → result; vector/text search retrieves similar setups.

### 5. `multi-agent-observability` and `openclaw-observability-monitor`
Good fit after initial product maturity. Current stack is hook events → SQLite/WebSocket/dashboard. HelixDB could add graph queries across sessions, agents, tools, files, errors, cost, and outcomes. Not urgent unless the dashboard needs cross-run semantic memory.

### 6. `organized-codebase`
Good as an optional plugin/template. Add a `with-helix-memory` template for Claude Code projects that need persistent project memory, codebase graph, and verification traces.

### 7. `mcporter`, `plugin-marketplace`, `hermes-webhooks`
Integration surfaces rather than primary homes. MCPorter could wrap Helix as typed MCP/CLI; plugin-marketplace could publish a Helix memory plugin; hermes-webhooks could stream events into Helix.

### 8. `orectic-penumbra-docs`
Good documentation/testbed fit. It already compares context/knowledge systems. Add HelixDB as the third axis: programmable graph-vector DB vs reviewed knowledge workspace.

## Suggested pilot

Pilot in `organized-harness` or `openclaw`, not in Jockey first.

MVP schema:
- Node labels: `Agent`, `Session`, `Run`, `ToolCall`, `Artifact`, `Repo`, `File`, `Skill`, `Finding`, `Memory`, `UserFact`, `Video`, `Moment`, `Entity`.
- Edges: `RAN`, `CALLED`, `PRODUCED`, `MODIFIED`, `MENTIONS`, `DERIVED_FROM`, `VALIDATED_BY`, `FAILED_WITH`, `FIXED_BY`, `RELATED_TO`.
- Indexed props: `tenant`, `repo`, `session_id`, `timestamp`, `label`, `source_url`.
- Vector props: `embedding` on `Memory`, `Finding`, `Artifact`, `Moment`, `FileChunk`.
- Text props: `content`, `summary`, `error`, `title`.

First build target:
- A tiny local `helix-memory` service that ingests Hermes/OpenClaw events and exposes MCP tools:
  - `remember_entity`
  - `search_memory`
  - `trace_run`
  - `related_context`
  - `commit_fact`

Risk/caveats:
- Need verify local runtime stability and SDK maturity; repo/docs mix v2/v3/Cloud naming.
- Local default is in-memory, so any serious local pilot must use `--disk`.
- Cloud/private-data posture needs diligence before NDA/M&A use.
- Need our own review/approval UX if replacing Penumbra staged deltas.

