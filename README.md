# Mealr — an agentic AI recipe platform

> **Case study / architecture showcase.** This repository documents the design of Mealr, a production serverless platform with conversational AI over a user's own recipes. The application source is private; this repo exists to walk through the architecture and the agentic-AI engineering behind it.

Built and architected by **Derrick Bryant** — [LinkedIn](https://www.linkedin.com/in/derrick-bryant) · [GitHub](https://github.com/dbryant4)

---

## Contents

- [What it is](#what-it-is)
- [The agentic AI, in depth](#the-agentic-ai-in-depth)
  - [Conversational RAG over personal data](#conversational-rag-over-personal-data-ask-service)
  - [Vector knowledge base](#vector-knowledge-base-kb-service)
  - [Document-AI ingestion pipeline](#document-ai-ingestion-pipeline-pdf-ingestor-service)
- [System architecture](#system-architecture)
- [Tech stack](#tech-stack)
- [Engineering practices on display](#engineering-practices-on-display)
- [My role](#my-role)

---

## What it is

Mealr turns a pile of scanned recipe PDFs into a private, searchable, **conversational** cookbook. Upload a photo or PDF of a recipe and an ingestion pipeline extracts it into structured data; ask questions in plain English ("what can I make with what's in the fridge?", "add the Tuesday pasta to my shopping list") and a retrieval-grounded assistant answers using **only your own recipes**, with citations.

It's a full multi-service platform — auth, APIs, a React web app, an event-driven ingestion pipeline, and a Bedrock-backed knowledge base — deployed entirely as serverless infrastructure-as-code.

---

## The agentic AI, in depth

This is the part that matters for an engineering audience. Mealr isn't a chatbot bolted onto a CRUD app — retrieval, grounding, and per-user isolation are designed in.

### Conversational RAG over personal data (`ask` service)
- **Amazon Bedrock Converse** with a **Knowledge Base**, model **Claude Sonnet 4.6**.
- **Multi-turn chat** — the client sends conversation history each turn; threads persist client-side.
- **Per-user retrieval scoping** — retrieval is filtered to the caller's identity (`userId` metadata), so one user can never retrieve another's recipes. Shared-library access is an explicit, granted exception (`ownerSub`).
- **Grounded citations** — when the answer references specific recipes, the response includes structured citations (`slug`, `title`, `snippet`, `url`), validated against what retrieval actually returned. Chit-chat and irrelevant retrieval hits are deliberately *not* cited.
- **Structured tool-style output** — a separate `listRecipes` field returns only positive matches the model recommends for shopping, kept distinct from display citations and validated against retrieval — reducing hallucinated recommendations.

### Vector knowledge base (`kb` service)
- Converts each structured recipe into a Markdown document plus a Bedrock metadata sidecar.
- Maintains a shared **Bedrock Knowledge Base backed by Amazon S3 Vectors**, with `userId` metadata for filtered retrieval.
- **Event-driven sync** via EventBridge, with debouncing when many recipes change at once.

### Document-AI ingestion pipeline (`pdf-ingestor` service)
- Drop a scanned recipe PDF in S3 → an **EventBridge → Step Functions** pipeline extracts it into structured JSON (ingredients with consistent units, steps, images), scoped by `userId`.
- Job status tracked in DynamoDB for the UI; SQS buffers overflow; idempotent reprocessing and data migrations are first-class.

---

## System architecture

![Mealr system architecture](docs/architecture.svg)

<sub>Three flows — the request path, the document-AI ingestion pipeline, and the conversational-RAG ask flow.</sub>

<details>
<summary>Text version (Mermaid)</summary>

```mermaid
flowchart TB
  user(["User browser"])

  subgraph edge ["Edge"]
    web["Web app<br/>React + Vite SPA on CloudFront"]
    gw["API gateway<br/>base-path routing"]
  end

  subgraph identity ["Identity"]
    cognito["Cognito User Pool<br/>OIDC + MFA"]
  end

  subgraph apis ["HTTP APIs — each with its own JWT authorizer"]
    account["account-api"]
    recipes["recipes-api"]
    shopping["shopping-list-api"]
    ask["ask-api  (conversational RAG)"]
  end

  subgraph ai ["Agentic AI"]
    kb["Knowledge Base<br/>Bedrock + S3 Vectors"]
    bedrock["Bedrock Converse<br/>Claude Sonnet 4.6"]
    ingest["PDF ingestor<br/>EventBridge + Step Functions"]
  end

  subgraph data ["Storage"]
    s3in["S3 — PDF uploads"]
    s3out["S3 — structured recipes"]
    ddb["DynamoDB — jobs & data"]
  end

  user --> web --> gw
  web --> cognito
  gw --> account & recipes & shopping & ask
  s3in --> ingest --> s3out
  s3out -- EventBridge --> kb
  ask --> kb --> bedrock
  ask --> bedrock
  recipes --> ddb
  ingest --> ddb
```

</details>

---

## Tech stack

| Area | Choices |
|---|---|
| **AI** | Amazon Bedrock (Converse + Knowledge Bases), Amazon S3 Vectors, Claude Sonnet 4.6 |
| **Compute** | AWS Lambda, Step Functions, EventBridge, SQS (serverless, event-driven) |
| **Data** | DynamoDB, S3 |
| **Auth** | Amazon Cognito (OIDC, MFA) |
| **Web** | React, Vite, TypeScript, Tailwind, TanStack Query, react-oidc-context |
| **APIs** | HTTP APIs with per-stack JWT authorizers; OpenAPI specs |
| **Infra** | AWS CDK (Python) — 100% infrastructure-as-code, multi-stack |
| **Quality** | pytest test suites, generated API docs, per-service semantic versioning |

---

## Engineering practices on display

- **Microservices done deliberately** — eight independently deployable services, each with its own stack, authorizer, OpenAPI contract, tests, and SemVer version, wired together through an API gateway and EventBridge.
- **Security & multi-tenancy** — Cognito OIDC + MFA, JWT authorizers per service, and retrieval that is *provably* scoped per user.
- **Event-driven, idempotent pipelines** — debounced KB sync, overflow buffering, reprocessing, and data migrations treated as first-class concerns.
- **Everything as code** — all infrastructure in CDK; reproducible deploys; architecture captured in a versioned diagram checked into the repo.

---

## My role

I designed and built the entire platform end to end — the agentic AI (RAG design, retrieval scoping, grounding and citation validation, the document-extraction pipeline), the serverless architecture, the APIs and auth, and the React front-end. Mealr is where I prove the thing I care about professionally: bringing **agentic AI into production reliably**, not just as a demo.

---

*Source code is private. This repository is a design write-up; reach out if you'd like a deeper walkthrough.*
