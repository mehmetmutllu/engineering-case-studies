# AI Co-Founder Studio — AI-Assisted Product Planning System

An AI-assisted product development and planning system: describe an idea in natural language, receive feasibility analysis, market/competitor scan, technical architecture and — critically — **persistent development context files** ready to drive AI-assisted implementation.

*Source code: public repository → [ai-co-founder-studio](https://github.com/mehmetmutllu/ai-co-founder-studio).*

## Problem

Two problems, one product:

1. Going from "idea" to "validated plan" takes weeks of scattered research (market, competitors, architecture, MVP scope).
2. AI-assisted development suffers from **context loss** — every new AI session starts blank unless the project's knowledge lives in persistent, structured files.

## My Role

Sole engineer and product designer.

## Architecture

- **React 18 + TypeScript + Vite** front end: landing page, studio dashboard, chat-driven analysis flow, model selector, tabbed analysis views.
- **shadcn/ui + Radix + Tailwind** component system with framer-motion interaction design.
- **Typed bilingual i18n** (TR/EN) — every UI string lives in a typed dictionary.
- **Planned AI backend** — the current release is a validated product prototype with simulated analysis output; AI-service integration is the next milestone.

## The Core Concept: Context Files as the Product

The studio's deliverable is not a report — it is a set of machine-consumable markdown files (`CLAUDE.md`-style project brief, agent instructions, specification, progress tracker) that AI coding assistants ingest directly. Planning output *becomes* the persistent context that development sessions consume, closing the gap between "planning" and "building" in AI-assisted workflows.

This mirrors the workflow I use on my own production systems (see [claude-code-engineering-skills](https://github.com/mehmetmutllu/claude-code-engineering-skills)).

## Main Technologies

React 18 · TypeScript · Vite · shadcn/ui · Tailwind CSS · framer-motion · TanStack Query · zod · Vitest · Playwright

## Important Engineering Decisions

- **Chunked context architecture** — planning output is split across multiple files so long AI development sessions don't drown in context bloat.
- **Prototype-first** — the full UX was built and validated before committing to an AI backend architecture.
- **Bilingual from day one** — typed i18n dictionary instead of retrofitting localization later.

## Outcome

Working product prototype; AI-service integration planned. Metrics: not applicable (pre-launch).
