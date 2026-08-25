---
name: dotnet-source-analysis
description: Analyze a .NET API source path and its OpenAPI or documentation inputs; discover routes, contracts, behavior, dependencies, auth, persistence, evidence, fingerprints, and blockers.
---

# Analyze .NET Source

Read the project and every supplied spec without modifying either. Parse OpenAPI/Swagger before route selection and treat it as the public route index. Inspect solution/project files, controllers/minimal APIs, DTOs, validators, middleware, auth policies, EF/Dapper/SQL access, integrations, configuration references, tests, and declared specs.

For each Route, record:

- canonical `routeId`, HTTP method, normalized path, and `operationId` when present;
- request parameters/body, response schemas/statuses/headers, validation and error contract;
- auth requirements, claims, roles, policies, and middleware dependencies;
- datastore reads/writes, transactions, side effects, external calls, and idempotency;
- evidence paths with symbols or line anchors and whether each fact is declared or inferred;
- source/spec conflicts, unsupported behavior, and missing evidence as Blockers.

Prefer declared spec for public contract and code for behavioral evidence. A missing or invalid spec is a Blocker unless the invocation explicitly enables inferred contracts. A conflict remains unresolved until the user decides. Produce an analysis artifact and update the Conversion manifest with content fingerprints.

Completion: every discovered Route is represented, every claim has evidence, and every uncertainty is either resolved or named as a Blocker.
