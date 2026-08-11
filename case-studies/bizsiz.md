# BizSiz — Multi-Tenant Education Management & Coaching Platform

A multi-tenant education management and coaching platform connecting institutions, teachers, coaches, students and parents through interconnected operational workflows.

*Source code: private (commercial product).*

## Problem

Education institutions (dershane/course centers) run their daily operations — weekly lesson scheduling, attendance, one-to-one lesson bookings, coaching programs, assessments and parent communication — across spreadsheets and disconnected tools. Data never flows between these processes: attendance doesn't inform reports, coaching plans don't see the lesson schedule, parents see nothing at all.

## My Role

Sole engineer: product design, architecture, backend, frontend, testing infrastructure and deployment.

## Architecture

- **Multi-tenant PHP/MySQL monolith** with four role-based panels: institution admin, teacher/coach, student and parent.
- **Tenant-level data isolation** enforced at the query layer — every tenant-owned row carries the institution ID, and access paths validate ownership (composite `id + tenant` lookups) rather than trusting the session alone.
- **Scheduling engine**: an automatic weekly program generator (wizard-driven) that places lessons against hard constraints (teacher availability windows, room/slot capacity, break slots, quotas) and soft preferences (gap minimization, day distribution), with a quality scorecard and preview before publishing.
- **Program lifecycle**: draft → review → published, with downstream consumers (attendance, reports, panels) reading only published programs.
- **One-to-one lesson module**: per-student entitlement (quota) tracking with audit logs, teacher availability checks and conflict detection at booking time.
- **Coaching module**: coach-guided study plans with progress tracking, homework and reporting.
- **Account lifecycle**: a canonical login resolver supporting multiple identifier types, role-record ↔ account synchronization and invitation flows.
- **White-label architecture**: the platform is deployed per-brand; **Biledem Koçluk** is a branded deployment for a real institution, including a Flutter mobile client built on the platform's REST API.

## Main Technologies

PHP · MySQL/MariaDB · REST APIs · JavaScript · Flutter (mobile client) · Playwright · GitHub Actions

## Important Engineering Decisions

- **Reversible, drilled migrations** — every schema change ships as a forward + reverse migration and is exercised by an automated drill script before it may touch staging; destructive changes go through rename-quarantine instead of DROP.
- **Canonical helpers over copy-paste** — recurring logic (active-program resolution, phone normalization, password rules, availability checks) is extracted into single canonical helpers; known unavoidable duplicates are tracked in a "twin registry" so edits to one side prompt a check of the other.
- **Evidence-gated changes** — features and fixes land with executable proof: smoke suites, end-to-end Playwright flows per role panel, and verification scripts that assert cross-module consistency.
- **Staging-first deployment** — every push auto-deploys to staging with migrations applied; production releases are gated behind manual test rounds.

## Testing

Playwright end-to-end suites per role panel (including full 4-panel sweeps), PHP smoke suites for core modules, migration drills, and standalone verification scripts for cross-module invariants (hundreds of assertions run per release).

## Deployment

GitHub Actions → staging auto-deploy with automatic migration application; manual, test-gated production releases.

## Challenges

- Keeping cross-module data synchronized (schedule ↔ attendance ↔ reports ↔ entitlements) in a system where every module both produces and consumes shared state.
- Constraint-solving for schedule generation at acceptable runtime on shared hosting.
- Evolving a live multi-tenant schema safely — solved with the reversible-migration + drill discipline above.

## Outcome

In production use by a real institution (pilot deployment under the Biledem Koçluk brand). Commercial metrics: not publicly disclosed.
