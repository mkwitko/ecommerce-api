# Engineering case study: Fastify backend and explicit boundaries

## Purpose

This repository implements the backend of the Community Storefront web application. It includes authentication, stores, products, orders, donations, payments and related workflows. This document focuses on code-verifiable architecture decisions rather than unverified claims of traffic, revenue or uptime.

## Diagram: request to persistence

~~~mermaid
flowchart TD
  Client["Generated frontend API client"] --> Route["Fastify route + Zod schemas"]
  Route --> Auth["Public / permission decision"]
  Auth --> Controller["HTTP controller"]
  Controller --> UseCase["Use-case service"]
  UseCase --> Repo["Repository interface"]
  UseCase --> Gateway["Gateway interface"]
  Repo --> DB[("PostgreSQL via Prisma")]
  Gateway --> Provider["External service adapter"]
~~~

## Three implementation decisions

**1. Per-use-case vertical slices.** The [login controller, schema and service](src/http/api/auth/login) demonstrate a consistent separation of HTTP transport, validation and application logic. This makes domain operations reviewable independently of the request lifecycle. The [architecture guide](docs/ARCHITECTURE.md) documents the responsibilities and the intended dependency direction.

**2. Explicit access decisions.** The architecture requires each route to declare public access or permission requirements, rather than relying on a permissive default. Example tests and access hooks are in [test/e2e](test/e2e) and [HTTP hooks](src/http/hooks). This is a *design property*, not a third-party security audit.

**3. One contract between web and API.** Fastify/ Zod schemas contribute to [generated OpenAPI](docs/openapi.json); the [web app](https://github.com/mkwitko/udv-shop) regenerates client types and query hooks from that contract. This reduces manually duplicated endpoint types, while adding a code-generation maintenance step to each schema change.

## Real engineering trade-offs

| Decision | Benefit | Cost / risk |
| --- | --- | --- |
| Repository/gateway interfaces | Easier service testing and provider changes | More abstractions to keep focused |
| Zod and generated OpenAPI | Consistent validation and typed clients | Synchronization across API and frontend builds |
| Explicit permissions | Harder to accidentally omit authorization | Each new route must consciously declare access |
| Gateway adapters for payments | Business rules do not hard-code provider SDKs | Reconciliation, idempotency and webhook tests remain essential |

## Where to verify in source

- [HTTP app composition](src/app.ts)
- [Auth use-case](src/http/api/auth/login)
- [Orders and checkout](src/http/api/orders)
- [Webhooks](src/http/api/webhooks)
- [Authentication scenarios](test/e2e)
- [Design decisions](docs/DECISIONS.md)

## Evidence still missing

No deployment benchmark, request rate, SLO, independent penetration test or production-scale measurement was obtained during this documentation work. To strengthen this case, add: controlled load-test methodology, real CI status, representative request/response examples with anonymized fixtures, and metrics captured before/after an actual optimization.
