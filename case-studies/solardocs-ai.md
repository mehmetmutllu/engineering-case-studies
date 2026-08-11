# SolarDocsAI — AI Document Automation for Solar Companies

AI-powered document automation platform for German solar companies: takes project inputs and produces completed, compliant documents automatically.

*Source code: private (commercial product).*

## Problem

German solar installation companies produce large volumes of repetitive project documentation. Filling these documents manually is slow, error-prone and scales linearly with headcount.

## My Role

Sole engineer — product, backend, AI integration and payment infrastructure.

## Architecture

- **PHP/MySQL application core** — accounts, projects, document management and a credit/balance system.
- **Python services** — PDF processing and automation pipeline.
- **AI-assisted document completion** — AI fills document fields from project context; generated output is delivered as completed PDFs.
- **Credit/balance billing** — usage is metered by credits, purchased via **Stripe**.

## Main Technologies

PHP · MySQL · Python · PDF automation · Stripe

## Important Engineering Decisions

- **Credits as the billing unit** — decouples pricing from document types and lets usage-based billing evolve without schema churn.
- **PDF pipeline isolated in Python** — document processing runs separately from the web application core.
- **AI prompts and completion logic kept server-side and private** — treated as proprietary product IP.

## Integrations

Stripe · AI model APIs · PDF toolchain

## Testing

Not publicly disclosed.

## Deployment

Not publicly disclosed.

## Outcome

Commercial product; metrics not publicly disclosed.
