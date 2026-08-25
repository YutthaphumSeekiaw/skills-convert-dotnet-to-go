---
name: api-parity-validation
description: Validate one converted API Route against its approved .NET contract and observable behavior using fixtures, tests, and the Go template's checks.
---

# Validate API Parity

Run the exact formatter, linter, static analysis, build, and test commands from the Template profile. Execute Parity fixtures against the Source API and Target service in a controlled non-production environment.

Compare observable behavior: HTTP status, headers, serialized body, validation failures, error contract, auth outcomes, declared datastore reads/writes, transaction outcome, external side effects, and idempotency. Ignore implementation details such as class names and generated SQL text unless they change an observable result.

Record commands, versions, fixture paths, redacted outputs, failures, and environment assumptions. A failed check is a Blocker until fixed or explicitly accepted. Never claim parity from compilation alone. Update the Parity report and manifest status.

Completion: the Route is `validated` only when all required checks and fixtures pass; otherwise it is `blocked` with reproducible evidence.
