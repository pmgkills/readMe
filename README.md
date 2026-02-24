# 🚀 Lineage UI — Production Setup & Multi-Agent Architecture Guide

This document captures the complete step-by-step setup for building a **large-scale data lineage UI** using:

* Marquez API (local)
* Next.js (App Router)
* TypeScript
* React Flow
* Tailwind CSS
* shadcn/ui
* TanStack Query
* dagre layout
* Zustand (optional)
* Claude Code `/agents` multi-agent orchestration
* Local Docker deployment

---

# 🧠 Architecture Overview

## Goal

Build a scalable lineage platform where a **main Claude agent delegates work** to specialized agents and the UI renders dataset- and column-level lineage with click-through navigation.

## Agent Flow

```
User Request
   ↓
Main Orchestrator
   ↓
├─ Marquez Fetcher
├─ Column Lineage Builder
├─ Graph Builder
└─ React UI Generator
```

---

# 🚀 STEP 1 — Bootstrap Next.js App

```bash
npx create-next-app@latest lineage-ui \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"

cd lineage-ui
```

---

# 🚀 STEP 2 — Install Core Dependencies

```bash
npm install reactflow @tanstack/react-query dagre zustand
npm install clsx tailwind-merge
npm install axios
```

---

# 🚀 STEP 3 — Install shadcn/ui

Initialize:

```bash
npx shadcn@latest init
```

Choose:

* TypeScript: **yes**
* Tailwind: **yes**
* App Router: **yes**

Install core components:

```bash
npx shadcn@latest add button card dialog sheet input badge scroll-area
```

---

# 🚀 STEP 4 — Create Folder Structure

```bash
mkdir -p src/lib
mkdir -p src/components/lineage
mkdir -p src/components/graph
mkdir -p src/store
mkdir -p src/app/api/marquez
mkdir -p src/types
mkdir -p agents
```

---

# 🚀 STEP 5 — Environment Configuration

Create:

```bash
touch .env.local
```

Add:

```env
MARQUEZ_BASE_URL=http://localhost:5000
```

---

# 🚀 STEP 6 — Next.js Marquez Proxy (Critical)

Create:

```bash
touch src/app/api/marquez/route.ts
```

## Purpose

* Avoid CORS
* Enable caching later
* Centralize Marquez access

## Minimal Implementation

```ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(req: NextRequest) {
  const url = new URL(req.url)
  const path = url.searchParams.get('path')

  const res = await fetch(
    `${process.env.MARQUEZ_BASE_URL}${path}`
  )

  const data = await res.json()
  return NextResponse.json(data)
}
```

✅ All agents must call this proxy.

---

# 🚀 STEP 7 — TanStack Query Provider

Create:

```bash
touch src/components/providers/query-provider.tsx
```

This will wrap the app and manage server state.

---

# 🚀 STEP 8 — React Flow Base Setup

Create:

```bash
touch src/components/graph/lineage-graph.tsx
touch src/components/graph/layout-dagre.ts
```

## Large DAG Requirements

You will later enable:

* node memoization
* incremental expansion
* virtualization
* cached layouts

---

# 🚀 STEP 9 — Zustand Store (Optional but Recommended)

Create:

```bash
touch src/store/lineage-store.ts
```

Use for:

* selected node
* graph cache
* UI state

---

# 🤖 STEP 10 — Claude Code Multi-Agent Setup

Create the agents directory (if not already):

```bash
mkdir -p agents
```

---

## 🧠 Main Orchestrator Agent

```bash
touch agents/main-orchestrator.yaml
```

```yaml
name: lineage-orchestrator
description: >
  Main coordinator for lineage UI generation and data processing.
  Delegates work to specialized agents and validates outputs.

model: claude-3-5-sonnet

system_prompt: |
  You are a senior data platform orchestrator.

  Your responsibilities:

  1. Understand lineage visualization requests
  2. Break work into subtasks
  3. Delegate to the correct specialist agent
  4. Validate returned structures
  5. Ensure outputs are production-ready

  Delegation rules:

  - Use marquez-fetcher for API retrieval
  - Use column-lineage-builder for column parsing
  - Use lineage-graph-builder for React Flow conversion
  - Use react-lineage-ui for UI generation

  Never fabricate lineage data.
  Always prefer structured JSON outputs.
```

---

## 🔹 Marquez Fetcher Agent

```bash
touch agents/marquez-fetcher.yaml
```

```yaml
name: marquez-fetcher
description: Fetches lineage data via Next.js proxy

model: claude-3-5-haiku

system_prompt: |
  You are a Marquez API expert.

  Responsibilities:

  - Call the Next.js proxy endpoint /api/marquez
  - Normalize responses
  - Handle large payloads safely
  - Return clean JSON

  Important:

  - Base proxy path: /api/marquez
  - Never call Marquez directly
  - Preserve namespace/job/dataset relationships

  Output must be valid JSON.
```

---

## 🔹 Column Lineage Builder Agent

```bash
touch agents/column-lineage-builder.yaml
```

```yaml
name: column-lineage-builder
description: Builds column-level lineage graph

model: claude-3-5-sonnet

system_prompt: |
  You are a data lineage modeling expert.

  Tasks:

  - Parse OpenLineage columnLineage facets
  - Build source-to-target column edges
  - Deduplicate nodes
  - Handle missing column lineage gracefully
  - Optimize for large graphs

  Output format:

  {
    "nodes": [],
    "edges": []
  }

  Never invent lineage.
```

---

## 🔹 Graph Builder Agent

```bash
touch agents/lineage-graph-builder.yaml
```

```yaml
name: lineage-graph-builder
description: Converts lineage into React Flow format

model: claude-3-5-sonnet

system_prompt: |
  You are a frontend graph visualization expert.

  Convert lineage data into:

  - React Flow nodes
  - React Flow edges
  - dagre layout hints

  Requirements:

  - Must scale to large DAGs
  - Avoid duplicate nodes
  - Provide stable node IDs
  - Include type field for custom nodes

  Output must be JSON only.
```

---

## 🔹 React UI Generator Agent

```bash
touch agents/react-lineage-ui.yaml
```

```yaml
name: react-lineage-ui
description: Generates React lineage UI components

model: claude-3-5-sonnet

system_prompt: |
  You are a senior React + Next.js engineer.

  Generate:

  - React Flow components
  - TanStack Query hooks
  - Click-through navigation
  - Performance optimizations for large graphs
  - shadcn-based UI panels

  Requirements:

  - Use App Router
  - Use TypeScript
  - Use Tailwind
  - Use React Flow best practices
  - Code must be production quality

  Never generate pseudo-code.
```

---

# 🐳 STEP 11 — Docker (Local Deployment)

Create placeholders:

```bash
touch Dockerfile
touch docker-compose.yml
```

These will later run:

* Next.js app
* Marquez
* optional Postgres

---

# 🛡️ npm Audit Remediation Guide

Modern JS stacks often show vulnerabilities. Follow this order.

---

## Step A — Safe Fix

```bash
npm audit fix
npm audit
```

---

## Step B — Force Fix (if needed)

```bash
npm audit fix --force
```

Then verify:

```bash
npm run build
npm run dev
```

---

## Step C — Inspect Report

```bash
npm audit
```

### Lower Risk (usually safe)

* devDependencies only
* postcss
* glob
* minimatch
* esbuild

### Higher Risk (investigate)

* axios
* next
* reactflow
* runtime dependencies

---

## Step D — Update Next.js

```bash
npm install next@latest
npm audit
```

---

## Step E — Clean Reinstall (very effective)

```bash
rm -rf node_modules package-lock.json
npm install
npm audit
```

---

# 🚀 Recommended Next Build Milestones

## Phase 1 (MVP)

* Dataset lineage graph
* Click-through navigation
* Marquez proxy
* Basic React Flow

## Phase 2

* Column lineage parsing
* Column drilldown panel
* Search + filtering

## Phase 3 (Large-Scale Hardening)

* Graph virtualization
* incremental expansion
* layout caching
* performance tuning

---

# ✅ Current System Status

You now have:

* Production Next.js foundation
* Multi-agent Claude architecture
* Marquez proxy layer
* Large DAG readiness
* Audit remediation process
* Docker scaffolding

---

# 🔮 High-Impact Next Steps

When ready, implement:

* React Flow + dagre production layout
* Graph performance tuning
* Docker compose for full stack
* Auth scaffolding (future)

---

**End of Document**
