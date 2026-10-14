# BizSiz — Multi-Tenant Education Management & Coaching Platform

A multi-tenant education management and coaching platform connecting institutions, teachers, coaches, students and parents through interconnected operational workflows.

*Source code: private (commercial product).*

## Problem

Education institutions (dershane/course centers) run their daily operations — weekly lesson scheduling, attendance, one-to-one lesson bookings, coaching programs, assessments and parent communication — across spreadsheets and disconnected tools. Data never flows between these processes: attendance doesn't inform reports, coaching plans don't see the lesson schedule, parents see nothing at all.

## My Role

Co-founder of the startup behind the product. I designed and built the platform's architecture, backend, frontend, test tooling and deployment pipeline. I no longer work on it full-time; I continue development part-time, driven by feedback from the institutions using it.

## Architecture

- **Multi-tenant PHP/MySQL monolith** with four role-based panels: institution admin, teacher/coach, student and parent.
- **Tenant-level data isolation** enforced at the query layer — every tenant-owned row carries the institution ID, and access paths validate ownership (composite `id + tenant` lookups) rather than trusting the session alone.
- **Scheduling engine**: an automatic weekly program generator (wizard-driven) that places lessons against hard constraints (teacher availability windows, room/slot capacity, break slots, quotas) and soft preferences (gap minimization, day distribution), with a quality scorecard and preview before publishing.
- **Program lifecycle**: draft → review → published, with downstream consumers (attendance, reports, panels) reading only published programs.
- **One-to-one lesson module**: per-student entitlement (quota) tracking with audit logs, teacher availability checks and conflict detection at booking time.
- **Coaching module**: coach-guided study plans with progress tracking, homework and reporting.
- **Account lifecycle**: a canonical login resolver supporting multiple identifier types, role-record ↔ account synchronization and invitation flows.
- **White-label architecture**: the platform is deployed per brand. **Biledem Koçluk** is a branded deployment for a real institution, including a Flutter mobile client built on the platform's REST API.

## Main Technologies

PHP · MySQL/MariaDB · REST APIs · JavaScript · Flutter (mobile client) · PHPStan · Playwright · GitHub Actions

## Important Engineering Decisions

- **Reversible, drilled migrations.** The schema has gone through 100+ migration files across three generations of tooling. The current runner uses numbered migrations (`001`–`080`) with 67 matching reverse scripts. Drill scripts (15 so far) run a full round-trip — fixture → forward → idempotent re-run → reverse → forward — before a change touches staging. A migration-lint workflow rejects `DROP`/`TRUNCATE` in forward files, so destructive changes go through rename-quarantine instead.
- **Tenant isolation as an executable check.** A runtime script (`tools/idor-runtime-test.sh`) logs in as one tenant and tries four access patterns: own record, another tenant's record, missing parameter, non-existent ID. Any deviation from the expected status/redirect fails it with a non-zero exit code. An earlier version only *printed* the expected behavior and always exited 0; an external audit caught this, and it was rewritten. CI additionally runs a static scan for new tenant-leak patterns.
- **Canonical helpers over copy-paste** — recurring logic (active-program resolution, phone normalization, password rules, availability checks) lives in single canonical helpers. Known unavoidable duplicates are tracked in a "twin registry", so an edit to one side prompts a check of the other.
- **Staging-first deployment** — every push to `main` auto-deploys to staging. Production releases are manual and gated.

## Testing

- **CI on every push/PR**: PHP syntax check, PHPStan level 1 (baseline-locked, with a canary that proves the analyzer is actually running), gitleaks secret scan, the tenant-leak scan, a check that commit hooks are in place, and a database-less smoke package.
- **132 PHP smoke scripts** for core modules, plus migration drills and verification scripts for cross-module invariants.
- **Playwright-driven browser scripts** for per-panel regression sweeps and visual/DOM measurements. These are scripts, not a test-runner suite.

## Deployment

18 GitHub Actions workflow files. Staging deploys over SSH on push to `main`, followed by a healthcheck. Production is `workflow_dispatch` only:

1. confirm string
2. schema-drift gate (blocks on manual changes in the live schema)
3. `mysqldump` backup with a size sanity check
4. git tag as rollback point
5. `composer install --no-dev` + migrations
6. healthcheck and a 6-URL panel smoke test, with automatic rollback (code + DB restore) on failure

Separate workflows handle staging refresh with anonymization and diagnostics.

## Challenges

- Keeping cross-module data synchronized (schedule ↔ attendance ↔ reports ↔ entitlements) in a system where every module both produces and consumes shared state.
- Constraint-solving for schedule generation at acceptable runtime on shared hosting.
- Evolving a live multi-tenant schema safely — handled by the reversible-migration + drill discipline above.

## Outcome

In production, including the Biledem Koçluk branded deployment for a real institution. Commercial metrics: not publicly disclosed.
