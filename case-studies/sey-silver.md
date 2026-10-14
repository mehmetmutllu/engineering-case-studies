# SEY Silver — Silver Jewelry E-Commerce

E-commerce platform for a silver jewelry brand: catalog with full variant matrix, cart, checkout with 3D Secure card payment, carrier abstraction, customer accounts and an admin panel.

*Source code: private (client-owned system).*

## Problem

The brand needed a fully owned e-commerce presence — product catalog, secure card payments and shipping — running reliably on affordable hosting, without platform lock-in or per-sale commissions of marketplace/e-commerce SaaS.

## My Role

Sole engineer — architecture, implementation, testing and deployment.

## Architecture

- **PHP 8.2 + MariaDB/MySQL** application without a framework: catalog, product variants, cart, order lifecycle, customer accounts and admin panel.
- **Product model**: prices stored as integer kuruş; full variant matrix (option groups → per-combination SKU, price, stock, active flag); per-product engraving.
- **PayTR 3D Secure payment flow** — a hand-written HTTP client (cURL + HMAC-SHA256, no SDK) with iframe-based 3DS and callback verification. The same code path runs in a deterministic stub mode for tests and in live mode for production.
- **Carrier abstraction** — a `CarrierClient` interface with MNG and Aras implementations. They currently return deterministic stub tracking numbers; the live API wiring is still pending.
- **Admin hardening** — optional TOTP 2FA (validated against the RFC 6238 test vectors), activity log, CSRF on every form, rate-limited login.
- Built to run on shared hosting (no heavy framework dependency); schema evolves through numbered migrations and a migration runner.

## Main Technologies

PHP 8.2 · MariaDB/MySQL · JavaScript · PayTR (3D Secure) · Playwright · SMTP

## Important Engineering Decisions

- **Payment flow treated as the critical path** — the checkout and PayTR callback chain are covered by Playwright specs in stub mode. An end-to-end run against the PayTR sandbox with test cards is still open.
- **Provider clients behind interfaces with stub modes** — payment, carriers and e-invoicing share the same pattern, so the full purchase path can be tested without external credentials.
- **Tests must prove they ran against the real page** — a stale base-URL variable once made the whole suite pass against a 404 page ("no overflow / no console errors" is trivially true on an empty page). That is now documented in the testing guide as a known failure mode.

## Integrations

PayTR · MNG / Aras (interface + stubs) · SMTP

## Testing

180 Playwright tests in 26 spec files, run across 8 Chromium viewports (320–1920 px) plus Firefox and WebKit at 1440 px.

A GitHub Actions CI workflow is defined (PHP lint + Playwright smoke against a MySQL service container). Its push/PR triggers are still commented out, so it is **manual-trigger only** for now. Tests run locally before each deploy.

## Deployment

Git-based deploy over SSH to the hosting account: push to a bare repository on the server, then a deploy script pulls, applies migrations and prints the deployed commit. The GitHub Actions staging/production deploy workflows also exist but are manual-trigger only.

## Outcome

Soft launch on the current hosting. Still open before the hard launch: DNS cut-over, SSL + HSTS, live PayTR keys and final real product data. Sales metrics: not publicly disclosed.
