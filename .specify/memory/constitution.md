<!--
Sync Impact Report
==================
Version change: (unversioned template) → 1.0.0
Bump rationale: Initial ratification; all placeholders replaced with concrete governance.

Modified principles (template placeholder → new title):
  - [PRINCIPLE_1_NAME] → I. Simplicity & Readable Code
  - [PRINCIPLE_2_NAME] → II. Tested Logic & Endpoints (NON-NEGOTIABLE)
  - [PRINCIPLE_3_NAME] → III. Consistent REST Contract
  - [PRINCIPLE_4_NAME] → IV. Secure by Default
  - [PRINCIPLE_5_NAME] → V. Sensitive-Data Readiness

Added sections:
  - Technology & Platform Constraints (was [SECTION_2_NAME])
  - Development Workflow & Quality Gates (was [SECTION_3_NAME])
  - Governance (filled)

Removed sections: none

Templates: not modified by this command (they read the constitution at runtime).
  - .specify/templates/plan-template.md: Constitution Check gates come from this file at runtime
  - .specify/templates/spec-template.md: no change required
  - .specify/templates/tasks-template.md: no change required

Deferred TODOs: none
-->

# FocusLedger Constitution

## Core Principles

### I. Simplicity & Readable Code

- Code MUST favor clarity over cleverness: descriptive names, small focused methods, and
  straightforward control flow.
- New third-party dependencies (NuGet packages) MUST NOT be added unless the built-in .NET /
  ASP.NET Core capabilities are demonstrably insufficient. The justification MUST be recorded in
  the feature's plan (Complexity Tracking) or the pull request description.
- New architectural layers, projects, abstractions, or patterns (e.g., repositories over the ORM,
  mediators, extra service tiers, interfaces with a single implementation that exist only "for
  the future") MUST NOT be introduced without a concrete, current need such as testability that
  cannot otherwise be achieved, or multiple real implementations.
- Build only what the current specification requires (YAGNI). Speculative extension points are
  prohibited.

**Rationale**: A small, readable codebase is cheaper to change, review, and secure. Every
dependency and layer is a long-term cost that must pay for itself.

### II. Tested Logic & Endpoints (NON-NEGOTIABLE)

- All business logic MUST be covered by unit tests that exercise its rules, edge cases, and
  failure paths without requiring HTTP, a real database, or external services.
- Every HTTP endpoint MUST have at least one integration test that calls it through the real
  ASP.NET Core pipeline (e.g., `WebApplicationFactory`), covering the success path and, where
  applicable, validation failure and authorization failure.
- Business logic MUST be kept out of endpoint/controller code so it can be unit tested directly.
- A change that adds or modifies business logic or an endpoint without the corresponding tests
  MUST NOT be merged. All tests MUST pass before merge.

**Rationale**: Unit tests protect the rules that matter; integration tests prove that routing,
validation, serialization, error handling, and auth are wired together correctly.

### III. Consistent REST Contract

- Endpoints MUST follow consistent REST conventions: plural, noun-based, lowercase resource paths;
  HTTP methods used for their standard semantics (GET reads, POST creates, PUT/PATCH updates,
  DELETE removes); and accurate status codes (e.g., 200, 201 with `Location`, 204, 400, 401,
  403, 404, 409, 422 where used consistently, 500).
- Every error response from every endpoint MUST use a single standard format: RFC 9457 Problem
  Details (`application/problem+json`), produced by the framework's built-in Problem Details
  support. Validation errors MUST list the offending fields. Unhandled exceptions MUST be
  converted to a Problem Details 500 response without leaking stack traces or internal details.
- All input (route values, query strings, headers used as input, and request bodies) MUST be
  validated before it reaches business logic. Invalid input MUST be rejected with a 400 Problem
  Details response.
- Request and response payloads MUST use explicit DTOs; persistence or domain entities MUST NOT
  be exposed directly over the API.

**Rationale**: A predictable contract makes the API easy to consume and test, and uniform error
handling prevents accidental information leaks.

### IV. Secure by Default

- Secrets (connection strings, keys, tokens, passwords, certificates) MUST NOT be committed to
  source code or checked-in configuration files. They MUST come from configuration sources such as
  environment variables, .NET User Secrets (local development only), or a secrets manager
  (e.g., Azure Key Vault) in deployed environments.
- Every protected endpoint MUST use established authentication and authorization standards
  (e.g., OAuth 2.0 / OpenID Connect with JWT bearer tokens) through ASP.NET Core's built-in
  authentication and authorization middleware. Custom or home-grown authentication schemes or
  cryptography are prohibited.
- Authorization MUST be deny-by-default: endpoints require an authenticated user unless explicitly
  marked anonymous, and anonymous access MUST be a deliberate, reviewed decision.
- Authorization decisions MUST be expressed as named policies or equivalent declarative rules,
  and resource-level access checks MUST verify the caller may act on the specific resource.

**Rationale**: Leaked secrets and broken access control are among the most common and costly
API vulnerabilities; standard, framework-provided mechanisms are well-tested and well-understood.

### V. Sensitive-Data Readiness

Sensitive data will be introduced in a later stage. The design MUST make it easy to protect
when that happens:

- Sensitive fields MUST NEVER be written to logs, traces, exception messages, or error responses.
  Logging MUST use structured logging with explicit, named properties; whole request bodies,
  DTOs, or entities MUST NOT be logged wholesale.
- Data flows MUST stay explicit (DTOs at the boundary, mapped deliberately) so that a field can
  later be marked sensitive and then masked, encrypted, or excluded in one well-defined place.
- When sensitive data is introduced, its classification, storage protection (e.g., encryption
  at rest), retention, and access rules MUST be defined in the feature specification before
  implementation.

**Rationale**: Retrofitting data protection into a codebase that logs and passes data around
freely is expensive and error-prone; establishing these habits now makes the later change small.

## Technology & Platform Constraints

- The project is a .NET Web API built on ASP.NET Core, targeting a currently supported .NET
  Long-Term Support (LTS) release.
- Built-in framework features (dependency injection, configuration, logging, Problem Details,
  validation, authentication/authorization, OpenAPI) MUST be preferred over third-party
  alternatives, per Principle I.
- Nullable reference types MUST be enabled, and compiler warnings MUST be treated as issues to
  resolve rather than suppress without justification.
- Configuration MUST be environment-specific through the standard .NET configuration system;
  no environment-specific values are hard-coded.

## Development Workflow & Quality Gates

- Each feature follows the Spec Kit flow: specification → plan → tasks → implementation. The plan's
  Constitution Check MUST pass (or record a justified exception) before implementation begins.
- Every pull request MUST, before merge:
  - build without errors and pass all unit and integration tests;
  - include unit tests for new or changed business logic and at least one integration test for
    each new or changed endpoint (Principle II);
  - use the standard Problem Details error format and validate all input (Principle III);
  - contain no secrets and apply authentication/authorization to protected endpoints
    (Principle IV);
  - not log sensitive or whole-payload data (Principle V);
  - justify any new dependency or architectural layer (Principle I).
- Reviewers MUST reject changes that violate a principle without a documented, approved exception.

## Governance

- This constitution supersedes all other development practices and guidance for FocusLedger.
  Where another document conflicts with it, this constitution prevails.
- **Amendments**: Proposed changes MUST be made through the `/speckit-constitution` workflow,
  include a Sync Impact Report, and be reviewed and approved by the project owner before merge.
  Dependent artifacts (plans, specs, tasks) MUST be reviewed for consistency after an amendment.
- **Versioning**: The constitution follows semantic versioning:
  - MAJOR: backward-incompatible removal or redefinition of a principle or governance rule;
  - MINOR: a new principle or section, or materially expanded guidance;
  - PATCH: clarifications, wording, and typo fixes with no semantic change.
- **Compliance**: Every plan MUST include a Constitution Check against these principles, and every
  pull request review MUST verify compliance with the Quality Gates above. Exceptions MUST be
  documented with rationale in the plan's Complexity Tracking section and are scoped to that
  feature only.

**Version**: 1.0.0 | **Ratified**: 2026-11-11 | **Last Amended**: 2026-11-11
