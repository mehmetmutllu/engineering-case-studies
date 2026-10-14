# Engineering Case Studies

A collection of architecture notes and engineering case studies from web, mobile, SaaS and AI systems I have built.

Production source code is kept private where projects contain client-owned or proprietary software. These case studies focus on **architecture, technical decisions, integrations, testing, deployment and engineering problems** rather than source code.

## Case Studies

| Project | Domain | Stack |
|---------|--------|-------|
| [Kelime Ezberle](case-studies/kelime-ezberle.md) | English vocabulary app for Turkish speakers (App Store) | Vanilla JS · Capacitor 8 · RevenueCat · Firebase · Cloudflare Workers |
| [AI Dungeon](case-studies/ai-dungeon-llm-game-master.md) | LLM Game Master for a mobile RPG (in development) | TypeScript · Cloudflare Workers + D1 · Unity · multi-provider LLM |
| [BizSiz](case-studies/bizsiz.md) | Multi-tenant education management & coaching SaaS | PHP · MySQL · REST · Flutter |
| [WatchSync AI](case-studies/watchsync-ai.md) | Multi-service luxury watch inventory & listing platform | Laravel · Next.js · FastAPI |
| [SolarDocsAI](case-studies/solardocs-ai.md) | AI document automation for German solar companies | PHP · MySQL · Python · Stripe |
| [SEY Silver](case-studies/sey-silver.md) | Silver jewelry e-commerce (soft launch) | PHP 8.2 · MariaDB · PayTR |
| [Pestoptan](case-studies/pestoptan.md) | Wholesale catalog & quotation platform | PHP 8.2 · MySQL |

## Why this repository exists

Most of my production work lives in private repositories due to client confidentiality. This repository makes the *engineering* behind that work reviewable: how the systems are structured, which trade-offs were made and how they are tested and deployed — without exposing proprietary implementations.

No proprietary source code, credentials, customer data or client-confidential details are included. Where a fact is not public, it is marked **"Not publicly disclosed."**
