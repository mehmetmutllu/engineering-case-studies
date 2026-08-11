# SEY Silver — Luxury Silver Jewelry E-Commerce

Production e-commerce platform for a luxury silver jewelry brand: catalog, cart, checkout with 3D Secure payment and shipping integration.

*Source code: private (client-owned production system).*

## Problem

The brand needed a fully owned e-commerce presence — product catalog, secure card payments and shipping — running reliably on affordable hosting, without platform lock-in or per-sale commissions of marketplace/e-commerce SaaS.

## My Role

Sole engineer — architecture, implementation, testing and deployment.

## Architecture

- **PHP 8 + MariaDB** application: catalog, product variants, cart, order lifecycle and admin panel.
- **iyzico 3D Secure** payment flow — full callback/verification handling for Turkish card payments.
- **Shipping API integration** for label/tracking automation.
- Built to run efficiently on shared hosting (no heavy framework dependency).

## Main Technologies

PHP 8 · MariaDB · JavaScript · iyzico (3D Secure) · shipping APIs · SMTP

## Important Engineering Decisions

- **Payment flow treated as the critical path** — the 3D Secure callback chain is covered by end-to-end tests rather than manual checks.
- **Playwright E2E testing** against real user flows (browse → cart → checkout) to protect the revenue path on every change.
- **Automated deployment** — changes ship through an automated Git-based deploy instead of FTP file pushes.

## Integrations

iyzico · shipping/logistics API · SMTP

## Testing

Playwright end-to-end suites over the storefront and checkout flows.

## Deployment

Automated Git-based deployment to production hosting.

## Outcome

Live production store. Sales metrics: not publicly disclosed.
