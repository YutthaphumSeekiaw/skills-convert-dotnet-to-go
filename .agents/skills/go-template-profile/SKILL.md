---
name: go-template-profile
description: Analyze a Go template path and produce the conventions and executable validation profile required to reuse it during .NET API conversion.
---

# Profile Go Template

Read the template without changing it. Identify module path, directory/package layout, router/framework, dependency injection, configuration, logging, error handling, auth middleware, datastore and migration conventions, API schema generation, testing patterns, formatter/linter/static checks, build commands, and extension points.

Record exact evidence paths and commands. Detect generated files, local-only assumptions, missing dependencies, and conventions that a converted Route must follow. Do not invent a framework convention when the template does not establish one; record the gap as a Blocker for approval.

Classify paths into two groups: the preserved template baseline (all root tooling/configuration, `scripts/`, `sqlc.yaml`, `third_party/`, `go.mod`, `go.sum`, and `kkpfg-kkpb` libraries) and mutable route areas (`cmd/`, `db/`, `docs/`, and `internal/`). The baseline is retained even when the selected Route does not use every helper directly; only mutable content may be pruned after dependency analysis.

Record the project naming convention: the final folder name of `outputPath` is `projectName`; list every project-owned name that must be renamed and distinguish it from external `github.com/kkpfg-kkpb/*` imports. Record the exact `gen-server` and `gen-db` commands, OpenAPI source path, database drivers, and whether PostgreSQL uses sqlc or SQL Server uses the template's normal database access path. Map the template to hexagonal roles: domain/application, ports, inbound adapters, outbound adapters, and composition root.

Produce a versioned Template profile consumed by planning and conversion. The profile must distinguish existing SQL Server support from a new target datastore and state how each is tested.

Completion: all required template conventions and validation commands are evidenced, portable paths are recorded, and unresolved gaps are named.
