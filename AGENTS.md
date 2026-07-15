# AGENTS.md

Instructions for agents and contributors working in this repo. This file is read by AI coding agents (Claude Code, etc.) and humans alike. Its main job: keep versioning consistent.

## What this repo is

A **public, code-free case study** documenting the architecture of Mealr. The application source is private. Deliverables here are docs and diagrams: `README.md` and `docs/`.

## Versioning

The single source of truth for the version is the **`VERSION`** file (currently `1.7.0`), using **semantic versioning** `MAJOR.MINOR.PATCH`.

Because this repo is documentation (not shipping code), interpret SemVer by *content impact*:

- **MAJOR** (`X.0.0`) — a substantial restructure or rewrite, or a change that invalidates the previous framing/claims.
- **MINOR** (`x.Y.0`) — new material: a new section, a new diagram, or documenting a newly added Mealr service (e.g., the MCP server addition that took this repo from `1.0.0` → `1.1.0`).
- **PATCH** (`x.y.Z`) — typo and wording fixes, link/diagram corrections, and other small edits that don't change the meaning.

Keep three things **in sync at the same number**: the `VERSION` file, the git **tag** (`vX.Y.Z`), and the **GitHub release**.

## How to update the version

When you make a change worth releasing:

1. Make the content change (edit `README.md` / `docs/`).
2. Update **`VERSION`** to the new number per the rules above.
3. Commit together:
   ```bash
   git add -A
   git commit -m "vX.Y.Z — <short summary of the change>"
   ```
4. Tag the commit (annotated):
   ```bash
   git tag -a vX.Y.Z -m "vX.Y.Z — <short summary>"
   ```
5. Push the commit and the tag:
   ```bash
   git push
   git push origin vX.Y.Z
   ```
6. Create the GitHub release from the tag:
   ```bash
   gh release create vX.Y.Z --title "vX.Y.Z — <title>" --notes "<release notes>"
   ```
   (or: repo → Releases → Draft a new release → choose the tag → publish)

## Guidance for AI agents

- If you change content in this repo, **also bump `VERSION`** following the rules above, and mention the new version in your commit message.
- Don't bump `MAJOR` for additive changes; new sections/diagrams are `MINOR`.
- Don't invent release notes that overstate scope — describe only what changed.

## Version history

- **1.7.0** — Corrected and expanded the ingestion section to the real architecture: three import sources (PDF, multi-page photo, URL), a **Dispatcher + Step Functions** pipeline (replacing the inaccurate "Coordinator fan-out over FIFO" depiction), per-user concurrency capped via a `QUEUED` DynamoDB row status (the SQS overflow queue was removed), ETag-conditional index writes, and an `Import Job Succeeded` event driving KB sync. Diagram section 2 and Mermaid redrawn to match.
- **1.6.0** — Added the `insights` service: a managed Bedrock AgentCore **Harness** agent that generates recipe-library insights, with per-user isolation enforced as a hard IAM boundary (inline tools read S3 under a scoped Job-Lambda role; the Harness has zero data access). Async, S3-backed; owner-only refresh. Passkey/WebAuthn MFA; service count 9 → 10.
- **1.5.0** — Ingestion pipeline is now agentic: a Coordinator fans out concurrent agentic recipe workers (Bedrock Converse, PDF inlined as a document block) over a FIFO queue, with per-user ordering and adaptive backoff.
- **1.4.1** — Diagram polish: fixed the MCP tool note overflowing the canvas; added a "Uploaded via web app" origin to the ingestion pipeline.
- **1.4.0** — Ask rebuilt as an agentic, streaming (SSE) assistant on the Strands Agents framework; MCP 2.0 replaced `ask_about_recipes` with `semantic_search_recipes` (retrieval-as-a-tool).
- **1.3.0** — MCP now exposes shopping-list tools (incl. write actions: create/add/update); documented the recipes-read / shopping-write permission boundary.
- **1.2.0** — Expanded MCP toolset (9 tools incl. `ask_about_recipes` / RAG over MCP); added the static marketing site to the platform overview.
- **1.1.0** — Added the MCP server (`mealr-mcp`) to the architecture, docs, and diagram.
- **1.0.0** — Initial public case study: architecture write-up + system diagram.
