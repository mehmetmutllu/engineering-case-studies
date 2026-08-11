# WatchSync AI — Multi-Service Watch Inventory & Listing Platform

A multi-service platform for luxury watch sellers to manage listings, inventory and stock across multiple e-commerce channels.

*Source code: public repository → [watchsync-ai](https://github.com/mehmetmutllu/watchsync-ai). Client-specific configuration and credentials remain private.*

## Problem

Luxury watch dealers sell the same physical inventory on several channels at once (eBay, Shopify storefronts, watch marketplaces). Managing listings by hand means duplicated data entry, price drift between channels and oversold stock.

## My Role

Lead engineer — architecture, backend and AI service; dashboard developed with a second developer under a PR-based review workflow.

## Architecture

Three services with strict separation of responsibilities:

- **Laravel 11 backend** — inventory domain model, channel integrations (eBay, Shopify), webhook signature verification, queue-driven synchronization (Redis + RabbitMQ), transactional email.
- **Next.js 14 frontend** — dealer dashboard for inventory, listings and channel status.
- **FastAPI (Python) AI service** — AI-powered listing enrichment and market/pricing data collection, containerized with Docker.

## Main Technologies

Laravel 11 · PHP · MySQL · Redis · RabbitMQ · Next.js 14 · TypeScript · FastAPI · Python · Docker · Sentry

## Important Engineering Decisions

- **Service split by workload type** — transactional business logic (PHP) is isolated from AI/scraping workloads (Python) so they scale and fail independently.
- **Queue-first channel sync** — marketplace APIs are slow and rate-limited; all channel operations run through queues rather than blocking user requests.
- **Webhook verification before state changes** — channel webhooks are signature-verified before they may touch inventory state.
- **Branch discipline** — protected `main`/`develop`, short-lived feature branches, PR-only merges with code ownership per service.

## Integrations

eBay APIs · Shopify APIs · watch market-data sources · SMTP · Sentry

## Testing

Service-level test setups per stack; browser-level verification on the dashboard.

## Deployment

Dockerized services; per-service environment contracts documented via `.env.example` files.

## Outcome

Active development. Commercial metrics: not publicly disclosed.
