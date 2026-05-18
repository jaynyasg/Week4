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
    eslint-baseline.json                     ← from U2 (existing ESLint config output)
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
    madge-circular-baseline.txt              ← from U24 (circular dependency report)
    madge-graph-web.svg                      ← from U24 (web dependency graph)
    madge-graph-api.svg                      ← from U24 (api dependency graph)
    dependency-audit-baseline.json           ← from U18 (pnpm audit CVE scan)
    dependency-outdated-baseline.json        ← from U18 (pnpm outdated freshness)
    dependency-summary-baseline.md           ← from U18 (human-readable summary)
    otel-trace-sample.json                   ← from U25 (sample OpenTelemetry trace)

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
- `eval/results/madge-circular-baseline.txt` — circular dependency report
- `eval/results/madge-graph-web.svg` — web/ dependency graph
- `eval/results/madge-graph-api.svg` — api/ dependency graph

**Approach:**

Read sources, take notes, answer every question. The output is a structured Markdown document with one subsection per checklist item. Be specific — cite file paths and line ranges where claims are grounded.

*Phase 1: First Contact*

**1.1 Repository Overview**
- Clone steps that worked, and any steps not in the README (record verbatim — feeds SUBMISSION.md setup section)
- Read every file in `docs/` (the Ship repo's own docs folder, not Week4's); summarize each file's key architectural decisions in your own words
- Read the `shared/` package: list every exported type, group by purpose (DTOs, enums, utility types), note which are used in `web/`, which in `api/`, which in both
- Create a Mermaid diagram showing how `web/`, `api/`, and `shared/` packages relate (imports, type sharing, build dependencies)
- **Run `pnpm dlx madge --circular --extensions ts,tsx web/src api/src` to detect circular import dependencies.** Commit the output to `eval/results/madge-circular-baseline.txt`. If circular dependencies exist, list them in §1.1 of ORIENTATION.md with file paths and the cycle chain — they are often hidden architectural issues worth flagging in §3.1 (weakest points). If none exist, note that explicitly as a strong architectural signal
- **Run `pnpm dlx madge --image eval/results/madge-graph-web.svg web/src/main.tsx` and `pnpm dlx madge --image eval/results/madge-graph-api.svg api/src/index.ts`** (paths adjusted to actual entry points) to generate visual dependency graphs. Commit both SVG files. The visual graph is a powerful orientation artifact that complements the hand-drawn Mermaid diagram

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

**Verification:** `ORIENTATION.md` at Ship fork root contains all 8 numbered subsections (1.1, 1.2, 1.3, 2.1, 2.2, 2.3, 2.4, 3.1). Every question is answered with specific file paths or evidence (not generic prose). Mermaid diagram for 1.1 renders correctly. Three `madge` artifacts committed: circular dependency report (text) and dependency graphs (SVG) for both `web/` and `api/`. Synthesis section (3.1) names specific decisions and weaknesses, not generic ones — circular dependencies, if any, are flagged here. **No audit measurement (U2–U5) begins until this unit is complete.**

---

### U2. Type Safety + Code Quality Baseline Measurement

**Goal:** Capture a reproducible baseline for type safety (grep counts + `type-coverage` percentage), positive TypeScript pattern inventory, AND existing ESLint warnings/errors. The combined output is a measurable picture of code quality before any fix lands. Violation count drives the 25% PDF improvement target; the broader signals feed AUDIT.md Discovery and ARCHITECTURE §9.1.

**Requirements:** 7-category audit (type safety category) + PDF orientation item 2.2 (TypeScript Patterns inventory) + code quality baseline for AUDIT.md.

**Dependencies:** U1, U24

**Files:**
- `eval/results/type-safety-baseline.json` — committed artifact (violation counts + type-coverage % + positive pattern inventory)
- `eval/results/eslint-baseline.json` — committed artifact (existing rule violations from the codebase's ESLint config)

**Approach:**

*Violation count (baseline for improvement target):*
- Run grep counts for `: any`, `as `, `!.`, `@ts-ignore`, `@ts-expect-error` across `web/src/`, `api/src/`, `shared/` — break down by package and violation type
- Check `tsconfig.json` strict mode setting; if not enabled, run `tsc --strict --noEmit` and record error count
- Identify the 5 most violation-dense files
- The 25% improvement target requires fixing at least `floor(total × 0.25)` violations; compute and record that threshold in the artifact

**Edge case — strict-mode explosion:** If `tsc --strict --noEmit` produces an overwhelming error count (specifically: `tsc_strict_errors > 5 × total_grep_violations`), do NOT plan to enable strict mode as part of this audit. Strict mode reveals implicit-any errors that grep cannot find, and turning it on in a codebase that was never written under strict can produce hundreds-to-thousands of errors. The 25% improvement target is on *grep violations* (the visible quality bar), not on strict-mode compilation errors. In this case:
- Record `tsc_strict_errors` count for transparency
- Mark `strict_mode_recommendation: "deferred_due_to_scale"` in the baseline JSON
- Note the gap in AUDIT.md as a documented residual risk and a future-work recommendation, not a failure
- The 25% target on grep violations still applies and remains achievable

*`type-coverage` percentage (sophisticated type-safety metric):*
- Install `type-coverage` as a dev dependency (`pnpm add -D type-coverage`)
- Run `pnpm dlx type-coverage --detail` and capture the reported percentage (% of identifiers that are NOT `any`, including implicit `any` from missing return types that grep misses)
- Record uncovered identifier locations with `--detail` output
- This metric will be tracked before/after — improvement target maps to a percentage delta in addition to raw count reduction
- Higher type-coverage with same grep counts means we eliminated implicit `any` from inference, which grep alone wouldn't catch

*Positive pattern inventory (cross-references U24 findings):*
- Count usages of: discriminated unions (look for `type X = A | B` patterns where each variant has a literal discriminator), generics (`<T>` in function/type definitions), utility types (`Pick`, `Omit`, `Partial`, `Required`, `Readonly`, `Record`), type guards (functions returning `x is Y`)
- Record one canonical example of each (file path + line range) — these become evidence for AUDIT.md Discovery section

*ESLint baseline (`eval/results/eslint-baseline.json`):*
- Verify ESLint is configured in the codebase (`.eslintrc.*` or `eslint.config.*`); if not present, note that and skip this baseline
- If configured: run `pnpm exec eslint . --format=json --output-file=eval/results/eslint-baseline.json` (or equivalent for the package manager)
- Parse the output and produce a summary: total errors, total warnings, top 5 rules by frequency, top 5 files by violation count
- Record summary in the baseline artifact alongside the raw JSON
- This is an existing-codebase quality measurement — we are NOT changing the ESLint config or fixing the violations as part of the audit (those are out of scope unless they happen to overlap with type safety fixes)

*Artifact shape (`eval/results/type-safety-baseline.json`):*
```json
{
  "violations": {
    "any": { "total": N, "by_package": { "web": N, "api": N, "shared": N }, "top_files": [...] },
    "as_assertions": { ... },
    "non_null_assertions": { ... },
    "ts_ignore": { ... }
  },
  "strict_mode": { "enabled": bool, "tsc_strict_errors": N },
  "type_coverage": { "percentage": 87.4, "uncovered_count": N, "sample_uncovered": [...] },
  "improvement_target": { "fix_at_least": floor(total * 0.25), "target_type_coverage": 90.0 },
  "positive_patterns": {
    "generics": { "count": N, "example": "path/to/file.ts:42" },
    "discriminated_unions": { "count": N, "example": "..." },
    "utility_types": { "count": N, "examples": ["Pick: ...", "Omit: ...", "Partial: ..."] },
    "type_guards": { "count": N, "example": "..." }
  },
  "eslint_summary": {
    "configured": true,
    "total_errors": N,
    "total_warnings": N,
    "top_rules": ["@typescript-eslint/no-explicit-any: 87", "..."],
    "top_files": ["web/src/components/Editor.tsx: 24", "..."]
  }
}
```

**Test scenarios:**
- Test expectation: none — measurement-only unit

**Verification:** `eval/results/type-safety-baseline.json` exists, is committed, contains per-package violation counts, top-5 file list, `type-coverage` percentage with sample uncovered identifiers, positive pattern inventory, AND ESLint summary. `eval/results/eslint-baseline.json` contains the raw ESLint JSON output (if ESLint is configured).

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

**Edge case — empty DB pre-flight:** Every benchmark script in `eval/` must verify the database is seeded before running, and fail-fast with a clear error message if not. Benchmarking against an empty DB produces meaningless P95 numbers (everything returns 404 or empty arrays fast) and would silently invalidate the entire audit category. Implementation:
- `eval/benchmark-api.js` calls `GET /documents` (or equivalent list endpoint) as a pre-flight check; if the response array length is < 100, exit with code 1 and message: `"Database appears un-seeded (<100 documents). Run pnpm db:seed first."`
- `eval/benchmark-queries.sql` includes a pre-flight `SELECT count(*) FROM documents` row at the top with a comment instructing the operator to verify the count is ≥ 500 before trusting subsequent EXPLAIN ANALYZE output
- The pre-flight check is itself tested: temporarily truncate the documents table and verify `benchmark-api.js --baseline` exits with code 1 instead of producing a misleading baseline

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

**Edge case — flaky test attribution:** Capture the *signature* of each pre-existing flaky test (test name, file path, failure pattern across the 3 runs) in the baseline artifact under a `flaky_baseline` field. This creates the attribution boundary: any *new* flake observed after improvements land that was NOT in this signature list is attributable to our changes. Implementation:
```json
"flaky_baseline": [
  { "test_id": "e2e/documents.spec.ts:should-load-list", "failures_in_3_runs": 1, "failure_pattern": "timeout waiting for selector" },
  ...
]
```
- Without this attribution boundary, "the test suite is still flaky after our changes" is ambiguous — we cannot tell if our work introduced new flakes or simply didn't fix old ones. The baseline is the contract.

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

### U7. Observability Layer — Custom Middleware + Sentry + Health Endpoints

**Goal:** Add a three-part observability layer: (1) fail-open custom Express middleware for structured request logs, (2) Sentry SDK integration for error tracking and performance monitoring on backend and frontend, (3) health/readiness endpoints. All three are additive — they do not modify any existing middleware or route handler logic.

**Requirements:** Enhancement package — observability layer (tier 1 always-on + tier 2 error tracking).

**Dependencies:** U1

**Files:**
- `api/src/middleware/observability.ts` — custom fail-open middleware
- `api/src/observability/sentry.ts` — backend Sentry initialization
- `web/src/observability/sentry.ts` — frontend Sentry initialization
- `api/src/routes/health.ts` (only if `/health` and `/ready` do not already exist)
- `api/src/__tests__/middleware/observability.test.ts`
- `.env.example` — add `SENTRY_DSN_API`, `SENTRY_DSN_WEB`, `SENTRY_ENVIRONMENT`, `SENTRY_TRACES_SAMPLE_RATE` (documented; not committed with real values)

**Approach:**

*Custom middleware (`api/src/middleware/observability.ts`):*
- Express middleware that records `method`, `path`, `status_code`, `duration_ms` as structured JSON to stdout on every request completion
- Catches and records `error_type`, `error_message` (no stack trace) on unhandled errors
- Emits event types: `request_completed`, `request_errored`, `server_started`
- Deliberately omits: `Authorization` header value, `Cookie` header value, request body, response body
- Fail-open: wraps the recording in try/catch so middleware errors never propagate to the request
- Attached in `api/src/app.ts` (or equivalent entry point) as the first middleware after body-parser — no changes to any other existing middleware

*Backend Sentry (`api/src/observability/sentry.ts`):*
- Initialize `@sentry/node` with `dsn: process.env.SENTRY_DSN_API`, `environment: process.env.SENTRY_ENVIRONMENT ?? "development"`, `tracesSampleRate: parseFloat(process.env.SENTRY_TRACES_SAMPLE_RATE ?? "0.1")`
- Initialize at the very top of `api/src/app.ts` (or main entry) — must run before other imports for auto-instrumentation
- Attach `Sentry.Handlers.requestHandler()` as the first middleware (before the custom observability middleware)
- Attach `Sentry.Handlers.errorHandler()` as the last error middleware
- Add `beforeSend` hook that strips PII-like fields: `Authorization`, `Cookie` headers; any request body field named `password`, `token`, `secret`
- If `SENTRY_DSN_API` is unset (e.g., local dev without Sentry), the SDK no-ops; no errors thrown

*Frontend Sentry (`web/src/observability/sentry.ts`):*
- Initialize `@sentry/react` in `web/src/main.tsx` (or app entry) with `dsn: import.meta.env.VITE_SENTRY_DSN_WEB`, integrations including `BrowserTracing` and `Replay` (Replay sampled at 0% by default; 100% on errors)
- Wire `Sentry.ErrorBoundary` as a top-level wrapper above the React tree — complements the granular per-component error boundaries added in U16
- `beforeSend` strips any user-typed document content from breadcrumbs (document content may contain sensitive info)
- If `VITE_SENTRY_DSN_WEB` is unset, SDK no-ops

*Health routes (`api/src/routes/health.ts`):*
- Verify whether `GET /health` and `GET /ready` already exist in `api/src/`; if they do, make no changes
- If not present: add `GET /health → { status: "ok" }` and `GET /ready → { status: "ready" | "degraded", db_connected: boolean }` — check DB connection with a lightweight ping query
- Register health routes before the observability middleware so they do not appear in structured logs (or register them after but filter them from logs by path prefix)
- Sentry's `requestHandler()` excludes health route paths from tracing via `ignoreTransactions` config

**Why three components and not one:**
- Custom middleware is always on, costs nothing, gives a structured log baseline that works without any external service. Demo-friendly and grader-reproducible.
- Sentry provides rich error context (breadcrumbs, source-mapped stack traces, user session replay, performance metrics) that stdout logs cannot. Free tier covers this project; gracefully no-ops when DSN absent.
- Health endpoints answer "is this app up" without requiring Sentry login.

**Edge case — Sentry free tier rate limit:** Sentry's free tier allows 5K events/month per project. Repeated dev/test runs that trigger error capture can exhaust this quota quickly (a single test loop that throws a handled error 1000 times burns 20% of the monthly quota). Mitigation:
- Use a dedicated Sentry project for `development` environment (separate from `production`) so dev events do not eat production quota
- Set `tracesSampleRate: 0.1` in dev (10% of transactions traced) and `tracesSampleRate: 1.0` only in `production`
- Set `Replay` integration to `sessionSampleRate: 0` in dev (replays only on errors, not on routine sessions)
- For deliberate error-capture tests (e.g., U16 ErrorBoundary smoke test), mock `@sentry/node` and `@sentry/react` so the test asserts on the mock without sending real events to Sentry
- Document the recommended Sentry project structure in `docs/observability-runbook.md`: one project for `production`, one for `staging`, one for `development`
- If a test run is observed to be sending real events: `SENTRY_DSN_API= SENTRY_DSN_WEB= pnpm test` runs with Sentry as no-op (no events sent)

**Patterns to follow:** Week 3 `agentforge/observability/events.py` for event structure; Week 3 `agentforge/http/app.py` for how health routes are registered.

**Test scenarios:**

*Custom middleware:*
- Happy path: middleware records `request_completed` event with correct method/path/status/duration for a normal GET request
- Error path: when a downstream handler throws, middleware records `request_errored` event without leaking the stack trace
- Sensitive data exclusion: `Authorization` header value does not appear in any logged output
- Fail-open: when the middleware's own logging code throws, the original request still completes with the correct status

*Sentry backend:*
- When `SENTRY_DSN_API` is unset, app starts without errors and no Sentry transport is initialized (verify via mock)
- `beforeSend` hook strips `Authorization` header from captured event payload (assertion against `Sentry.captureException` mock)
- Health route `GET /health` does NOT generate a Sentry transaction

*Sentry frontend:*
- (smoke test in U10 a11y suite or U15 new E2E test) ErrorBoundary catches a deliberately-thrown render error and renders fallback UI without crashing the page

*Health routes:*
- `GET /health` returns `{ status: "ok" }` with 200 (only if route is new)
- `GET /ready` returns `db_connected: true` when DB is reachable, `db_connected: false` and `status: "degraded"` when DB ping fails (only if route is new)

**Verification:** Unit tests pass. Middleware is attached and logs structured events during `pnpm dev`. Sentry SDK initializes without errors when DSN env vars are unset. With a test DSN configured, a deliberately-thrown error is captured by Sentry (verify in Sentry UI or via local dev console). No existing test regressions.

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

**Edge case — code splitting requires CSR:** `React.lazy()` and `Suspense` for route-level code splitting only work in **client-side-rendered (CSR) applications**. If Ship uses any server-side rendering (SSR) — even partial — naive `React.lazy()` will throw `Error: A React component suspended while rendering, but no fallback UI was specified` on the server, breaking those routes. Before introducing `React.lazy()`:
- Verify Ship is CSR-only: check `web/` for SSR signals — `renderToString`, `renderToPipeableStream`, `getServerSideProps` (Next.js), `loader` exports (Remix), `entry-server.tsx` (Vite SSR templates). The PDF describes Ship as a Vite + React SPA, which is CSR-only, but verify post-fork
- If SSR is in use: code-split via Vite's manual `build.rollupOptions.output.manualChunks` config instead, which produces a similar bundle improvement without affecting render lifecycle. Document the choice in `eval/results/bundle-after.json`
- If CSR-only: proceed with `React.lazy()` + `Suspense` as planned. Smoke-test each lazy-loaded route in the browser before committing — chunk loading errors are runtime, not build-time, and would not surface during `pnpm build`

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

**Edge case — migration failure and rollback:** `CREATE INDEX CONCURRENTLY` can fail mid-flight for several reasons: lock timeout (another transaction held a competing lock), duplicate index name (re-run after partial failure), disk space exhaustion, or query cancellation. When `CONCURRENTLY` fails, PostgreSQL leaves the index in an `INVALID` state — it exists in `pg_index` but cannot be used by the query planner and must be dropped before retrying. The plan must include:
- A companion **down migration** (`DROP INDEX IF EXISTS idx_documents_type_workspace;`) committed alongside the up migration — proves the change is reversible
- The up migration uses `CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_documents_type_workspace ON documents (document_type, workspace_id);` — `IF NOT EXISTS` makes retry safe
- Verification step: after running the up migration, query `SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'documents' AND indexname = 'idx_documents_type_workspace'` AND `SELECT indisvalid FROM pg_index WHERE indexrelid = 'idx_documents_type_workspace'::regclass` — both must return `true`
- If the index is `INVALID` after the up migration, run the down migration, investigate the cause, and retry. Do NOT proceed to benchmark `db-query-after.md` measurement until the index is `VALID`

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

### U16. Error Handling Fixes (with Sentry Capture)

**Goal:** Fix 3 error handling gaps; at least one must address a real user-facing data loss or confusion scenario. Every fix also routes its error path through Sentry (initialized in U7) so the team gets telemetry alongside the UX improvement.

**Requirements:** Runtime error handling improvement target.

**Dependencies:** U5 (error baseline and silent failures cataloged), U6 (gate complete), U7 (Sentry SDK initialized backend + frontend)

**Files:**
- `web/src/` — React error boundaries, loading states, network failure recovery UI
- `web/src/components/ErrorBoundary.tsx` — custom error boundary that wraps `Sentry.ErrorBoundary` and shows app-specific recovery UI
- `api/src/middleware/async-error-handler.ts` — Express async error middleware
- `api/src/` — `process.on('unhandledRejection')` handler that captures to Sentry
- `eval/results/error-after.md`

**Approach:**
- Select the 3 highest-severity gaps from U5:
  - **Data loss priority:** network disconnect during collaborative edit that causes silent data loss — add a reconnection recovery banner and verify Yjs reconnect preserves unsaved state. Capture reconnect failures via `Sentry.captureMessage("yjs_reconnect_failed", { extra: { documentId, attempts } })`
    - **Edge case — expired session on reconnect:** if the user disconnects (laptop closed, network drop) for longer than the session token TTL, the WebSocket reconnect will succeed at the transport layer but the server-side auth check will fail. Without explicit handling, the user appears "connected" while every Yjs sync silently fails. The fix must: (a) detect 401/403 from the server during reconnect handshake or first sync attempt, (b) display a clear UI message ("Session expired — please refresh and sign in"), (c) NOT silently retry the reconnect loop forever with a dead token. Send `Sentry.captureMessage("yjs_reconnect_session_expired", { level: "info" })` to distinguish auth-failure reconnects from network-failure reconnects in telemetry
  - **User confusion priority:** missing error boundary around the document editor — unhandled render errors currently crash the entire app; wrap with a custom `ErrorBoundary` that composes `Sentry.ErrorBoundary` to capture the error AND renders app-specific recovery UI (not Sentry's generic fallback)
  - **Server-side priority:** unhandled promise rejection in a route handler — add Express async error middleware that converts unhandled rejections to 500 responses with structured JSON. Also add `process.on('unhandledRejection', (reason, promise) => { Sentry.captureException(reason); /* structured log */ })` for rejections outside the request scope
- Each fix requires: reproduction steps, before behavior description, after behavior description, screenshot or test evidence
- Add reproduction steps to `eval/results/error-after.md`
- Each fix's Sentry capture is verified manually: trigger the error in dev with a test DSN; confirm the event appears in the Sentry UI with useful breadcrumbs and source-mapped stack trace

**Patterns to follow:**
- React error boundary pattern: `Sentry.ErrorBoundary` with `fallback={<AppRecoveryUI />}` — composes Sentry's capture with custom UI
- Express async error pattern: `app.use((err, req, res, next) => { Sentry.captureException(err); res.status(500).json({ error: "internal", message: err.message }); })`

**Test scenarios:**
- Error boundary: mount a component that throws during render; assert the error boundary catches it, renders a recovery message instead of crashing the app, AND `Sentry.captureException` is called with the original error (use `@sentry/react` mock)
- Malformed input: submit a `POST /documents` request with a body exceeding the size limit; assert a 413 response with descriptive message (not a crash or silent 500); assert Sentry receives the captured error with `extra: { body_size: N, limit: M }`
- Async error middleware: a route handler that throws an unawaited promise; assert the middleware converts it to a 500 with structured JSON response AND Sentry captures the rejection
- Network recovery: (manual test with DevTools network throttle) disconnect during a collaborative edit, reconnect — assert document content is preserved, a status indicator shows "reconnecting" then "connected", AND Sentry records a breadcrumb `{ category: "yjs.reconnect", level: "warning" }` if reconnect took > 5 seconds
- Sentry no-op test: with `SENTRY_DSN_API` unset, all three fixes still work end-to-end (UI recovery, structured 500 response, etc.) — Sentry's no-op behavior must not break the underlying error handling

**Verification:** `eval/results/error-after.md` with before/after for each fix and notes on whether the error appeared in Sentry test environment. Console error count (manual recheck) lower than baseline. `pnpm test` green.

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

**Edge case — Lighthouse score variance:** Lighthouse accessibility scores vary ±2-3 points run-over-run even with zero code change, due to network jitter, font load timing, and CPU contention. A "+10 point improvement" on a single before/after pair could partly be noise. To produce a defensible measurement:
- Run Lighthouse **3 times before** the fix and **3 times after** the fix on the same page, same device, same network conditions
- Compute the **median** score for each set (resistant to outliers)
- Improvement is the delta between medians, not the delta between any two single runs
- Require: `median_after - median_before ≥ 10` on the lowest-scoring page, OR `0 Critical/Serious axe violations` on top 3 pages (the OR-target gives a deterministic fallback when Lighthouse is too noisy)
- Commit all 6 raw Lighthouse JSON exports per page so the median calculation is reproducible

**Test scenarios:**
- `pnpm test:a11y` passes (zero Critical/Serious violations on the 3 target pages) — this is the deterministic axe-based measurement, not subject to score variance
- Lighthouse accessibility score on the lowest-scoring page: median of 3 after-runs is ≥ median of 3 before-runs + 10
- Keyboard navigation: Tab key reaches every interactive element in the header navigation and issue list (manual verification)
- Icon-only buttons (e.g., "close", "add", "delete") have an `aria-label` that NVDA/VoiceOver reads correctly

**Verification:** Before/after Lighthouse JSON artifacts committed (6 files: 3 before, 3 after per target page). Median calculation shown in `eval/results/a11y-after.json`. `pnpm test:a11y` passes. `pnpm test` green.

---

### U18. THREAT_MODEL.md + Dependency Security Baseline

**Goal:** Write a security-oriented threat model for the Ship application AND capture the dependency health baseline (`pnpm audit` + `pnpm outdated`) as measurable evidence behind THREAT_MODEL claims. The baseline turns OWASP A6 (Vulnerable Components) and A8 (Software/Data Integrity) from prose claims into verified numbers.

**Requirements:** Enhancement package — threat model + dependency security baseline.

**Dependencies:** U1 (codebase orientation complete), U16 (error handling gaps are known)

**Files:**
- `THREAT_MODEL.md` — Ship fork root
- `eval/results/dependency-audit-baseline.json` — `pnpm audit --json` raw output
- `eval/results/dependency-outdated-baseline.json` — `pnpm outdated --format=json` output (or `pnpm dlx npm-check-updates --jsonAll` if pnpm output is too sparse)
- `eval/results/dependency-summary-baseline.md` — human-readable summary derived from both JSON files

**Approach:**

*Dependency security baseline (run before writing THREAT_MODEL.md):*
- `pnpm audit --json > eval/results/dependency-audit-baseline.json` — captures CVE list with severity (Critical/High/Medium/Low), affected packages, dependency chain, and recommended versions
- `pnpm outdated --format=json > eval/results/dependency-outdated-baseline.json` (or `pnpm dlx npm-check-updates --format json,group --jsonAll > eval/results/dependency-outdated-baseline.json` for richer output)
- Parse both into a summary (`eval/results/dependency-summary-baseline.md`):
  - Audit summary: total Critical/High/Medium/Low CVE count, top 5 worst CVEs with affected packages
  - Outdated summary: count of dependencies major-version-behind, minor-version-behind, patch-behind; top 10 most-outdated
  - Combined risk score: any Critical or High CVE in a production dependency is flagged as a finding
- These baselines feed THREAT_MODEL.md A6 (Vulnerable and Outdated Components) and A8 (Software/Data Integrity) sections with concrete evidence, not assumptions
- **No fixing** at this stage — fixing CVEs and outdated dependencies is out of audit scope unless they overlap with an existing improvement target. Document the baseline; flag the worst findings in THREAT_MODEL.md residual risks

**Edge case — CVE that cannot be fixed:** `pnpm audit` may flag Critical or High CVEs in transitive dependencies where no compatible patch exists, or where the patched version introduces a breaking change that would cascade across the codebase. The plan must accommodate this:
- For each Critical/High CVE that is NOT fixed during this audit, THREAT_MODEL.md §7 (Residual risks) must include a **"Won't-fix with rationale"** entry naming: CVE ID, affected package, dependency chain (`direct-dep → ... → vulnerable-dep`), why the patch is not viable (no compatible version exists / breaking change / not yet patched upstream), and compensating control or mitigation (e.g., "exploit requires authenticated workspace admin, mitigated by U16 input validation")
- Format:
  ```markdown
  ### Residual: CVE-2024-XXXXX — Critical, won't-fix
  - **Package:** `transitive-pkg@1.2.3` (via `direct-dep@^4.0.0 → ... → transitive-pkg`)
  - **Why not fixed:** Patched in `transitive-pkg@2.0.0` but `direct-dep` is pinned to `transitive-pkg@^1.x`; upgrading `direct-dep` to a version that allows the patch requires a breaking API change in our route handlers
  - **Compensating control:** [exploit prerequisite] mitigated by [existing defense / U16 fix]
  - **Future work:** track `direct-dep` v5 release timeline
  ```
- This pattern is honest documentation — pretending all CVEs are fixed is worse than acknowledging the gap with rationale

*Write `THREAT_MODEL.md` using Week 3 `THREAT_MODEL.md` as a style reference:*

1. **Summary** (~400 words): highest-risk surfaces in a TypeScript/React/Express/PostgreSQL stack — rich-text editor XSS, ORM injection surface, WebSocket auth, session management, Terraform state exposure, dependency CVE exposure
2. **Assets:** user documents, sprint/project metadata, session tokens, WebSocket connections, PostgreSQL credentials, Terraform state
3. **Trust boundaries:** unauthenticated public → authenticated user → workspace admin; WebSocket channel; PostgreSQL network access; CI/CD secrets
4. **Attack surface map:** auth endpoints, document CRUD routes, WebSocket message handling, TipTap rich-text output (XSS surface), `document_type` discriminator (injection surface if unsanitized), Terraform IAM scope, Docker build context, dependency tree — each row with OWASP reference and expected control
5. **Existing defenses:** auth middleware, ORM parameterization (verify in `api/src/db/`), Docker multi-stage build, `pnpm` lockfile
6. **Dependency Security Baseline:** subsection summarizing `dependency-summary-baseline.md` findings — total CVE counts by severity, top 5 worst, most-outdated production dependencies, overall risk posture
7. **Residual risks:** TipTap HTML sanitization scope, WebSocket message validation completeness, Terraform state file exposure, rate limiting absence, any Critical/High CVEs from audit baseline that were not fixed during this project, dependency drift (count of major-version-behind)
8. **Framework references:** OWASP Web Top 10 2021 (A1 Broken Access Control, A3 Injection, A5 Security Misconfiguration, A6 Vulnerable Components, A7 XSS, A8 Software/Data Integrity), NIST SSDF

**Test scenarios:**
- Test expectation: none — documentation + measurement unit

**Verification:** `THREAT_MODEL.md` at fork root covers all 8 sections. OWASP references present. Existing defenses verified against actual code (not assumed). `eval/results/dependency-audit-baseline.json` and `eval/results/dependency-outdated-baseline.json` are committed. `eval/results/dependency-summary-baseline.md` summarizes the worst findings. The Dependency Security Baseline subsection of THREAT_MODEL.md cites specific numbers from these artifacts, not generic prose.

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

**Edge case — TBD marker leak:** The draft version of ARCHITECTURE.md uses `[BASELINE: TBD]` and `[AFTER: TBD]` markers as placeholders. Every marker must be replaced with a real value before the final submission. A grep check is the verification:
- `grep -c "TBD" ARCHITECTURE.md` must return `0`
- `grep -c "\[BASELINE:" ARCHITECTURE.md` must return `0`
- `grep -c "\[AFTER:" ARCHITECTURE.md` must return `0`
- If any of those returns nonzero, list the offending line numbers (`grep -n "TBD" ARCHITECTURE.md`) and fill them in before U22 (final AUDIT.md update). A `TBD` left in a final-submission artifact is a visible defect that signals incomplete work to a grader

**Verification:** `ARCHITECTURE.md` at fork root with all 13 sections. Before/after sections present in Data Model, Real-Time Collaboration, New Features, Database. Mermaid diagram renders correctly. Tools table has at least 15 entries (including all 5 evaluation tools and 2 observability tools added in this project) with "Why not" column filled. **`grep -c "TBD" ARCHITECTURE.md` returns 0.**

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

### U25. OpenTelemetry Distributed Tracing

**Goal:** Add OpenTelemetry SDK auto-instrumentation for the Express API — vendor-neutral distributed traces covering HTTP, Express middleware, and PostgreSQL queries. Exports to a configurable OTLP endpoint OR to a console exporter when no backend is configured (so local dev still produces visible trace output).

**Requirements:** Enhancement package — observability tier 3 (distributed tracing standard).

**Dependencies:** U1, U7 (Sentry + custom middleware should already be wired before adding OTel to avoid concurrent init conflicts)

**Files:**
- `api/src/tracing.ts` — OTel SDK initialization (must be the FIRST import in the app entry point)
- `api/src/app.ts` or `api/src/index.ts` — add `import "./tracing"` as the literal first line (no changes elsewhere)
- `api/package.json` — add OTel dependencies
- `.env.example` — add `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_SERVICE_NAME`, `OTEL_TRACES_EXPORTER`
- `eval/results/otel-trace-sample.json` — committed sample trace output proving auto-instrumentation works
- `docs/observability-runbook.md` (in Ship fork) — how to view traces locally, how to point at a Jaeger/Tempo/Honeycomb backend, how to interpret a trace

**Approach:**

*Dependencies to add (`api/package.json`):*
- `@opentelemetry/sdk-node`
- `@opentelemetry/auto-instrumentations-node`
- `@opentelemetry/exporter-trace-otlp-http`
- `@opentelemetry/api`
- `@opentelemetry/resources`
- `@opentelemetry/semantic-conventions`

*Initialization (`api/src/tracing.ts`):*
- Create a `NodeSDK` instance configured with:
  - `serviceName: process.env.OTEL_SERVICE_NAME ?? "ship-api"`
  - `traceExporter`: switch on `process.env.OTEL_TRACES_EXPORTER` — `"otlp"` (default if endpoint set), `"console"` (default if endpoint unset), or `"none"` (test mode)
  - `instrumentations: [getNodeAutoInstrumentations({ "@opentelemetry/instrumentation-fs": { enabled: false } })]` — disable fs instrumentation (noisy, low signal)
- Call `sdk.start()` synchronously at module load
- Register a `process.on("SIGTERM")` hook that calls `sdk.shutdown()` to flush pending spans
- Wrap initialization in try/catch — if OTel fails to start, log a warning and continue (fail-open, same posture as the custom middleware)

*Sentry coexistence note:*
- Sentry's `requestHandler` and OTel auto-instrumentation can both wrap Express. Order matters: OTel must initialize first (via the `tracing.ts` import at line 1), then Sentry. This is documented in `docs/observability-runbook.md`
- If they conflict in practice, the runbook documents the workaround: use Sentry's `tracesSampleRate: 0` (rely on OTel for traces) OR disable OTel's HTTP instrumentation. Default plan is to keep both running

*Exporter selection by environment:*
- Local dev with no backend: console exporter (traces print to stdout in human-readable form for debugging)
- Local dev with Jaeger via Docker (`docker run jaegertracing/all-in-one`): `OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318`
- Production: point at any OTLP-compatible backend (Honeycomb, Grafana Tempo, Datadog, AWS X-Ray via collector)
- Documentation in `docs/observability-runbook.md` covers all three setups

*Sample trace output:*
- After wiring OTel, run a request through `pnpm dev`, capture one full trace from the console exporter, save it to `eval/results/otel-trace-sample.json`
- This is the evidence artifact proving auto-instrumentation produces meaningful spans (HTTP → Express → PostgreSQL chain)

**Why OpenTelemetry on top of Sentry and the custom middleware:**
- Sentry is excellent for errors and performance summaries but is a vendor-locked format. OTel is vendor-neutral — the same instrumentation feeds Honeycomb, Tempo, Datadog, Jaeger, etc.
- Custom middleware is great for stdout structured logs but doesn't produce parent-child span relationships needed for distributed tracing
- OTel auto-instrumentation specifically captures the PostgreSQL query layer, which complements U14's EXPLAIN ANALYZE evidence with real production query timing
- Demonstrates industry-standard observability discipline, not just a custom solution

**Patterns to follow:** OpenTelemetry Node.js getting-started guide (https://opentelemetry.io/docs/languages/js/getting-started/nodejs/); standard pattern is `--require ./tracing` flag in package.json scripts OR first-line import

**Test scenarios:**
- Unit test: `tracing.ts` exports a `shutdown()` function that resolves successfully
- Unit test: when `OTEL_TRACES_EXPORTER=none`, no exporter is registered and `sdk.start()` does not throw
- Unit test: when init fails (mock by passing invalid config), the app starts anyway and logs a warning (fail-open)
- Integration smoke: with console exporter active, hitting `GET /documents` produces a trace JSON line on stdout containing at least one span with `http.method=GET` and at least one span with `db.system=postgresql`
- Integration smoke: hitting `GET /health` is excluded from traces (low-signal noise)

**Verification:** `pnpm dev` starts the API with OTel active. Sample trace captured to `eval/results/otel-trace-sample.json` shows the HTTP → Express → PostgreSQL span chain for a real document fetch. `docs/observability-runbook.md` documents all three exporter modes. No existing test regressions. Sentry and OTel coexist without errors.

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
