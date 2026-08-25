---
name: dotnet-to-go-planning
description: Build an approved route-scoped .NET-to-Go conversion plan from source analysis and a Go template profile, including target datastore and parity acceptance criteria.
---

# Plan .NET-to-Go Conversion

Use the analysis artifact and Template profile. Present a route selection table keyed by canonical `routeId`. For every selected Route, decide and record:

- contract and type mappings;
- handler, service, repository, middleware, and configuration mappings;
- auth/claims/roles mapping;
- `target datastore`: `sql-server-existing` or a named new datastore;
- schema mapping, migration scripts, data migration plan, and compatibility/read strategy;
- external dependencies, transactional behavior, idempotency, and observability;
- implementation files, tests, Parity fixtures, acceptance criteria, risks, and Blockers.

Separate decisions from evidence. Require explicit resolution for source/spec conflicts and unsupported features. Keep data migration as a plan; do not perform it during route conversion.

Completion: each selected Route has an approved plan, target datastore, evidence references, acceptance criteria, and no unresolved decision hidden in prose.
