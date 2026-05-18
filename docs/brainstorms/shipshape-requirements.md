# ShipShape — Audit and Enhancement Requirements

**Project:** ShipShape — Auditing and Improving a Production TypeScript Codebase
**Target repo:** `US-Department-of-the-Treasury/ship`
**Date:** 2026-05-18
**Status:** Draft

---

## Overview

Fork, deeply audit, and measurably improve the US Department of the Treasury's `ship` project management tool — a TypeScript monorepo combining documentation, issue tracking, and sprint planning. Work proceeds in two integrated layers:

- **Layer 1 — ShipShape 7-Category Audit + Improvements:** Required by the GFA Week 4 PDF. Baseline measurements → diagnosis → targeted fixes with before/after proof in all 7 categories.
- **Layer 2 — Enhancement Package:** Additive new files that layer Week-3-style engineering rigor on top: comprehensive ARCHITECTURE.md, THREAT_MODEL.md, USERS.md, AUDIT.md, AI-COST-ANALYSIS.md, SUBMISSION.md, additional unit/API tests, observability middleware, and automated evaluation/benchmark scripts.

**Core constraint:** The unified document model (one PostgreSQL table for all content types), Yjs real-time collaboration infrastructure, and Express framework are treated as architectural constants. Improvements are targeted fixes and additions — not rewrites.

---

## Goals

1. Produce a complete, defensible audit report with baseline measurements across all 7 categories before touching any code.
2. Deliver measurable improvement in every category, each backed by reproducible before/after evidence.
3. Build a reusable evaluation harness (benchmark scripts, test additions, observability middleware) that future maintainers can re-run to detect regressions.
4. Produce engineering documentation (ARCHITECTURE.md, THREAT_MODEL.md, USERS.md) of the quality established in Week 3.
5. Demonstrate that every change preserves existing test behavior and does not degrade any currently-passing Playwright test.

---

## Primary Actors

| Actor | Role |
|---|---|
| Auditor / Engineer | Runs measurements, diagnoses root causes, implements improvements |
| Grader / Report Viewer | Reads audit docs, watches demo, evaluates before/after evidence |
| Future Maintainer | Re-runs benchmark and eval scripts after future changes |
| End Users of Ship | Admins, members, guests using the project management tool (documented in USERS.md) |

---

## Success Criteria

All 7 improvement targets from the ShipShape PDF must be met:

| Category | Improvement Target |
|---|---|
| Type Safety | Eliminate ≥25% of type safety violations; every fix uses meaningful types |
| Bundle Size | ≥15% reduction in total production bundle, OR ≥20% reduction in initial load bundle via code splitting |
| API Response Time | ≥20% P95 improvement on at least 2 endpoints, measured under identical load conditions |
| Database Query Efficiency | ≥20% query count reduction on at least one user flow, OR ≥50% improvement on slowest query |
| Test Coverage | Add 3 meaningful new tests for previously untested critical paths (or fix 3 flaky tests with RCA) |
| Runtime Error Handling | Fix 3 error handling gaps; at least one must involve a real user-facing data loss or confusion scenario |
| Accessibility | +10 Lighthouse points on the lowest-scoring page, OR fix all Critical/Serious violations on top 3 pages |

Enhancement layer is complete when all new documents (ARCHITECTURE.md, THREAT_MODEL.md, USERS.md, AUDIT.md, AI-COST-ANALYSIS.md, SUBMISSION.md) are present at repo root and the observability middleware + evaluation scripts are runnable without configuration.

---

## Scope

### In scope

- Fork the `US-Department-of-the-Treasury/ship` repo; all work on the fork.
- Editing existing files in `web/`, `api/`, `shared/`, `e2e/`, and config files (`tsconfig.json`, `vite.config.ts`, `package.json`) for the 7 improvement targets.
- Adding new files: documentation, test files, observability middleware, benchmark scripts, evaluation reports.
- Deploying the improved fork to a publicly accessible URL.

### Not in scope (protected from changes)

- The unified document model data schema (PostgreSQL table structure, `document_type` discriminator, migration files).
- The Yjs CRDT and WebSocket real-time collaboration core (`api/src/collaboration/`).
- The Terraform infrastructure definitions.
- Any existing Playwright test that currently passes — changes to test files only permitted to fix confirmed flaky tests with documented RCA.

---

## Layer 1 — 7-Category Audit and Improvements

### Category 1: Type Safety

**Audit methodology:**
- `grep -rn ': any' --include='*.ts' --include='*.tsx'` across `web/src/`, `api/src/`, `shared/`
- Count `as ` type assertions, `!` non-null assertions, `@ts-ignore`, `@ts-expect-error`
- Check `tsconfig.json` for `strict: true`; if off, run `tsc --strict --noEmit` and count errors
- Break down by package and by violation type
- Identify the 5 most violation-dense files

**Baseline deliverable:** filled metric table (total `any`, `as`, `!`, `@ts-ignore`, strict mode status, top 5 files).

**Improvement approach:** Fix the highest-density violations with genuine type narrowing, discriminated unions, and utility types. Replacing `any` with `unknown` without narrowing does not count. Each fix must preserve behavior (all tests still pass).

**Evidence requirement:** Before/after grep counts per package. TypeScript compiler error count before/after if strict mode is disabled.

---

### Category 2: Bundle Size

**Audit methodology:**
- `pnpm build` in `web/`; record total output size
- Use `rollup-plugin-visualizer` or `vite-bundle-visualizer` to generate treemap
- Identify largest chunks, largest individual dependencies, unused imports
- Cross-reference `package.json` dependencies against actual imports with `depcheck` or manual grep

**Baseline deliverable:** total bundle KB, largest chunk (name + size), chunk count, top 3 largest dependencies, unused dependency list.

**Improvement approach (choose one or both):**
- Remove genuinely unused dependencies and dead import paths
- Introduce lazy loading / `React.lazy` + `Suspense` code splitting for heavy routes (editor, sprint board)

**Evidence requirement:** Side-by-side `dist/` output before/after. Bundle visualizer screenshot/output before/after. No functionality removed.

---

### Category 3: API Response Time

**Audit methodology:**
- Seed database: 500+ documents, 100+ issues, 20+ users, 10+ sprints (via `pnpm db:seed` or custom seed script)
- Identify the 5 most-used API endpoints by tracing frontend network requests
- Benchmark with `autocannon` or `k6` at 10, 25, 50 concurrent connections; record P50/P95/P99
- Hypothesize root cause for slowest endpoints

**Baseline deliverable:** benchmark table (endpoint × P50/P95/P99 at each concurrency level).

**Improvement approach:** Address root causes — likely missing query indexes, unbatched DB calls, or unoptimized middleware. Each fix documented with root cause analysis.

**Evidence requirement:** Before/after `autocannon` output run under identical conditions (same seed data, same hardware, same concurrency).

---

### Category 4: Database Query Efficiency

**Audit methodology:**
- Enable PostgreSQL query logging via Docker env: `POSTGRES_LOG_STATEMENT=all`
- Execute 5 user flows; count total queries per flow
- Run `EXPLAIN ANALYZE` on the slowest queries
- Check `WHERE` clause columns against existing indexes in migration files
- Flag N+1 patterns: list queries triggered per item in list views

**Baseline deliverable:** user flow table (total queries, slowest query time, N+1 detected).

**Special focus — unified document model:** The single-table design with `document_type` discriminator is the highest-risk surface for N+1 queries and missing indexes. Audit `WHERE document_type = $1` queries for index coverage.

**Improvement approach (choose one):**
- Add a targeted index on the most-queried column combination (e.g., `(document_type, parent_id, workspace_id)`)
- Batch an N+1 query into a single `IN (...)` query

**Evidence requirement:** Before/after `EXPLAIN ANALYZE` output. Before/after query count per affected flow.

---

### Category 5: Test Coverage and Quality

**Audit methodology:**
- `pnpm test` — record pass/fail counts and runtime
- Read all test files in `e2e/`; catalog covered user flows
- Run suite 3 times; flag flaky tests
- Map critical flows (document CRUD, real-time sync, auth, sprint management) against test coverage
- Configure and run `c8` or `istanbul` for line/branch coverage if not already set up

**Baseline deliverable:** total tests, pass/fail/flaky counts, runtime, critical flows with zero coverage, code coverage % per package.

**Improvement approach:** Add 3 new Playwright tests for the highest-risk uncovered flows, OR fix 3 confirmed flaky tests with documented RCA and deterministic replacement. Every new test includes a comment naming the regression it prevents.

**Evidence requirement:** Before/after test count. Coverage report delta. New tests must pass on 3 consecutive runs.

---

### Category 6: Runtime Error and Edge Case Handling

**Audit methodology:**
- Open browser DevTools, run through normal usage, count console errors and warnings
- Test network disconnect mid-collaborative-edit; check for data loss and UI recovery
- Submit malformed inputs: empty forms, 10,000-character strings, HTML/script tags, null bytes
- Test two users editing the same document field simultaneously
- Throttle to 3G; identify silent failures and missing loading states
- Check server logs for unhandled promise rejections

**Baseline deliverable:** console error count, unhandled rejection count, network disconnect recovery status (pass/partial/fail), list of missing error boundaries, list of silent failures with reproduction steps.

**Improvement approach:** Fix 3 error handling gaps. At least one must involve real user-facing data loss or confusion. Each fix requires reproduction steps, before/after behavior description, and screenshot or test evidence.

**Evidence requirement:** Before/after console error counts. Screenshot or recording of before (broken) and after (fixed) state.

---

### Category 7: Accessibility Compliance

**Audit methodology:**
- Run Lighthouse accessibility audit on every major page; record scores
- Run `axe-core` via browser extension or `pa11y` CLI; categorize violations by severity (Critical/Serious/Moderate/Minor)
- Test full keyboard navigation: Tab, Enter, Escape, arrow keys to every interactive element
- Test with screen reader (NVDA or VoiceOver)
- Check color contrast ratios against WCAG 2.1 AA minimum (4.5:1 for text)

**Baseline deliverable:** Lighthouse score per page, total Critical/Serious violations, keyboard navigation completeness, color contrast failures, list of missing ARIA labels.

**Improvement approach:** Achieve +10 Lighthouse points on the lowest-scoring page OR fix all Critical/Serious violations on the 3 most important pages (dashboard, document editor, issue list).

**Evidence requirement:** Before/after Lighthouse HTML report or axe scan JSON output. Violations count before/after.

---

## Layer 2 — Enhancement Package (Add-Only New Files)

### ARCHITECTURE.md (Primary new artifact)

A comprehensive architecture document at repo root. Required sections:

#### 1. Executive Summary (~400 words)
High-level description of Ship's purpose, the monorepo structure (`web/`, `api/`, `shared/`, `e2e/`), and the core architectural decisions (everything-is-a-document, server-is-truth, boring technology). How the Week 4 enhancement layer (tests, observability, evals) integrates without disrupting the existing system.

#### 2. System Diagram
Mermaid `flowchart TB` diagram showing:
- Browser (React + Vite frontend)
- WebSocket connection (Yjs + presence)
- Express API (routes, middleware chain)
- PostgreSQL (unified document model)
- Authentication layer
- Enhancement overlays: observability middleware, eval harness, test suite

#### 3. Component and Request Flow
How a typical request travels from React component → API route → database query → response, including the middleware chain. Trace two critical flows: document creation and real-time collaborative edit.

#### 4. Data Model
Unified document model explanation: the `documents` table, `document_type` discriminator, parent-child relationships, workspace scoping. Why one table, what it costs (query complexity), what it gains (unified API, simpler migrations).

#### 5. Real-Time Collaboration
How Yjs CRDTs synchronize state between clients. WebSocket lifecycle: connection, sync, presence, disconnect-and-reconnect. Server-side Yjs state persistence.

#### 6. Testing Flow
Full test strategy: Playwright E2E tests (existing), new API unit tests (added by this project), accessibility test suite (added), benchmark/load tests (added). How to run each tier. What each tier proves.

#### 7. Evaluation and Benchmarking
The `eval/` scripts added by this project. How to run the baseline/after benchmark comparison. How to interpret before/after results for each of the 7 categories.

#### 8. Observability
The Express observability middleware added by this project: structured request logging (method, path, status, duration), error event capture, health and readiness endpoints. Why fail-open (if middleware errors, requests continue). What is deliberately excluded from logs (auth tokens, document body content).

#### 9. New Features and Enhancements
Each measurable improvement from Layer 1, explained as an architectural decision: what changed, why the original was suboptimal, what tradeoffs were made. Indexed to the 7 categories.

#### 10. Security
Trust boundaries: unauthenticated requests, authenticated user sessions, WebSocket auth, PostgreSQL access. OWASP relevance: input validation gaps, XSS surface in the rich-text editor, SQL injection surface in the ORM layer, WebSocket auth enforcement, Terraform IAM scope. Residual risks and what remains out of scope for this project.

#### 11. Database and Query Design
The unified document model's index strategy before and after improvements. EXPLAIN ANALYZE findings. N+1 patterns identified and how they were addressed. What query patterns are safe at scale vs. those that will degrade.

#### 12. Tools Used — and Why

| Tool | Role | Why chosen | Alternatives considered | Why not |
|---|---|---|---|---|
| React + Vite | Frontend framework + build | Vite is fast, React is well-understood | Next.js | SSR complexity not needed for a project management tool |
| TipTap + Yjs | Rich-text editor + CRDT sync | Production-grade collaborative editor with Yjs interop | Slate, ProseMirror directly | More plumbing required; TipTap abstracts the hard parts |
| Express | API server | Simple, mature, explicit middleware chain | Fastify, NestJS | NestJS adds framework complexity; Fastify would require migration |
| PostgreSQL | Primary database | Relational integrity, mature, proven at scale | SQLite, MongoDB | SQLite not suitable for multi-user; Mongo loses join/query power |
| Playwright | E2E testing | Best browser automation, parallel runs, official framework | Cypress, Selenium | Playwright has better concurrency and TS-first API |
| pnpm workspaces | Monorepo package management | Fast, disk-efficient, strict by default | Turborepo, Nx | Overkill for this size; pnpm alone is sufficient |
| Terraform | Infrastructure as code | Declarative cloud resources, state management | CDK, Pulumi | Team chose Terraform; changing it is out of scope |
| Docker | Containerization | Reproducible local environment, matches production | Podman | Docker is the project standard; no reason to switch |
| `autocannon` / `k6` | Load testing | Fast, scriptable, good P95/P99 output | Apache Bench, Locust | `autocannon` is Node-native; `k6` offers scripted scenarios |
| `axe-core` / `pa11y` | Accessibility scanning | WCAG rule coverage, CI-friendly CLI | Lighthouse only | Lighthouse is a score; axe gives actionable violation details |
| `rollup-plugin-visualizer` | Bundle analysis | Treemap output, Vite-compatible | `source-map-explorer` | Visualizer integrates directly into Vite config |
| OpenTelemetry / custom middleware | Observability | Structured logs, fail-open, no external service required | Datadog, New Relic | External services need credentials; custom middleware is zero-dependency for MVP |

#### 13. Known Tradeoffs
- Unified document model simplifies the API but requires careful index strategy at scale.
- Yjs state persistence on the server means WebSocket reconnect can replay full document state, but server memory grows with document size.
- Deterministic accessibility fixes (ARIA labels, contrast) are low-risk; keyboard navigation fixes in the TipTap editor are higher-risk and tested carefully.
- Observability middleware is additive and fail-open; it does not replace production APM.

---

### THREAT_MODEL.md

Security analysis of the Ship application covering:

- **Assets:** User documents, sprint/project metadata, session tokens, WebSocket connections, PostgreSQL credentials, Terraform state.
- **Trust boundaries:** Unauthenticated public → authenticated user → admin; WebSocket channel; PostgreSQL network access.
- **Attack surface map:** Authentication endpoints, document CRUD endpoints, WebSocket message handling, rich-text editor input (XSS surface), Terraform IAM scope, Docker build context.
- **OWASP relevance:** LLM Top 10 does not apply (no LLM); OWASP Web Top 10 applies: A1 Broken Access Control (document permission model), A3 Injection (ORM queries), A5 Security Misconfiguration (Docker/Terraform), A7 XSS (rich-text editor output).
- **Existing defenses:** Auth middleware, ORM parameterization, Docker multi-stage build.
- **Residual risks:** TipTap HTML sanitization scope, WebSocket message validation, Terraform state file exposure.
- **Framework references:** OWASP Web Top 10 2021, NIST SSDF.

---

### USERS.md

User documentation covering:

- **Primary users:** Admin (workspace configuration, user management), Member (document creation, issue tracking, sprint participation), Guest (read-only access if enabled).
- **User flows by role:** per-role capability matrix against Ship's features.
- **Non-users:** Unauthenticated internet users, automated scanners.
- **Access groups and permission model:** how Ship enforces workspace-level access.

---

### AUDIT.md

The formal audit report with all 7 category tables filled in:
- Measurement methodology for each category
- Baseline numbers (to be filled during audit phase)
- Identified weaknesses ranked by severity
- Improvement implemented and evidence

---

### AI-COST-ANALYSIS.md

AI tool usage and cost during development:

- Tools used: Claude Code, model version, tasks delegated
- Token/cost estimates per task category (orientation, audit analysis, fix implementation, doc writing)
- Reflection on where AI assistance was highest-leverage (codebase orientation, doc generation) vs. where manual work dominated (reading Playwright output, running benchmarks)
- Comparison: development time with AI assistance vs. estimated without

---

### SUBMISSION.md

Final submission control document:

- Canonical links: forked repo URL, deployed application URL
- Deliverables checklist (GitHub repo, audit report, improvement documentation, discovery write-up, demo video, AI cost analysis, deployed app, social post)
- Evidence inventory: before/after measurement artifacts by category
- Final blockers before submission

---

### Additional Tests (new test files, no edits to existing)

**API unit tests** (`api/src/__tests__/` or `api/src/routes/__tests__/`):
- At minimum: one test per critical API route (document CRUD, auth, sprint management)
- Use `supertest` + `jest` or `vitest`
- Each test includes a comment naming the regression it prevents
- Must not require a live database (use test doubles or an in-memory fixture)

**Accessibility regression tests** (`e2e/accessibility/`):
- `pa11y-ci` or `axe-playwright` integration running against the 3 most important pages
- Integrated into existing `pnpm test` script or a separate `pnpm test:a11y` command
- Captures Critical/Serious violation count as a numeric baseline for regression detection

**Benchmark scripts** (`eval/` or `scripts/benchmark/`):
- `benchmark-api.js` — runs `autocannon` against the 5 key endpoints; outputs structured JSON with P50/P95/P99; accepts `--baseline` flag to save and `--compare` flag to diff against saved baseline
- `benchmark-queries.sql` — EXPLAIN ANALYZE queries for the 5 key user flows; runnable via `psql`
- `benchmark-bundle.sh` — runs `pnpm build`, captures `dist/` sizes, outputs structured summary
- All scripts are runnable with `node scripts/benchmark/<script> --help` and print usage

---

### Observability Middleware (new file, additive)

**`api/src/middleware/observability.ts`** (or `observability.js`):

A fail-open Express middleware that attaches to the request pipeline without modifying any existing middleware. Records:
- `method`, `path`, `status_code`, `duration_ms` for every request
- `error_type`, `error_message` (without stack traces) for unhandled errors
- `event_type`: one of `request_completed`, `request_errored`, `server_started`

Explicitly excluded from logs: `Authorization` header value, `Cookie` header value, request body content, response body content.

**`api/src/routes/health.ts`** (new, or verify existing):
- `GET /health` → `{ status: "ok" }`
- `GET /ready` → `{ status: "ready" | "degraded", db_connected: boolean, collab_connected: boolean }`

These endpoints do not exist in the current Ship app (to be verified); if they do exist, no changes are made.

---

## Constraints

1. All 7 improvement targets must be met by the Friday deadline; measurement methodology must be reproducible.
2. Every existing passing Playwright test must still pass after improvements.
3. The unified document model schema, Yjs collab layer, and Terraform configs are not changed.
4. Before/after evidence for each improvement must be committed artifacts in the repo (`eval/results/` or `docs/audit/`), not ephemeral terminal output.
5. Each improvement commit is separated by category with a descriptive message referencing the category (e.g., `fix(type-safety): replace any in document routes with Document type`).
6. No removal of functionality to improve bundle size or query count.

---

## Deliverables Checklist

| Deliverable | Source |
|---|---|
| Forked GitHub repo with clearly labeled improvement branches | Fork of `US-Department-of-the-Treasury/ship` |
| Audit report with baseline measurements for all 7 categories | `AUDIT.md` |
| Improvement documentation per category | `AUDIT.md` + `docs/improvements/` |
| Discovery write-up (3 things learned) | `AUDIT.md` or `docs/discoveries.md` |
| ARCHITECTURE.md | Repo root |
| THREAT_MODEL.md | Repo root |
| USERS.md | Repo root |
| AI-COST-ANALYSIS.md | Repo root |
| SUBMISSION.md | Repo root |
| Observability middleware | `api/src/middleware/observability.ts` |
| Benchmark scripts | `eval/` or `scripts/benchmark/` |
| Additional unit tests | `api/src/__tests__/` |
| Accessibility regression tests | `e2e/accessibility/` |
| Demo video (3-5 min) | Linked from `SUBMISSION.md` |
| Deployed application | Linked from `SUBMISSION.md` |
| Social post | Linked from `SUBMISSION.md` |

---

## References

- ShipShape assignment PDF: `GFA Week 4 - ShipShape.pdf`
- Kickoff PDF: `ShipShape — Kickoff.pdf`
- Target repo: `https://github.com/US-Department-of-the-Treasury/ship`
- Week 3 reference patterns: `Week3/` (ARCHITECTURE.md, THREAT_MODEL.md, USERS.md, AI-COST-ANALYSIS.md, AUDIT.md, SUBMISSION.md, tests/, evals/, observability/)
- Week 3 architecture: `Week3/ARCHITECTURE.md`
- Week 3 threat model: `Week3/THREAT_MODEL.md`
