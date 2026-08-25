---
name: api-route-conversion
description: Implement one approved API Route from .NET in a cloned Go template while preserving its contract and approved observable behavior.
---

# Convert API Route

Work on exactly one approved Route at a time. Start from the full cloned template produced by the orchestrator and follow its Template profile. Preserve the template baseline, including root tooling/configuration, `scripts/`, `sqlc.yaml`, `third_party/`, `go.mod`, `go.sum`, and all `kkpfg-kkpb` libraries. Implement the smallest coherent vertical slice: route registration, request/response types, validation, auth mapping, application logic, datastore access, error mapping, integrations, configuration, and tests.

Preserve the approved HTTP method/path, status codes, headers, serialization, validation, error contract, transaction boundaries, side effects, and idempotency. Use the planned target datastore only. Add migration artifacts when planned, but do not execute data migration.

When behavior cannot be evidenced or mapped, create a Blocker and stop the Route. Keep secrets as references and redact them from generated artifacts. Update the route artifact and Conversion manifest, then produce a diff against the cloned output.

Before completion, inspect only mutable areas (`cmd/`, `db/`, `docs/`, and `internal/`) for unrelated route registrations, handlers, generated models, domain services, external clients, SQL queries, seeds, and migrations. Remove each unrelated item or record why it is in the selected Route's dependency closure. Verify that root tooling, `scripts/`, `sqlc.yaml`, `third_party/`, `go.mod`, `go.sum`, and every `kkpfg-kkpb` dependency remain intact. Completion: the Route compiles in the Template profile, its focused tests exist, its artifact and scope report validate against schema, and every implementation choice points to evidence or an approved decision.
