# Pestoptan — Wholesale Catalog & Quotation Platform

Production platform for a wholesale business (cleaning, hygiene and consumable products): customers browse a filtered catalog, build a quotation cart and request an offer; the sales team is notified automatically.

*Source code: private (client-owned production system).*

## Problem

The wholesaler took orders over phone and messaging apps. Customers had no self-service way to browse the full product range, and every quotation request consumed sales-team time just to collect the item list.

## My Role

Sole engineer — product, implementation and deployment.

## Architecture

- **PHP 8.2 + MySQL (PDO)**, intentionally framework-free for the hosting target.
- **Catalog with advanced filtering** across product categories.
- **Cart-based quotation workflow** — customers assemble a quote list and submit it with a single form; no online payment, matching how wholesale pricing actually works.
- **Automated sales-team notification** — each quotation request is emailed to the business (PHPMailer + SMTP) with the full item list.
- **Admin panel** for product and catalog management.

## Main Technologies

PHP 8.2 · MySQL (PDO) · JavaScript · PHPMailer · SMTP

## Important Engineering Decisions

- **Quotation cart instead of checkout** — modeling the real wholesale sales process (negotiated pricing) rather than forcing an e-commerce checkout that would never be used.
- **Environment-based configuration** — no credentials in the codebase; deploys are reproducible across environments.
- **Git-based production deploy** — hosting pulls from the repository, removing manual FTP steps.

## Integrations

SMTP (quotation notifications)

## Testing

Not publicly disclosed.

## Deployment

Git-based deploy to production hosting.

## Outcome

Live production system used by the business's customers. Metrics: not publicly disclosed.
