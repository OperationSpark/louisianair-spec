# ADR-0007: OpenAPI Spec-First Code Generation with oapi-codegen

## Status

Proposed

## Context

The project API is expected to become a public-facing interface. That makes API contract quality, consistency, and documentation accuracy first-order concerns.

We need a workflow that:

- Keeps the API contract explicit and reviewable before implementation
- Produces handlers/types from an OpenAPI source of truth
- Reduces drift between implementation and published API documentation
- Supports long-term maintainability as endpoints evolve

There is a team preference for readable application code, and junior engineers should be able to navigate the codebase without high cognitive overhead. We also prefer validation and security behavior to live in hand-written application code and middleware, rather than deep inside generated code.

## Decision

Adopt a **spec-first OpenAPI workflow** and use **[`oapi-codegen`](https://github.com/oapi-codegen/oapi-codegen)** for Go server/client code generation.

Implementation direction:

- OpenAPI specification is the primary source of truth for the API contract
- Generated code is committed as build artifact and regenerated on spec changes
- Business logic and validation/security middleware live in hand-written code around generated interfaces and types
- API documentation is derived from the OpenAPI spec and reviewed as part of normal change flow

Rationale:

- For a public API, spec-first gives stronger guarantees that external docs and wire-level behavior stay aligned
- Contract-first review (paths, schemas, errors, examples) catches design issues earlier than code-first approaches
- `oapi-codegen` generated code is generally easier to read and reason about for day-to-day maintenance
- Readability and onboarding speed are prioritized over maximum performance-centric generation features
- Validation in non-generated code gives us explicit control over policy, behavior, and error contracts

## Consequences

### Positive

- API contract is explicit, versionable, and reviewable independent of implementation
- Better documentation correctness for external consumers
- Cleaner change management: contract changes are visible in spec diffs
- Faster scaffolding for handlers/types and reduced hand-written boilerplate
- More approachable generated code for junior and mid-level contributors
- Clear separation of concerns: generated contract types vs hand-written validation/business rules

### Negative

- Validation and security are not "fully handled" by generation; middleware and app code must be maintained carefully
- Extra team discipline is required to keep validation behavior consistent across endpoints
- `oapi-codegen` currently targets OpenAPI 3.0 (not first-class 3.1 support)
- Team must maintain discipline around spec design quality and regeneration workflow

## Mitigations

- Keep generated code isolated (for example, `internal/gen` or similar) and avoid manual edits
- Add concise contributor docs for "edit spec -> generate -> implement -> test" workflow
- Put human-readable boundaries around generated code (service interfaces, adapters, package layout)
- Require OpenAPI review in PRs for endpoint/schema changes
- Add standardized validation middleware and shared error mapping to reduce per-endpoint drift

## Alternatives Considered

### Code-first API design and spec generation

Rejected. Code-first can be faster initially, but for a public API it increases risk of contract/documentation drift and makes interface review less intentional.

### Spec-first with Ogen

Not selected for now. Ogen is a strong option for high-performance generated runtime behavior, but its generated abstractions are less readable for our team and onboarding goals. We are intentionally trading some generator-provided validation/runtime behavior for clearer hand-written application logic.
