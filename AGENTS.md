# AGENTS.md

## Project Overview

Dify is an open-source platform for developing LLM applications with an intuitive interface combining agentic AI workflows, RAG pipelines, agent capabilities, and model management.

The codebase is split into:

- **Backend API** (`/api`): Python Flask application organized with Domain-Driven Design
- **Frontend Web** (`/web`): Next.js 15 application using TypeScript and React 19
- **Docker deployment** (`/docker`): Containerized deployment configurations

## Backend Workflow

- Run backend CLI commands through `uv run --project api <command>`.

- Backend QA gate requires passing `make lint`, `make type-check`, and `uv run --project api --dev dev/pytest/pytest_unit_tests.sh` before review.

- Use Makefile targets for linting and formatting; `make lint` and `make type-check` cover the required checks.

- Integration tests are CI-only and are not expected to run in the local environment.

## Frontend Workflow

```bash
cd web
pnpm lint
pnpm lint:fix
pnpm test
```

## Testing & Quality Practices

- Follow TDD: red → green → refactor.
- Use `pytest` for backend tests with Arrange-Act-Assert structure.
- Enforce strong typing; avoid `Any` and prefer explicit type annotations.
- Write self-documenting code; only add comments that explain intent.

## Language Style

- **Python**: Keep type hints on functions and attributes, and implement relevant special methods (e.g., `__repr__`, `__str__`).
- **TypeScript**: Use the strict config, lean on ESLint + Prettier workflows, and avoid `any` types.

## General Practices

- Prefer editing existing files; add new documentation only when requested.
- Inject dependencies through constructors and preserve clean architecture boundaries.
- Handle errors with domain-specific exceptions at the correct layer.

## Project Conventions

- Backend architecture adheres to DDD and Clean Architecture principles.
- Async work runs through Celery with Redis as the broker.
- Frontend user-facing strings must use `web/i18n/en-US/`; avoid hardcoded text.

## Paid.ai Clone Integration

The Paid.ai clone is an OSS implementation of a monetized AI gateway, integrated as a subdirectory (`paid-ai-clone`) within the Dify repository. It provides LLM usage tracking, metering, and billing capabilities to enable paid AI services.

### Stack

- **LiteLLM**: LLM proxy for routing requests to providers (OpenAI, Anthropic, etc.) with built-in usage tracking and event webhooks.
- **OpenMeter**: Open-source metering service for tracking usage metrics (e.g., tokens, calls) and generating bills.
- **Lago**: Open-source billing platform for managing subscriptions, customers, invoices, and payments.
- **Ory Stack**: Kratos (identity), Hydra (OAuth2), Keto (authorization) for user authentication and API security.
- **ReportLab**: Python library for generating PDF receipts/invoices.
- **Docker Compose**: Orchestrates services (LiteLLM, OpenMeter, Lago, Postgres, Redis, Grafana, Traefik).

### Integration with Dify

- The clone extends Dify's backend services by adding a dedicated webhook service that consumes LiteLLM events.
- Events (e.g., chat completions) are processed to update OpenMeter meters and Lago invoices/subscriptions.
- Fits Dify's event-driven architecture: Webhook events can trigger Celery tasks for async processing (e.g., PDF generation).
- DDD alignment: EventPayload as entity, WebhookService for orchestration, repositories for persisting usage data if integrated with Dify DB.
- Security: JWT validation via Ory Hydra for authenticated requests.

### Conventions Adherence

- **DDD/Clean Architecture**: Webhook service uses entities (e.g., `EventPayload`), services (e.g., `MeteringService`, `BillingService`), and repositories (e.g., for idempotency checks). Dependencies injected via constructors.
- **TDD**: Unit tests for event validation/processing (pytest, Arrange-Act-Assert); integration tests for webhook endpoints and DB interactions (CI-only).
- **Strong Typing**: Pydantic models for all payloads (e.g., `EventPayload`, `MeterUpdate`); type hints everywhere, no `Any`.
- **Self-Documenting Code**: PEP8 compliance; comments only for complex intent (e.g., idempotency logic).
- **Async**: Background tasks in FastAPI for non-blocking sends to OpenMeter/Lago; Celery for heavy tasks like PDF generation.
- **Errors**: Domain exceptions (e.g., `InvalidEventError`, `MeteringFailureError`) raised at service layer.
- **QA Gates**: Before completion, run `make lint`, `make type-check`, and pytest unit tests in the clone directory.

### Compliance Requirements

The Paid.ai clone implementation must adhere to the following standards for security, privacy, and data handling:

- **ISO 27001**: Information security management system.
- **AICPA SOC 2**: Trust services criteria for security, availability, processing integrity, confidentiality, and privacy.
- **GDPR**: General Data Protection Regulation for data protection and privacy in the EU.
- **HIPAA**: Health Insurance Portability and Accountability Act for protecting sensitive patient health information (if applicable to use cases).

All components, including webhook processing, data storage in OpenMeter/Lago, and PDF generation, must ensure compliance through encryption, access controls, audit logging, and data minimization.

### Full Clone Scope

The implementation clones every page and feature of Paid.ai exactly, including:

- Frontend pages (dashboard, billing, usage analytics, settings) built with Next.js/TypeScript to match UI/UX.
- Backend APIs matching all functions (user auth via Ory, metering via OpenMeter, billing via Lago, LLM proxy via LiteLLM).
- Observability with Langfuse for tracing.
- Ensure backend endpoints and features (e.g., subscription management, invoice generation, event webhooks) fully replicate Paid.ai capabilities.

## Paid.ai Clone Integration

The Paid.ai clone is an OSS implementation of a monetized AI gateway, integrated as a subdirectory (`paid-ai-clone`) within the Dify repository. It provides LLM usage tracking, metering, and billing capabilities to enable paid AI services.

### Stack

- **LiteLLM**: LLM proxy for routing requests to providers (OpenAI, Anthropic, etc.) with built-in usage tracking and event webhooks.
- **OpenMeter**: Open-source metering service for tracking usage metrics (e.g., tokens, calls) and generating bills.
- **Lago**: Open-source billing platform for managing subscriptions, customers, invoices, and payments.
- **Ory Stack**: Kratos (identity), Hydra (OAuth2), Keto (authorization) for user authentication and API security.
- **Langfuse**: Open-source LLM observability platform for tracing, monitoring, and analytics.
- **ReportLab**: Python library for generating PDF receipts/invoices.
- **Docker Compose**: Orchestrates services (LiteLLM, OpenMeter, Lago, Postgres, Redis, Grafana, Traefik).

### Integration with Dify

- The clone extends Dify's backend services by adding a dedicated webhook service that consumes LiteLLM events.
- Events (e.g., chat completions) are processed to update OpenMeter meters and Lago invoices/subscriptions.
- Fits Dify's event-driven architecture: Webhook events can trigger Celery tasks for async processing (e.g., PDF generation).
- DDD alignment: EventPayload as entity, WebhookService for orchestration, repositories for persisting usage data if integrated with Dify DB.
- Security: JWT validation via Ory Hydra for authenticated requests.
- Webhook integration in `api/services/event_service.py` for Dify compatibility.

### Conventions Adherence

- **DDD/Clean Architecture**: Webhook service uses entities (e.g., `EventPayload`), services (e.g., `MeteringService`, `BillingService`), and repositories (e.g., for idempotency checks). Dependencies injected via constructors.
- **TDD**: Unit tests for event validation/processing (pytest, Arrange-Act-Assert) in `api/tests/test_event_service.py`; integration tests for webhook endpoints and DB interactions (CI-only).
- **Strong Typing**: Pydantic models for all payloads (e.g., `EventPayload`, `MeterUpdate`); type hints everywhere, no `Any`.
- **Self-Documenting Code**: PEP8 compliance; comments only for complex intent (e.g., idempotency logic).
- **Async**: Background tasks in FastAPI for non-blocking sends to OpenMeter/Lago; Celery for heavy tasks like billing updates and PDF generation.
- **Errors**: Domain exceptions (e.g., `InvalidEventError`, `MeteringFailureError`) raised at service layer for clean boundaries.
- **QA Gates**: Before completion, run `make lint`, `make type-check`, and pytest unit tests in the clone directory.

### Full Frontend Replication

The Paid.ai clone requires a full replication of the Paid.ai website (https://app.paid.ai), including every page and feature, built with Next.js/TypeScript/React 19 to match the exact UI/UX.

#### Frontend Pages and Features

- **Dashboard**: Usage and analytics overview, displaying real-time metrics, charts for token consumption, API calls, and cost breakdowns.
- **Billing**: Invoices, subscriptions management (including Stripe integration for payments), plan upgrades/downgrades, and payment history.
- **Usage**: Real-time charts for LLM usage (tokens in/out, models used), historical data, and export options.
- **Settings**: API keys management, plan selection, user profile, integration settings (e.g., webhook URLs), and debug modes.

All pages must replicate the layout, components, and interactions from app.paid.ai, using Tailwind CSS for styling to ensure responsive design.

#### Backend API Support

Backend APIs must fully support the frontend UI via endpoints in the webhook/FastAPI service:

- `/api/usage`: Fetch usage data (e.g., dashboard charts, real-time metrics) aggregated from OpenMeter.
- `/api/billing`: Handle billing views (invoices, subscriptions) integrated with Lago; support Stripe webhooks for payments.
- `/api/analytics`: Provide data for analytics charts (e.g., token trends, cost projections).
- `/api/settings`: Manage API keys, plans, and user settings, with Ory for authentication.

Use DDD services for data aggregation (e.g., `UsageAggregationService`, `BillingQueryService`) to fetch and transform data from OpenMeter, Lago, and LiteLLM logs. Ensure endpoints return typed responses (Pydantic models) for frontend consumption.

#### Integration with Existing Stack

- **Authentication**: Ory Stack (Kratos for identity, Hydra for OAuth2) for user login, JWT tokens in API requests.
- **Billing Data**: Lago for subscription/invoice queries; sync with Stripe for payment processing.
- **Usage Tracking**: OpenMeter for metering data; query meters for UI charts.
- **LLM Proxy**: LiteLLM for proxying calls; expose usage events to frontend via APIs.
- **Observability**: Langfuse integration for traces; display in UI debug mode (e.g., settings toggle to show request traces).

#### Dify Conventions for Frontend

- Use strict ESLint/Prettier configuration, aligned with Dify's `/web` setup.
- No `any` types; enforce explicit TypeScript annotations.
- User-facing strings from `web/i18n/en-US/`; avoid hardcoded text.
- Build the Next.js app in `/paid-ai-clone/web` subdirectory.

#### Testing and Compliance

- **TDD for Frontend**: Use Jest for unit/integration tests; cover components, API fetches, and user flows (e.g., subscription upgrade).
- **UI Compliance**: Implement GDPR consent banners, privacy notices, and data export features. Ensure HIPAA compliance for any health-related use cases via anonymization. Adhere to ISO 27001/SOC 2 through secure API calls (HTTPS, JWT validation) and audit logging of user actions.
