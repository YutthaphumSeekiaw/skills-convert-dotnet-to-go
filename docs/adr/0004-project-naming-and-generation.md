---
status: accepted
---

# Project Naming And Generation

Each conversion output uses the final folder name of `outputPath` as its projectName for project-owned module, service, command, and documentation names. Generated API code must come from the selected OpenAPI document through the template's `gen-server` command; PostgreSQL uses the template's `gen-db`/sqlc workflow, while SQL Server uses the template's normal database access path. External `github.com/kkpfg-kkpb/*` dependency names remain unchanged.

The converted code follows the template's hexagonal architecture, with domain/application logic separated from ports, inbound adapters, outbound adapters, and the composition root. This keeps template conventions and generation workflows intact while allowing route-level pruning in mutable application areas.