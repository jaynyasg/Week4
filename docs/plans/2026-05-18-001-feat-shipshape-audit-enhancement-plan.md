---
title: "feat: ShipShape — Full Audit, Enhancement Layer, and Measurable Improvements"
date: 2026-05-18
status: active
origin: docs/brainstorms/shipshape-requirements.md
target_repo: https://github.com/US-Department-of-the-Treasury/ship (fork)
---

# feat: ShipShape — Full Audit, Enhancement Layer, and Measurable Improvements

**Target repo:** Forked `US-Department-of-the-Treasury/ship`. All file paths below are relative to the fork root unless otherwise noted. Paths are forward declarations against the known repo structure; exact locations are verified on fork.

**Plan home:** This document lives in the Week4 planning repo at `docs/plans/`. It is not committed to the Ship fork itself.

**Origin document:** `docs/brainstorms/shipshape-requirements.md`

**Week 3 style references:** `Week3/ARCHITECTURE.md`, `Week3/THREAT_MODEL.md`, `Week3/USERS.md`, `Week3/AI-COST-ANALYSIS.md`, `Week3/agentforge/observability/`, `Week3/tests/`

---

## Problem Frame

Fork a live, production-grade TypeScript monorepo (the US Treasury's Ship project management tool), audit it across 7 measurable categories, implement targeted improvements that hit every PDF-defined improvement target, and layer on a Week-3-style engineering package: ARCHITECTURE.md (before/after in one document), THREAT_MODEL.md, USERS.md, AUDIT.md, AI-COST-ANALYSIS.md, SUBMISSION.md, observability middleware, benchmark scripts, API unit tests, and accessibility regression tests.

**What is not changing:** The unified document model (single `documents` table, `document_type` discriminator), the Yjs CRDT real-time collaboration core, and the Terraform infrastructure definitions are architectural constants. No existing passing Playwright test may be broken.

---

## Scope Boundaries

### In scope
- Fork setup, local environment, baseline test run
- **Codebase orientation phase** — systematic completion of all 24+ questions in the PDF Appendix Codebase Orientation Checklist, produced as `ORIENTATION.md`
- 7-category baseline measurement with committed artifact evidence
- All 7 improvement targets (type safety, bundle size, API performance, DB query efficiency, test coverage, error handling, accessibility)
- ARCHITECTURE.md with before/after sections (written after all improvements land)
- ORIENTATION.md, THREAT_MODEL.md, USERS.md, AI-COST-ANALYSIS.md, SUBMISSION.md, AUDIT.md
- Observability middleware (`api/src/middleware/observability.ts`)
- Health/readiness endpoints (added if not present; untouched if present)
- Benchmark scripts (`eval/` directory) — rerunnable with `--baseline`/`--compare` flags
- API unit tests (`api/src/__tests__/`)
- Accessibility regression tests (`e2e/accessibility/`)
- Deployed fork on a publicly accessible URL

### Protected from changes
- `documents` table schema and migration files
- `api/src/collaboration/` (Yjs + WebSocket core)
- `terraform/` directory
- Any existing Playwright test file that currently passes (flaky-test fixes excepted, with documented RCA)

### Deferred to Follow-Up Work
- Replacing Express with Fastify or NestJS
- Migrating from PostgreSQL to a different data store
- Adding authentication provider changes
- Production APM (Datadog, New Relic) beyond the local observability middleware
- Multi-tenant workspace isolation improvements beyond what the current schema supports

---

## Key Technical Decisions

| Decision | Choice | Rationale |
|---|---|---|
| ARCHITECTURE.md timing | Written after all improvements land; single doc with before/after sections | Ensures accuracy; graders see the full delta in one place |
| Benchmark script format | Structured JSON output with `--baseline`/`--compare` flags | Committed artifacts are the evidence standard; must be reproducible by a grader |
| Observability middleware | Fail-open Express middleware, no external service | Additive, zero dependency, does not affect existing request flow |
| Unit test framework | Vitest (preferred) or Jest — match whatever is already configured in `api/` | Follow existing test conventions; do not introduce a second test runner |
| Accessibility test integration | `axe-playwright` or `pa11y-ci` in a new `e2e/accessibility/` directory | Runs as a separate suite so it never blocks the existing Playwright suite |
| Bundle analysis tool | `rollup-plugin-visualizer` added to `vite.config.ts` | Vite-native; treemap output; no build process changes required beyond plugin add |
| Load test tool | `autocannon` (Node-native, scriptable) | Zero-install via npx; P50/P95/P99 output; outputs JSON for committed artifacts |
| Type safety measurement | `grep` counts + `tsc --strict --noEmit` error count | Reproducible by any reviewer without special tooling |

---

## High-Level Technical Design

*This illustrates the intended sequencing and integration shape. It is directional guidance for review, not implementation specification.*

```
Phase 0: Fork + Setup
  └── verify env, run existing tests, record baseline test count

Phase 0.5: Codebase Orientation (PDF Appendix gate)
  └── ORIENTATION.md
        ├── Phase 1 First Contact: repo overview, data model, request flow
        ├── Phase 2 Deep Dive: real-time collab, TypeScript patterns, testing infra, build/deploy
        └── Phase 3 Synthesis: 3 strongest + 3 weakest + onboarding advice + 10x scaling break point

Phase 1: Baseline Measurement (commit artifacts after each category)
  ├── type-safety-baseline.json    (grep counts, tsc error count)
  ├── bundle-baseline.json         (dist/ sizes, chunk count)
  ├── api-benchmark-baseline.json  (autocannon P50/P95/P99 per endpoint)
  ├── db-query-baseline.md         (EXPLAIN ANALYZE output, query counts)
  ├── test-coverage-baseline.json  (pass/fail/flaky counts, coverage %)
  ├── error-baseline.md            (console errors, unhandled rejections list)
  └── a11y-baseline.json           (Lighthouse scores, axe violation counts)

Phase 2: Enhancement Infrastructure (additive only)
  ├── api/src/middleware/observability.ts   (fail-open request logger)
  ├── api/src/routes/health.ts             (GET /health, GET /ready — if not present)
  ├── eval/benchmark-api.js                (autocannon wrapper, --baseline/--compare)
  ├── eval/benchmark-queries.sql           (EXPLAIN ANALYZE scripts)
  ├── eval/benchmark-bundle.sh             (pnpm build + dist/ size capture)
  ├── api/src/__tests__/                   (Vitest/Jest API unit tests)
  └── e2e/accessibility/                   (axe-playwright or pa11y-ci suite)

Phase 3: Category Improvements (one branch per category)
  ├── fix/type-safety          → web/src/, api/src/, shared/
  ├── fix/bundle-size          → vite.config.ts, web/src/ (lazy imports)
  ├── fix/api-performance      → api/src/routes/, api/src/db/
  ├── fix/db-queries           → api/src/db/ (indexes, batch queries)
  ├── feat/test-coverage       → e2e/ (3 new Playwright tests)
  ├── fix/error-handling       → web/src/ (error boundaries), api/src/ (rejection handlers)
  └── fix/accessibility        → web/src/ (ARIA, contrast, keyboard nav)

Phase 4: Documentation (after all improvements land)
  ├── ARCHITECTURE.md          (before/after for every changed area)
  ├── THREAT_MODEL.md
  ├── USERS.md
  ├── AUDIT.md                 (all 7 categories: baseline + improvement + evidence)
  ├── AI-COST-ANALYSIS.md
  └── SUBMISSION.md

Phase 5: Evidence Commit + Deploy
  ├── eval/results/            (before/after artifacts per category)
  └── deployed fork URL
```

---

## Output Structure

New files and directories added to the Ship fork:

```
eval/
  benchmark-api.js
  benchmark-queries.sql
  benchmark-bundle.sh
  results/
    type-safety-baseline.json
    type-safety-after.json
    bundle-baseline.json
    bundle-after.json
    api-benchmark-baseline.json
    api-benchmark-after.json
    db-query-baseline.md
    db-query-after.md
    test-coverage-baseline.json
    test-coverage-after.json
    error-baseline.md
    a11y-baseline.json
    a11y-after.json

api/src/
  middleware/
    observability.ts
  __tests__/
    (unit test files per route)

e2e/
  accessibility/
    (pa11y-ci or axe-playwright test files)

ORIENTATION.md          ← new, at repo root (PDF Appendix checklist)
ARCHITECTURE.md         ← new, at repo root
THREAT_MODEL.md         ← new, at repo root
USERS.md                ← new, at repo root
AUDIT.md                ← new, at repo root
AI-COST-ANALYSIS.md     ← new, at repo root
SUBMISSION.md           ← new, at repo root
```

---

## Implementation Units

### U1. Fork, Local Environment, and Baseline Test Run

**Goal:** Get the Ship fork running locally, verify all existing tests pass, and document setup steps including anything not in the README.

**Requirements:** Prerequisite for all other units.

**Dependencies:** None.

**Files:**
- Fork at `github.com/<your-username>/ship`
- `SUBMISSION.md` — initial stub with fork URL and setup notes (created here, filled in U23)

**Approach:**
- Fork `US-Department-of-the-Treasury/ship` on GitHub
- Clone locally; follow README setup (pnpm install, Docker for PostgreSQL, env setup)
- Run `pnpm test`; record pass/fail counts and any setup friction
- Document every step not in the README — this feeds ARCHITECTURE.md and AUDIT.md
- Record the existing test count and runtime as the baseline for U7 (test coverage)
- Create branch convention: `audit/baseline`, `fix/<category>`, `feat/<category>`

**Test scenarios:**
- Test expectation: none — setup unit with no behavioral code change

**Verification:** `pnpm test` passes. Local app serves at localhost. Fork URL is accessible on GitHub.

---

### U24. Codebase Orientation (PDF Appendix Gate)

**Goal:** Complete the full PDF Appendix Codebase Orientation Checklist before any audit measurement begins. Produce `ORIENTATION.md` as a committed artifact containing systematic answers to all 24+ checklist questions across the 3 orientation phases. The PDF explicitly states: *"Your orientation notes become part of your final submission."*

**Requirements:** PDF Appendix — Codebase Orientation Checklist (Phase 1, 2, 3). Prerequisite gate for all baseline measurement units (U2–U5).

**Dependencies:** U1

**Files:**
- `ORIENTATION.md` — Ship fork root (committed artifact)

**Approach:**

Read sources, take notes, answer every question. The output is a structured Markdown document with one subsection per checklist item. Be specific — cite file paths and line ranges where claims are grounded.

*Phase 1: First Contact*

**1.1 Repository Overview**
- Clone steps that worked, and any steps not in the README (record verbatim — feeds SUBMISSION.md setup section)
- Read every file in `docs/` (the Ship repo's own docs folder, not Week4's); summarize each file's key architectural decisions in your own words
- Read the `shared/` package: list every exported type, group by purpose (DTOs, enums, utility types), note which are used in `web/`, which in `api/`, which in both
- Create a Mermaid diagram showing how `web/`, `api/`, and `shared/` packages relate (imports, type sharing, build dependencies)

**1.2 Data Model**
- Find database schema (migrations in `api/src/db/migrations/` or seed files in `api/src/db/seed/`); list every table with column names and types
- Document the unified document model: how one `documents` table serves docs, issues, projects, and sprints — which columns are type-specific, which are universal, what goes into `metadata` JSONB
- Document `document_type` discriminator usage: list every query that filters on it (grep `document_type` in `api/src/`), note which queries lack `document_type` filter
- Document relationship handling: parent-child (`parent_id` self-reference), workspace membership, document linking patterns

**1.3 Request Flow**
- Pick one concrete user action (recommended: creating an issue) and trace it end-to-end:
  - React component triggering the action (file path + line range)
  - Network request shape (method, path, body, headers)
  - Middleware chain hit (in order)
  - Route handler (file path + line range)
  - Database query executed (the actual SQL or ORM call)
  - Response shape returned
  - React state update on success
- Document the full middleware chain: list every middleware in order, with its responsibility, in `api/src/`
- Document authentication: how session tokens are validated, what happens to unauthenticated requests (response code, body shape), session token storage mechanism

*Phase 2: Deep Dive*

**2.1 Real-time Collaboration**
- Document how the WebSocket connection is established (handshake URL, auth mechanism, initial sync payload)
- Document how Yjs syncs document state (binary update format, broadcast pattern, room/channel scoping)
- Document concurrent-edit behavior: trace what happens when two users edit the same field — does Yjs converge automatically, is server arbitration required, how does TipTap render the merge
- Document server-side Yjs state persistence: when does it write (on every update, debounced, on disconnect), where does it write (which table column), what is the format (BYTEA snapshot of full Y.Doc)

**2.2 TypeScript Patterns**
- Record TypeScript version (from `package.json` or `tsconfig.json`)
- Record full `tsconfig.json` settings: strict mode on/off, individual strict flags (noImplicitAny, strictNullChecks, etc.)
- Document type-sharing pattern: how `shared/` types reach `web/` and `api/` — direct import, build-time copy, or symlink
- Find and cite 1 example each (with file path + line range) of: generics in use, discriminated union, utility type (`Pick`/`Omit`/`Partial`/`Required`/`Readonly`), type guard function
- Note any TypeScript patterns not previously recognized; research them; record the name, what it does, and one sentence on when to use it. These feed the Discovery section of AUDIT.md

**2.3 Testing Infrastructure**
- Document Playwright test structure: directory layout, naming convention, suite organization, `playwright.config.ts` highlights
- Document Playwright fixtures used (custom fixtures in `e2e/fixtures/` or `test.extend` patterns); list each fixture and what it provides
- Document test database lifecycle: how is the test DB created, seeded, and torn down (per-test, per-file, per-run); is it shared with dev DB or isolated
- Run `pnpm test` and record: total test count, runtime, all-pass status, any flaky tests on 3 consecutive runs

**2.4 Build and Deploy**
- Read `Dockerfile`; document each stage and what it produces (multi-stage build outputs)
- Read `docker-compose.yml`; list every service it starts (web, api, db, etc.), the port mappings, and inter-service dependencies
- Read Terraform configs in `terraform/`; list the cloud resources expected (VPC, ECS/Fargate/Lambda, RDS, S3, IAM roles, etc.) and which cloud provider
- Document CI/CD pipeline: is there a `.github/workflows/` or `.gitlab-ci.yml`; what does it run on push/PR; deploy target

*Phase 3: Synthesis (graded section — answer thoughtfully)*

**3.1 Architecture Assessment**
- **3 strongest architectural decisions in this codebase and why** — name each decision, cite where it's evident in code, explain why it's strong (e.g., unified document model = simple migrations, single API surface, cheap to add new content types)
- **3 weakest points and where improvement should focus** — name each weakness, cite evidence (file path or behavior observed), explain the cost (e.g., no rate limiting on document creation = vulnerable to abuse from authenticated users)
- **Onboarding advice** — write a paragraph: if you had to onboard a new engineer to this codebase today, what would you tell them first? What concept must they understand before reading code? What patterns will surprise them?
- **10x scaling break point** — if Ship had 10x more users (more workspaces, more documents, more concurrent WebSocket connections), what would break first? Why? What's the rough magnitude of work to fix it?

**Execution note:** This is reading-and-writing work, not code-changing work. Take real notes. The graded value of this unit is the *quality and specificity* of the synthesis section (3.1), not just answering the questions.

**Test scenarios:**
- Test expectation: none — documentation unit, no code change

**Verification:** `ORIENTATION.md` at Ship fork root contains all 8 numbered subsections (1.1, 1.2, 1.3, 2.1, 2.2, 2.3, 2.4, 3.1). Every question is answered with specific file paths or evidence (not generic prose). Mermaid diagram for 1.1 renders correctly. Synthesis section (3.1) names specific decisions and weaknesses, not generic ones. **No audit measurement (U2–U5) begins until this unit is complete.**

---

### U2. Type Safety Baseline Measurement

**Goal:** Capture a reproducible baseline count of all type safety violations AND inventory positive TypeScript patterns already in use. The violation count drives the 25% improvement target; the positive pattern inventory feeds AUDIT.md Discovery and validates the U24 orientation findings against measurable data.

**Requirements:** 7-category audit (type safety category) + PDF orientation item 2.2 (TypeScript Patterns inventory).

**Dependencies:** U1, U24

**Files:**
- `eval/results/type-safety-baseline.json` — committed artifact

**Approach:**

*Violation count (baseline for improvement target):*
- Run grep counts for `: any`, `as `, `!.`, `@ts-ignore`, `@ts-expect-error` across `web/src/`, `api/src/`, `shared/` — break down by package and violation type
- Check `tsconfig.json` strict mode setting; if not enabled, run `tsc --strict --noEmit` and record error count
- Identify the 5 most violation-dense files
- The 25% improvement target requires fixing at least `floor(total × 0.25)` violations; compute and record that threshold in the artifact

*Positive pattern inventory (cross-references U24 findings):*
- Count usages of: discriminated unions (look for `type X = A | B` patterns where each variant has a literal discriminator), generics (`<T>` in function/type definitions), utility types (`Pick`, `Omit`, `Partial`, `Required`, `Readonly`, `Record`), type guards (functions returning `x is Y`)
- Record one canonical example of each (file path + line range) — these become evidence for AUDIT.md Discovery section

*Artifact shape:*
```json
{
  "violations": {
    "any": { "total": N, "by_package": { "web": N, "api": N, "shared": N }, "top_files": [...] },
    "as_assertions": { ... },
    "non_null_assertions": { ... },
    "ts_ignore": { ... }
  },
  "strict_mode": { "enabled": bool, "tsc_strict_errors": N },
  "improvement_target": { "fix_at_least": floor(total * 0.25) },
  "positive_patterns": {
    "generics": { "count": N, "example": "path/to/file.ts:42" },
    "discriminated_unions": { "count": N, "example": "..." },
    "utility_types": { "count": N, "examples": ["Pick: ...", "Omit: ...", "Partial: ..."] },
    "type_guards": { "count": N, "example": "..." }
  }
}
```

**Test scenarios:**
- Test expectation: none — measurement-only unit

**Verification:** `eval/results/type-safety-baseline.json` exists, is committed, contains per-package violation counts, top-5 file list, AND positive pattern inventory with concrete examples.

---

### U3. Bundle Size Baseline Measurement

**Goal:** Capture a reproducible production bundle size baseline before any optimization.

**Requirements:** 7-category audit (bundle size category).

**Dependencies:** U1

**Files:**
- `eval/results/bundle-baseline.json`
- `vite.config.ts` — add `rollup-plugin-visualizer` (dev dependency only; does not affect production build output)

**Approach:**
- Add `rollup-plugin-visualizer` as a dev dependency with `emitFile: true` to generate a `stats.html` treemap
- Run `pnpm build` in `web/`; record total `dist/` size in KB, chunk count, largest chunk name + size
- Identify top 3 largest dependencies from the treemap
- Cross-reference `package.json` dependencies against actual imports with `depcheck` or manual grep to flag unused dependencies
- Commit `eval/results/bundle-baseline.json` with all metrics

**Test scenarios:**
- `pnpm build` succeeds after adding visualizer plugin
- `stats.html` treemap is generated and readable
- Test expectation: none beyond build success — measurement unit

**Verification:** `eval/results/bundle-baseline.json` committed with total KB, chunk count, top 3 deps, unused dep list.

---

### U4. API Response Time and Database Query Baseline

**Goal:** Capture response time and query efficiency baselines under realistic data load.

**Requirements:** 7-category audit (API response time + database query efficiency categories).

**Dependencies:** U1

**Files:**
- `eval/benchmark-api.js`
- `eval/benchmark-queries.sql`
- `eval/results/api-benchmark-baseline.json`
- `eval/results/db-query-baseline.md`

**Approach:**
- Seed the database with realistic volume: 500+ documents, 100+ issues, 20+ users, 10+ sprints (via `pnpm db:seed` or a custom seed script — document which)
- Identify the 5 most-used API endpoints by tracing frontend network requests during common flows (document load, issue list, sprint board, search, auth)
- Write `eval/benchmark-api.js` as an `autocannon` wrapper: accepts `--baseline` flag (saves JSON to `eval/results/`), `--compare` flag (diffs against saved baseline), runs each endpoint at 10/25/50 concurrent connections, outputs P50/P95/P99
- Enable PostgreSQL query logging via Docker env (`POSTGRES_LOG_STATEMENT=all`)
- Execute 5 user flows; count total queries per flow; run `EXPLAIN ANALYZE` on the slowest query per flow
- Write `eval/benchmark-queries.sql` containing the EXPLAIN ANALYZE queries, runnable via `psql`
- Flag N+1 patterns: list-view queries where one query per item is triggered instead of a batch
- Commit baseline artifacts

**Execution note:** Seed data must be committed or scripted — do not rely on ephemeral local state. If `pnpm db:seed` exists, use it; if not, write a minimal seed script and commit it to `eval/seed.ts` or `eval/seed.js`.

**Test scenarios:**
- `autocannon` benchmark completes for all 5 endpoints at all 3 concurrency levels
- `eval/results/api-benchmark-baseline.json` contains P50/P95/P99 for each endpoint
- Test expectation: none beyond script execution — measurement unit

**Verification:** Both baseline artifacts committed. `node eval/benchmark-api.js --baseline` runs without error against seeded local database.

---

### U5. Test Coverage, Error Handling, and Accessibility Baselines

**Goal:** Capture baselines for the three remaining audit categories: test coverage, runtime error handling, and accessibility.

**Requirements:** 7-category audit (categories 5, 6, 7).

**Dependencies:** U1

**Files:**
- `eval/results/test-coverage-baseline.json`
- `eval/results/error-baseline.md`
- `eval/results/a11y-baseline.json`

**Approach:**

*Test coverage:*
- Run `pnpm test` three times; record pass/fail/flaky counts and total runtime
- Read all `e2e/` test files; catalog covered and uncovered critical flows (document CRUD, real-time sync, auth, sprint management)
- Configure `c8` or `@vitest/coverage-v8` for line/branch coverage if not present; run and record per-package %
- Commit `eval/results/test-coverage-baseline.json`

*Runtime error handling:*
- Open browser DevTools; run through normal usage; count console errors and warnings
- Test: network disconnect mid-collaborative-edit, reconnect; malformed inputs (empty, 10k chars, script tags); concurrent edit by two users; 3G throttle
- Check server logs for unhandled promise rejections
- Commit `eval/results/error-baseline.md` with findings table

*Accessibility:*
- Run Lighthouse accessibility audit on each major page; record scores
- Run `axe-core` via browser extension or `pa11y` CLI; categorize violations by severity
- Test keyboard navigation completeness (Tab/Enter/Escape/arrows to every interactive element)
- Record color contrast failures
- Commit `eval/results/a11y-baseline.json`

**Test scenarios:**
- Test expectation: none — measurement-only unit

**Verification:** All three baseline artifacts committed. `eval/results/` contains 7 total baseline files after U2–U5.

---

### U6. AUDIT.md — Baseline Report (Phase 1 Gate)

**Goal:** Write the formal audit report with all 7 category baselines filled in, plus the synthesis sections required by the PDF (Discovery + Architecture Assessment), as required by the Phase 1 PDF gate.

**Requirements:** All 7 audit categories must be complete before this unit. PDF Discovery Requirement (3 things learned) + PDF Appendix Phase 3 Synthesis (Architecture Assessment).

**Dependencies:** U2, U3, U4, U5, U24

**Files:**
- `AUDIT.md` — committed to Ship fork root

**Approach:**
- Write `AUDIT.md` following the Week 3 `AUDIT.md` and `AUDIT_V2.md` style (see `Week3/AUDIT.md`)
- Include per-category sections (1–7): measurement methodology, tools used, baseline numbers (from committed artifacts), identified weaknesses ranked by severity
- Include a **Discovery** section per PDF Discovery Requirement: 3 specific things learned during orientation/audit (TypeScript feature, architectural pattern, library, or engineering practice not previously known) — each with: (1) name, (2) codebase file path + line range, (3) what it does and why it matters, (4) how you would apply this knowledge in a future project. Source these from U24 orientation findings (especially 2.2 TypeScript Patterns) and U2 positive pattern inventory
- Include an **Architecture Assessment** section per PDF Appendix Phase 3 Synthesis (this is graded — answer with specificity, not generics):
  - **3 strongest architectural decisions** — each with name, evidence in code (file path), and a paragraph explaining why it's strong. Source from U24 §3.1
  - **3 weakest points** — each with name, evidence (file path or observed behavior), cost or risk, and where improvement should focus. Source from U24 §3.1 and the audit baseline findings
  - **Onboarding advice** — what you would tell a new engineer on day one about this codebase
  - **10x scaling break point** — what would break first at 10x current load, why, and rough magnitude to fix
- Leave improvement subsections per category as stubs at this stage (filled in U22 after fixes land)
- Mark clearly: "Phase 1 Gate — baseline only, no fixes yet"

**Patterns to follow:** Week 3 `AUDIT.md` and `AUDIT_V2.md` for section structure and discovery write-up style.

**Test scenarios:**
- Test expectation: none — documentation unit

**Verification:** `AUDIT.md` at repo root contains all 7 category tables with real baseline numbers, a Discovery section with 3 entries (each citing a file path and explaining future application), and an Architecture Assessment section answering all 4 PDF synthesis questions with specific evidence. No improvement claims yet.

---

### U7. Observability Middleware and Health Endpoints

**Goal:** Add fail-open request observability and health/readiness endpoints to the Express API.

**Requirements:** Enhancement package — observability layer.

**Dependencies:** U1

**Files:**
- `api/src/middleware/observability.ts`
- `api/src/routes/health.ts` (only if `/health` and `/ready` do not already exist)
- `api/src/__tests__/observability.test.ts`

**Approach:**

*Middleware (`api/src/middleware/observability.ts`):*
- Express middleware that records `method`, `path`, `status_code`, `duration_ms` as structured JSON to stdout on every request completion
- Catches and records `error_type`, `error_message` (no stack trace) on unhandled errors
- Emits event types: `request_completed`, `request_errored`, `server_started`
- Deliberately omits: `Authorization` header value, `Cookie` header value, request body, response body
- Fail-open: wraps the recording in try/catch so middleware errors never propagate to the request
- Attached in `api/src/app.ts` (or equivalent entry point) as the first middleware after body-parser — no changes to any other existing middleware

*Health routes (`api/src/routes/health.ts`):*
- Verify whether `GET /health` and `GET /ready` already exist in `api/src/`; if they do, make no changes
- If not present: add `GET /health → { status: "ok" }` and `GET /ready → { status: "ready" | "degraded", db_connected: boolean }` — check DB connection with a lightweight ping query
- Register health routes before the observability middleware so they do not appear in structured logs (or register them after but filter them from logs by path prefix)

**Patterns to follow:** Week 3 `agentforge/observability/events.py` for event structure; Week 3 `agentforge/http/app.py` for how health routes are registered.

**Test scenarios:**
- Happy path: middleware records `request_completed` event with correct method/path/status/duration for a normal GET request
- Error path: when a downstream handler throws, middleware records `request_errored` event without leaking the stack trace
- Sensitive data exclusion: `Authorization` header value does not appear in any logged output
- Fail-open: when the middleware's own logging code throws, the original request still completes with the correct status
- Health route: `GET /health` returns `{ status: "ok" }` with 200 (only if route is new)
- Readiness route: `GET /ready` returns `db_connected: true` when DB is reachable, `db_connected: false` and `status: "degraded"` when DB ping fails (only if route is new)

**Verification:** Unit tests pass. Middleware is attached and logs structured events during `pnpm dev`. No existing test regressions.

---

### U8. Benchmark and Evaluation Scripts

**Goal:** Build the rerunnable benchmark scripts that capture before/after evidence for API performance, bundle size, and database queries.

**Requirements:** Enhancement package — evaluation harness.

**Dependencies:** U4 (benchmark-api.js started there; finalize and add --compare here), U3 (bundle baseline established there)

**Files:**
- `eval/benchmark-api.js` (finalized from U4 draft)
- `eval/benchmark-bundle.sh`
- `eval/benchmark-queries.sql` (finalized from U4 draft)
- `eval/README.md` — describes each script, flags, and expected output format

**Approach:**

*`eval/benchmark-api.js`:*
- `--baseline`: runs autocannon against all 5 endpoints at 10/25/50 concurrency; saves JSON to `eval/results/api-benchmark-<timestamp>.json` and symlinks/copies to `api-benchmark-latest.json`
- `--compare <file>`: loads the named baseline file and the current run; prints a diff table showing P50/P95/P99 delta per endpoint, highlights regressions (>5% slower) and improvements (>10% faster)
- `--endpoint <path>`: run a single endpoint only
- Accepts `BASE_URL` env var; defaults to `http://localhost:3000`

*`eval/benchmark-bundle.sh`:*
- Runs `pnpm build` in `web/`
- Captures `dist/` total size in KB, chunk count, largest file
- Outputs structured JSON to stdout; saves to `eval/results/bundle-<timestamp>.json`
- Accepts `--compare <file>` flag; prints size delta

*`eval/benchmark-queries.sql`:*
- Contains named EXPLAIN ANALYZE blocks for each of the 5 user flows
- Each block is prefixed with a comment naming the flow and the expected index use
- Runnable via `psql $DATABASE_URL -f eval/benchmark-queries.sql`

*`eval/README.md`:*
- Quick-start for each script
- Prerequisites (running local app, seeded database)
- How to capture baseline, how to compare after changes
- Expected output format

**Test scenarios:**
- `node eval/benchmark-api.js --baseline` completes without error against a running local app
- `node eval/benchmark-api.js --compare eval/results/api-benchmark-baseline.json` prints a diff table
- `bash eval/benchmark-bundle.sh` outputs valid JSON with size fields
- Test expectation: these are utility scripts; test them by running them, not by unit-testing internals

**Verification:** All three scripts run without error. `eval/README.md` present. Scripts are referenced from AUDIT.md methodology section.

---

### U9. API Unit Tests

**Goal:** Add unit tests for critical API routes — coverage the E2E suite cannot reliably catch at unit speed.

**Requirements:** Enhancement package — API unit tests.

**Dependencies:** U1 (verify existing test runner: Vitest or Jest), U7 (observability middleware is testable here)

**Files:**
- `api/src/__tests__/routes/documents.test.ts` — document CRUD route unit tests
- `api/src/__tests__/routes/auth.test.ts` — authentication middleware and route tests
- `api/src/__tests__/middleware/observability.test.ts` — observability middleware (moved/shared with U7)
- `api/src/__tests__/db/queries.test.ts` — critical database helper unit tests (if query helpers exist as standalone functions)

**Approach:**
- Use `supertest` for HTTP-layer tests; mock the database layer with test doubles (no live DB required for unit tests)
- Match the existing test runner (verify by reading `api/package.json` `scripts.test` and `devDependencies`)
- Each test file imports a narrowly scoped unit; no full-app bootstrap required
- Each test includes a comment on the first line naming the regression it prevents

**Patterns to follow:** Existing test structure in `api/` (if any); Week 3 `tests/agentforge/` for comment-per-test discipline.

**Test scenarios:**

*Document routes:*
- `GET /documents` returns 200 with array when authenticated
- `GET /documents` returns 401 when unauthenticated
- `POST /documents` with valid body returns 201 and created document
- `POST /documents` with missing required fields returns 400 with descriptive error
- `DELETE /documents/:id` with a non-existent ID returns 404
- `DELETE /documents/:id` owned by another user returns 403

*Auth middleware:*
- Valid session token passes through to handler
- Expired token returns 401
- Missing token returns 401
- Malformed token (not a valid JWT shape) returns 401 without crashing

*Observability middleware:*
- (see U7 test scenarios)

**Verification:** All new unit tests pass. No existing tests broken. Test file count in `api/src/__tests__/` is ≥ 3.

---

### U10. Accessibility Regression Test Suite

**Goal:** Add an automated accessibility regression suite that can be re-run to detect regressions after future changes.

**Requirements:** Enhancement package — accessibility regression tests.

**Dependencies:** U1, U5 (baseline a11y violations cataloged)

**Files:**
- `e2e/accessibility/a11y.spec.ts` — axe-playwright tests for major pages
- `package.json` — add `test:a11y` script (does not alter existing `test` script)

**Approach:**
- Use `@axe-core/playwright` (if Playwright is already the test framework — verify in `e2e/package.json`)
- Cover the 3 most important pages identified in U5: dashboard/main page, document editor, issue list
- Each test: navigate to page, run `checkA11y()`, assert zero Critical or Serious violations
- Tests run against a locally served app (same setup as existing Playwright tests)
- Register as a separate `pnpm test:a11y` script so they never block the main `pnpm test` run
- The baseline violation count from U5 serves as the before-state; after U20 (accessibility fixes), re-running this suite should pass

**Patterns to follow:** Existing `e2e/` Playwright test structure and fixtures.

**Test scenarios:**
- Dashboard page has zero Critical or Serious axe violations after fixes (U20) land
- Document editor page has zero Critical or Serious axe violations after fixes land
- Issue list page has zero Critical or Serious axe violations after fixes land
- Each test must be runnable in isolation (`--grep`)

**Verification:** `pnpm test:a11y` passes after U20 (accessibility fixes) land. Suite produces JSON output that can be committed as `eval/results/a11y-after.json`.

---

### U11. Type Safety Fixes

**Goal:** Eliminate ≥25% of type safety violations identified in U2, using genuine type narrowing and meaningful types.

**Requirements:** Type safety improvement target (25% reduction; all tests still pass).

**Dependencies:** U2 (baseline count and threshold computed), U6 (AUDIT.md baseline gate complete)

**Files:**
- Files in `web/src/`, `api/src/`, `shared/` with the highest violation density (identified in U2)
- `eval/results/type-safety-after.json`

**Approach:**
- Work through the top-5 violation-dense files first (highest ROI)
- Replace `any` with specific types using: discriminated unions for `document_type` variants, utility types (`Pick`, `Omit`, `Partial`) for API payloads, `unknown` + type guard functions for external/parsed data
- Remove `as` casts where the actual type is knowable; narrow with `instanceof` or discriminant checks where runtime behavior requires it
- Remove `!` non-null assertions where optional chaining (`?.`) or an explicit null check is sufficient
- Each fix must compile cleanly: run `tsc --noEmit` after each file change
- After all fixes: re-run grep counts and `tsc --strict --noEmit`; save to `eval/results/type-safety-after.json`
- Verify: improvement target is `(baseline_count - after_count) / baseline_count ≥ 0.25`

**Execution note:** Fix one file at a time; run `pnpm test` after each file to catch regressions early rather than batch-fixing then debugging.

**Test scenarios:**
- All existing Playwright tests pass after fixes
- `tsc --noEmit` reports no new errors introduced by the fixes
- The `eval/results/type-safety-after.json` artifact shows ≥25% violation reduction from baseline

**Verification:** `eval/results/type-safety-after.json` committed. `pnpm test` green. Grep count delta meets target.

---

### U12. Bundle Size Reduction

**Goal:** Reduce total production bundle size by ≥15%, or reduce initial page load bundle by ≥20% via code splitting.

**Requirements:** Bundle size improvement target.

**Dependencies:** U3 (baseline measured), U6 (gate complete)

**Files:**
- `web/vite.config.ts` — code splitting configuration (lazy routes)
- `web/src/` — lazy import conversions for heavy routes (editor, sprint board)
- `web/package.json` — remove confirmed-unused dependencies
- `eval/results/bundle-after.json`

**Approach:**
- Identify the highest-impact action from the U3 baseline: unused dependency removal OR code splitting (choose based on treemap data)
- **Code splitting path:** convert heavy route components to `React.lazy()` + `Suspense` — the editor and sprint board are the most likely candidates given their dependency weight; add a loading fallback UI
- **Unused dependency path:** for each flagged unused dependency in U3, verify with a manual grep for imports before removing from `package.json`; do not remove a dependency that is imported anywhere
- After changes: run `pnpm build`, capture new `dist/` sizes via `bash eval/benchmark-bundle.sh`, save to `eval/results/bundle-after.json`
- Confirm: no functionality removed; all existing tests pass

**Test scenarios:**
- `pnpm build` succeeds; no new build errors
- All existing Playwright tests pass (lazy-loaded routes still load correctly)
- `eval/results/bundle-after.json` shows ≥15% total size reduction or ≥20% initial chunk reduction
- Lazy-loaded routes render correctly in the browser (manual smoke test)

**Verification:** Before/after bundle JSON artifacts committed. `pnpm test` green. Improvement target met.

---

### U13. API Response Time Improvements

**Goal:** Achieve ≥20% P95 improvement on at least 2 of the 5 benchmarked endpoints.

**Requirements:** API response time improvement target.

**Dependencies:** U4 (baseline measured), U6 (gate complete)

**Files:**
- `api/src/routes/` — targeted route-level fixes
- `api/src/db/` — query batching or index additions where root cause is DB-layer
- `eval/results/api-benchmark-after.json`

**Approach:**
- Read the U4 baseline to identify the 2 slowest endpoints and their hypothesized root causes
- Common root causes in Express + PostgreSQL: unbatched ORM queries in list endpoints, synchronous operations blocking the event loop, missing SELECT field projections (fetching all columns when only 3 are needed), missing response caching for read-heavy static-ish data
- For each targeted endpoint: apply the fix, re-run `node eval/benchmark-api.js --compare eval/results/api-benchmark-baseline.json`; iterate until the target is met
- Document root cause and fix for each endpoint — this feeds AUDIT.md and ARCHITECTURE.md
- Commit `eval/results/api-benchmark-after.json`

**Execution note:** Run benchmarks under identical conditions (same seed data, same machine, same concurrency level) as the baseline.

**Test scenarios:**
- All existing Playwright tests pass after route changes
- `eval/results/api-benchmark-after.json` shows ≥20% P95 improvement on ≥2 endpoints
- No P95 regressions on the other 3 endpoints (within 5% tolerance)

**Verification:** Before/after benchmark JSON committed. `pnpm test` green. Root cause documented.

---

### U14. Database Query Efficiency Improvements

**Goal:** Achieve ≥20% query count reduction on at least one user flow, or ≥50% improvement on the slowest query.

**Requirements:** Database query efficiency improvement target.

**Dependencies:** U4 (baseline EXPLAIN ANALYZE + N+1 patterns), U6 (gate complete)

**Files:**
- Migration file in `api/src/db/migrations/` — index addition (if chosen approach)
- `api/src/db/` — query batching refactor (if N+1 approach)
- `eval/results/db-query-after.md`

**Approach:**
- Choose the highest-impact improvement from U4 findings:
  - **Index path:** add a composite index on `(document_type, workspace_id)` or the most-filtered column combination; write as a new migration; run `EXPLAIN ANALYZE` before/after to confirm index is used
  - **N+1 path:** find the list endpoint with the worst N+1 pattern; refactor to a single `IN (...)` batch query; verify query count drops in the per-flow log
- The index approach is lower-risk (no application code change, purely additive)
- The N+1 batch refactor is higher-impact but requires care with the unified document model — test that `document_type` filtering still works correctly after batching

**Note on schema constraint:** Adding an index is NOT a schema change to protected tables — it is a performance-only addition. Modifying the `documents` table structure (columns, constraints) is out of scope. Index additions are explicitly in scope.

**Test scenarios:**
- Migration runs cleanly with `pnpm db:migrate`
- `EXPLAIN ANALYZE` output in `eval/results/db-query-after.md` shows index used for the targeted query
- Improvement target met (≥20% query count reduction or ≥50% slowest query improvement)
- All existing Playwright tests pass after migration

**Verification:** Before/after EXPLAIN ANALYZE output committed. Migration file present. `pnpm test` green.

---

### U15. Test Coverage Additions

**Goal:** Add 3 meaningful new Playwright tests for previously untested critical paths.

**Requirements:** Test coverage improvement target (3 new tests for untested flows, or 3 flaky test fixes with RCA).

**Dependencies:** U5 (uncovered flows identified), U6 (gate complete)

**Files:**
- New test files in `e2e/` — one per covered flow, or additions to existing spec files (named clearly)

**Approach:**
- Select the 3 highest-risk uncovered flows from U5 (prioritize flows where a bug would cause data loss or security issues over cosmetic flows)
- Likely candidates: document deletion with confirmation (data loss risk), real-time sync conflict resolution (correctness risk), sprint completion with open issues (state transition risk)
- Each test: write a clear description, use existing `e2e/` fixtures and patterns, include a comment on the first line naming the regression it prevents
- If 3+ flaky tests were identified in U5: fix those instead — document root cause (timing assumptions, missing `await`, race conditions) and the deterministic replacement

**Patterns to follow:** Existing `e2e/` test structure, page object patterns, and fixture usage.

**Test scenarios:**
Each new test IS the test scenario. Three examples (actual choices driven by U5 findings):
- **Document deletion:** navigate to a document, delete it, confirm it no longer appears in the list and navigating to its URL returns 404 — prevents regression where soft-deletes accidentally show deleted content
- **Real-time sync:** open same document in two browser contexts, edit in one, assert the other receives the update within 2 seconds — prevents regression where Yjs sync silently drops edits
- **Sprint board drag:** drag an issue from "In Progress" to "Done", assert its status persists after page reload — prevents regression where drag-drop state is lost on refresh

**Verification:** `pnpm test` passes with new tests included. New tests pass on 3 consecutive runs (not flaky). Test file count delta visible in `eval/results/test-coverage-after.json`.

---

### U16. Error Handling Fixes

**Goal:** Fix 3 error handling gaps; at least one must address a real user-facing data loss or confusion scenario.

**Requirements:** Runtime error handling improvement target.

**Dependencies:** U5 (error baseline and silent failures cataloged), U6 (gate complete)

**Files:**
- `web/src/` — React error boundaries, loading states, network failure recovery UI
- `api/src/` — unhandled promise rejection handlers, malformed input validation
- `eval/results/error-after.md`

**Approach:**
- Select the 3 highest-severity gaps from U5:
  - **Data loss priority:** network disconnect during collaborative edit that causes silent data loss — add a reconnection recovery banner and verify Yjs reconnect preserves unsaved state
  - **User confusion priority:** missing error boundary around the document editor — unhandled render errors currently crash the entire app; wrap with `React.ErrorBoundary` that shows a recovery UI
  - **Server-side priority:** unhandled promise rejection in a route handler — add a `process.on('unhandledRejection')` handler and/or Express async error middleware that converts unhandled rejections to 500 responses with structured JSON
- Each fix requires: reproduction steps, before behavior description, after behavior description, screenshot or test evidence
- Add reproduction steps to `eval/results/error-after.md`

**Test scenarios:**
- Error boundary: mount a component that throws during render; assert the error boundary catches it and renders a recovery message instead of crashing the app
- Malformed input: submit a `POST /documents` request with a body exceeding the size limit; assert a 413 response with descriptive message (not a crash or silent 500)
- Network recovery: (manual test with DevTools network throttle) disconnect during a collaborative edit, reconnect — assert document content is preserved and a status indicator shows "reconnecting" then "connected"

**Verification:** `eval/results/error-after.md` with before/after for each fix. Console error count (manual recheck) lower than baseline. `pnpm test` green.

---

### U17. Accessibility Fixes

**Goal:** Achieve +10 Lighthouse accessibility score on the lowest-scoring page, or fix all Critical/Serious violations on the top 3 pages.

**Requirements:** Accessibility improvement target.

**Dependencies:** U5 (baseline violations cataloged), U10 (accessibility regression tests ready to validate fixes), U6 (gate complete)

**Files:**
- `web/src/` — ARIA labels, roles, contrast fixes, keyboard navigation fixes
- `eval/results/a11y-after.json`

**Approach:**
- Address violations in priority order: Critical → Serious → Moderate
- Common fixes for React apps: add `aria-label` to icon-only buttons, add `role="status"` to live regions, fix color contrast by updating Tailwind color tokens (if the design system allows), add visible focus indicators (`outline` styles), ensure all form inputs have associated `<label>` elements
- Avoid touching TipTap editor internals for keyboard navigation — the editor has its own accessibility model; focus on the surrounding UI chrome
- After fixes: re-run Lighthouse on the 3 target pages; run `pnpm test:a11y` (U10 suite)
- Save Lighthouse JSON exports and axe scan output to `eval/results/a11y-after.json`

**Test scenarios:**
- `pnpm test:a11y` passes (zero Critical/Serious violations on the 3 target pages)
- Lighthouse accessibility score on the lowest-scoring page is ≥ baseline + 10
- Keyboard navigation: Tab key reaches every interactive element in the header navigation and issue list (manual verification)
- Icon-only buttons (e.g., "close", "add", "delete") have an `aria-label` that NVDA/VoiceOver reads correctly

**Verification:** Before/after Lighthouse JSON artifacts committed. `pnpm test:a11y` passes. `pnpm test` green.

---

### U18. THREAT_MODEL.md

**Goal:** Write a security-oriented threat model for the Ship application, grounded in OWASP Web Top 10 2021.

**Requirements:** Enhancement package — threat model.

**Dependencies:** U1 (codebase orientation complete), U16 (error handling gaps are known)

**Files:**
- `THREAT_MODEL.md` — Ship fork root

**Approach:**
Write using Week 3 `THREAT_MODEL.md` as a style reference. Sections:

1. **Summary** (~400 words): highest-risk surfaces in a TypeScript/React/Express/PostgreSQL stack — rich-text editor XSS, ORM injection surface, WebSocket auth, session management, Terraform state exposure
2. **Assets:** user documents, sprint/project metadata, session tokens, WebSocket connections, PostgreSQL credentials, Terraform state
3. **Trust boundaries:** unauthenticated public → authenticated user → workspace admin; WebSocket channel; PostgreSQL network access; CI/CD secrets
4. **Attack surface map:** auth endpoints, document CRUD routes, WebSocket message handling, TipTap rich-text output (XSS surface), `document_type` discriminator (injection surface if unsanitized), Terraform IAM scope, Docker build context — each row with OWASP reference and expected control
5. **Existing defenses:** auth middleware, ORM parameterization (verify in `api/src/db/`), Docker multi-stage build, `pnpm` lockfile
6. **Residual risks:** TipTap HTML sanitization scope, WebSocket message validation completeness, Terraform state file exposure, rate limiting absence
7. **Framework references:** OWASP Web Top 10 2021 (A1 Broken Access Control, A3 Injection, A5 Security Misconfiguration, A7 XSS), NIST SSDF

**Test scenarios:**
- Test expectation: none — documentation unit

**Verification:** `THREAT_MODEL.md` at fork root. Covers all 7 sections. OWASP references present. Existing defenses verified against actual code (not assumed).

---

### U19. USERS.md

**Goal:** Document the Ship application's user types, capabilities, and access patterns.

**Requirements:** Enhancement package — users doc.

**Dependencies:** U1 (codebase orientation: read auth and RBAC code)

**Files:**
- `USERS.md` — Ship fork root

**Approach:**
Write using Week 3 `USERS.md` as a style reference. Sections:

1. **Primary users:** Workspace Admin (configuration, user management, billing), Member (document creation, issue tracking, sprint participation), Guest (read-only, if implemented — verify in codebase)
2. **User flows by role:** per-role capability matrix (can create documents, can delete documents, can manage sprints, can invite members, etc.)
3. **Access groups:** how workspace-level access is enforced in `api/src/`
4. **Non-users:** unauthenticated internet crawlers, automated scanners, third-party app integrations (if not present)
5. **Agent roles** (if applicable): CI bot, seed scripts, migration runner — their capabilities and constraints

**Test scenarios:**
- Test expectation: none — documentation unit

**Verification:** `USERS.md` at fork root. Role capability matrix present. Access enforcement claims verified against actual auth/middleware code.

---

### U20. AI-COST-ANALYSIS.md and SUBMISSION.md

**Goal:** Document AI tool usage and costs during development; finalize the submission control document.

**Requirements:** Enhancement package + PDF submission requirement.

**Dependencies:** All previous units (AI usage spans the whole sprint)

**Files:**
- `AI-COST-ANALYSIS.md` — Ship fork root
- `SUBMISSION.md` — Ship fork root (finalized from U1 stub)

**Approach:**

*AI-COST-ANALYSIS.md:*
- Write using Week 3 `AI-COST-ANALYSIS.md` as a style reference
- Track: model used (Claude Sonnet 4.6), tasks delegated (codebase orientation, audit analysis, doc generation, fix implementation, benchmark script writing), estimated token/cost per category
- Reflection: where AI assistance had highest leverage (doc generation, benchmark script scaffolding, type annotation suggestions) vs. where manual work dominated (reading Playwright output, running benchmarks, verifying fixes in browser)
- Comparison: development time with AI vs. estimated without

*SUBMISSION.md:*
- Canonical links: forked repo URL, deployed application URL
- Deliverables checklist (all 7 PDF items + enhancement package)
- Evidence inventory: before/after artifact paths per category
- Final blockers section (demo video link, social post link)

**Test scenarios:**
- Test expectation: none — documentation unit

**Verification:** Both files at fork root. SUBMISSION.md checklist has no unchecked required items. AI cost estimates are present with reasoning.

---

### U21. ARCHITECTURE.md — Before/After (Written After All Improvements)

**Goal:** Write a comprehensive ARCHITECTURE.md containing both the original system state and every improvement made, in a single document with clearly labeled before/after sections.

**Requirements:** Enhancement package — architecture doc with before/after state.

**Dependencies:** U11–U17 (all improvements complete), U7 (observability added), U8 (benchmark scripts added), U9 (unit tests added), U10 (a11y tests added)

**Files:**
- `ARCHITECTURE.md` — Ship fork root

**Approach:**
Write using Week 3 `ARCHITECTURE.md` as a style reference. All 13 sections:

1. **Executive Summary** (~400 words): Ship's purpose, monorepo structure, core architectural decisions, and the scope of this Week 4 enhancement project
2. **System Diagram** (Mermaid `flowchart TB`): Browser → WebSocket/HTTP → Express API → PostgreSQL; enhancement overlays (observability middleware, eval harness, health routes, test suites)
3. **Component and Request Flow**: two traced flows — document creation (HTTP path) and collaborative edit (WebSocket path) — with middleware chain shown
4. **Data Model**: unified document model deep-dive — `documents` table, `document_type` discriminator, parent-child relationships, workspace scoping. **Before:** original index set. **After:** new index added (U14), rationale and EXPLAIN ANALYZE delta
5. **Real-Time Collaboration**: Yjs CRDT sync lifecycle, WebSocket connection management, server-side state persistence. **Before:** error recovery behavior. **After:** error boundary and reconnection improvements (U16)
6. **Testing Flow**: Playwright E2E (existing) → API unit tests (added U9) → accessibility suite (added U10) → benchmark scripts (added U8). How to run each tier; what each proves
7. **Evaluation and Benchmarking**: `eval/` scripts, `--baseline`/`--compare` usage, how artifacts in `eval/results/` prove before/after state
8. **Observability**: observability middleware design (U7) — what is logged, what is deliberately excluded, fail-open design, health/readiness endpoints
9. **New Features and Enhancements**: each of the 7 improvement areas with before state, root cause diagnosis, change made, and after state evidence
10. **Security**: trust boundaries, OWASP mapping, existing defenses (verified), residual risks (from THREAT_MODEL.md), what U16's error handling fixes addressed
11. **Database and Query Design**: unified document model index strategy — **before** (original indexes) and **after** (new index from U14); EXPLAIN ANALYZE comparison; N+1 patterns found and how addressed; safe vs. degrading query patterns at scale
12. **Tools Used — and Why** (full table with Tool / Role / Why chosen / Alternatives considered / Why not — covering React/Vite, TipTap/Yjs, Express, PostgreSQL, Playwright, pnpm, autocannon, axe-core/pa11y, rollup-plugin-visualizer, Vitest/Jest, custom observability middleware)
13. **Known Tradeoffs**: unified document model at scale, Yjs memory footprint, deterministic accessibility fixes vs. TipTap editor internals, observability middleware scope

**Test scenarios:**
- Test expectation: none — documentation unit

**Verification:** `ARCHITECTURE.md` at fork root with all 13 sections. Before/after sections present in Data Model, Real-Time Collaboration, New Features, Database. Mermaid diagram renders correctly. Tools table has at least 10 entries with "Why not" column filled.

---

### U22. AUDIT.md — Final Update with Improvements and Evidence

**Goal:** Update AUDIT.md (written at U6 baseline) with improvement results, root cause analysis, and evidence artifact references for all 7 categories.

**Requirements:** All 7 improvement targets must be met before this unit.

**Dependencies:** U11–U17 (all improvements), U8 (evidence artifacts)

**Files:**
- `AUDIT.md` — update in place

**Approach:**
- Add an "Improvements" subsection to each of the 7 category sections
- Each subsection: root cause analysis, change made, before measurement (link to `eval/results/*-baseline.*`), after measurement (link to `eval/results/*-after.*`), proof of reproducibility
- Add the 3 discovery write-ups (if not already in U6): name, codebase location, what it does, how you'd apply it in a future project
- Confirm the Phase 1 gate section is still marked clearly; add a Phase 2 completion marker

**Test scenarios:**
- Test expectation: none — documentation unit

**Verification:** `AUDIT.md` has improvement subsections for all 7 categories. Every improvement cites an `eval/results/` artifact. Discovery section has 3 entries with codebase file references.

---

### U23. Deploy and Final Submission

**Goal:** Deploy the improved Ship fork to a publicly accessible URL; finalize SUBMISSION.md.

**Requirements:** PDF submission — deployed application.

**Dependencies:** All previous units.

**Files:**
- `SUBMISSION.md` — finalized
- `Dockerfile` / `docker-compose.yml` — verify or update for the fork (do not change structure, only env config)

**Approach:**
- Deploy to Render, Railway, Fly.io, or Vercel/Neon combination — choose based on easiest PostgreSQL + WebSocket support (Render recommended: matches Week 3 patterns and supports persistent disks for uploads)
- Configure environment variables for the deployment (database URL, session secret, any API keys)
- Verify: deployed app serves, auth works, document creation works, real-time collaboration works between two browser tabs
- Record deployed URL in SUBMISSION.md
- Run `pnpm test` one final time locally; confirm all tests pass
- Verify `eval/results/` contains all 14 artifacts (7 baselines + 7 after)
- Add demo video link and social post link to SUBMISSION.md once recorded

**Test scenarios:**
- Deployed app: `GET <deployed-url>/health` returns `{ status: "ok" }`
- Deployed app: creating a document and navigating back to it shows the content (database persistence working)
- Deployed app: opening in two browser tabs and editing in one updates the other in <3 seconds (WebSocket working)

**Verification:** Deployed URL is publicly accessible. SUBMISSION.md checklist complete. `pnpm test` green on final local run.

---

## System-Wide Impact

- **Existing Playwright suite:** must remain fully green throughout. U11–U17 each require a test run before commit.
- **Database:** one new migration (U14 index) — additive only, no schema changes. Existing queries are unaffected; the index only accelerates reads.
- **Build pipeline:** `rollup-plugin-visualizer` added as a dev dependency (U3); does not change production build output.
- **API startup:** observability middleware (U7) adds ~1ms per request to response time. Negligible; fail-open.
- **TypeScript compiler:** U11 fixes reduce error count. If strict mode was previously off and U11 enables it, downstream files may surface new errors — resolve them before landing the fix.

---

## Risks and Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Type safety fixes introduce a runtime regression not caught by E2E tests | Medium | Run `pnpm test` after each file; write a targeted unit test for any fixed function with complex behavior |
| Code splitting breaks a lazy-loaded route in production | Medium | Add a Playwright smoke test for each lazily-loaded route after U12 |
| Database index causes migration failure on seeded data | Low | Test migration on local seeded database before committing; index additions are generally safe |
| Accessibility fixes to ARIA attributes confuse existing Playwright selectors | Low | Playwright selects by accessible role by default — adding ARIA labels improves selector reliability, not degrades it |
| Deployed environment doesn't support WebSocket (Vercel) | Medium | Choose Render or Railway which support persistent WebSocket connections |
| Benchmark results are not reproducible across machines | Medium | Commit baseline on the same machine used for after-measurement; document hardware in `eval/README.md` |

---

## Deferred Implementation Notes

- Exact method names and ORM query patterns in `api/src/db/` — determined on fork when actual code is read
- Specific Tailwind color token changes for contrast fixes — depends on the current design system token values
- Whether `pnpm db:seed` exists or a custom seed script is needed — determined in U4
- Exact Playwright fixture patterns to follow for new tests (U15) — read from existing `e2e/` on fork
- Whether health routes already exist in the Ship API — verified in U7 before adding

---

## References

- Requirements document: `docs/brainstorms/shipshape-requirements.md`
- ShipShape assignment PDF: `GFA Week 4 - ShipShape.pdf`
- Target repo: `https://github.com/US-Department-of-the-Treasury/ship`
- Week 3 architecture style: `Week3/ARCHITECTURE.md`
- Week 3 threat model style: `Week3/THREAT_MODEL.md`
- Week 3 observability style: `Week3/agentforge/observability/events.py`, `Week3/agentforge/observability/langfuse_client.py`
- Week 3 test style: `Week3/tests/agentforge/`
- Week 3 submission control style: `Week3/SUBMISSION.md`
