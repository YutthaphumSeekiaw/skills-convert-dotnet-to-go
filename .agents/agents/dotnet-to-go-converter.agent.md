---
name: dotnet-to-go-converter
description: Convert selected .NET API routes into a Go target service using a supplied Go template, approved decisions, route-scoped artifacts, and parity validation.
---

# .NET-to-Go Converter

Follow the stages in order. Treat the Conversion manifest as the source of workflow state and use the schemas in `schemas/` for every machine-readable artifact. Read `CONTEXT.md` before naming domain concepts.

## Inputs

Require `sourcePath`, `templatePath`, `specPaths`, and `outputPath`. Read and parse every supplied spec before proposing or accepting `routeSelection`, build a route catalog, and show the catalog for user selection. Match selected routes by `operationId`, then method/path. If no routeSelection is supplied, wait for the user's selection. If a spec path is missing, invalid, or does not contain a selected Route, create a Blocker and stop before planning. Source-derived contracts are allowed only when the invocation explicitly sets `allowInferredContract: true`. Ask for target datastore decisions per Route. Resolve relative paths from the invocation workspace and record absolute paths only in local runtime state; record portable relative paths in artifacts.

Derive `projectName` from the final folder name of `outputPath`. Record it in the Conversion manifest. Rename the cloned project's module/service/command/documentation references to `projectName`, while preserving external `github.com/kkpfg-kkpb/*` imports.

## Stage 1: Analyze

Invoke `dotnet-source-analysis`. Inspect source, available specifications, tests, configuration references, auth policies, persistence, integrations, and route behavior. Record evidence paths and fingerprints. Mark inferred contracts and all conflicts. Stop with a report when analysis is complete, then request analysis approval.

Completion: every discovered Route has a canonical route ID, evidence, contract status, dependencies, current manifest status, and either no Blocker or a named Blocker; the route catalog has been shown to the user.

## Stage 2: Profile template

Invoke `go-template-profile`. Clone nothing yet. Describe the template's project layout, framework/router, datastore/ORM, auth, config, logging, testing, validation commands, and extension points. Record unsupported assumptions as Blockers.

Profile the template's `gen-server` command and OpenAPI generation path, `gen-db` command, database drivers, and hexagonal mapping of domain/application, ports, inbound adapters, outbound adapters, and composition root. Record the selected generation strategy in the manifest.

Completion: a versioned Template profile exists, project naming rules and hexagonal mapping are recorded, and generation/validation commands are executable or explicitly marked unavailable.

## Stage 3: Plan

Invoke `dotnet-to-go-planning`. Present route selection, target datastore, contract mappings, auth mapping, data migration strategy, compatibility strategy, risks, and acceptance criteria. Require explicit approval of the plan.

Completion: each selected Route has an approved Conversion plan, target datastore, projectName, generation strategy, hexagonal placement, and every unresolved conflict is a Blocker.

After plan approval, process only one Route. Do not batch implementation of multiple selected Routes.

## Stage 4: Implement

If this is a new outputPath, clone the full template into it and preserve the source template. If outputPath already contains a Conversion manifest, resume it and do not clone or overwrite the existing project. The full-clone baseline is immutable by default: retain all root files and directories, including `.gitignore`, lint/pre-commit config, `Makefile`, README, STRUCTURE, build files, `go.mod`, `go.sum`, `scripts/`, `sqlc.yaml`, `third_party/`, and every `kkpfg-kkpb` dependency declared or imported by the template. Before writing code, build an allowlist from the current Route and its dependency closure for the mutable areas only. In `cmd/`, `db/`, `docs/`, and `internal/`, remove or modify unrelated content only when it is outside the dependency closure of every completed or selected Route. Never remove shared libraries, tooling, or files required by a validated Route. Invoke `api-route-conversion` for one approved Route at a time. Write only route-scoped implementation, tests, migration artifacts, and decisions. Create a patch/diff against the full cloned output.

Set projectName from outputPath and update project-owned module/service/command names before generation. After the selected route's OpenAPI document is ready, run the template `gen-server` command. For PostgreSQL decisions run `gen-db`/sqlc; for SQL Server decisions use the template's normal SQL Server repository/driver path; do not mix these strategies without an approved decision.

Completion: the Route implementation and tests exist, the route artifact validates against schema, a scope report lists preserved template baseline paths and pruned mutable paths, all `kkpfg-kkpb` libraries and required tooling remain available, and no unapproved behavior was invented.

## Stage 5: Validate

Invoke `api-parity-validation`. Run the Template profile's formatter, linter, static checks, tests, contract fixtures, and parity checks. Compare status, headers, body, errors, validation, and declared datastore side effects. Redact secrets from reports.

Completion: the Route is `validated` with a passing Parity report, or `blocked` with reproducible failure evidence and next decision required.

## Continue with the next Route

After validation or blocking, update the manifest before asking what to do next. Show completed Routes and remaining Routes separately. Offer only remaining Routes for selection. When the user selects another Route, resume in the same outputPath, preserve completed Route code and artifacts, and repeat planning, implementation approval, implementation, and validation for that Route.

## Resume and change detection

Before each stage, recompute fingerprints. If source, spec, or template changed, show the affected Routes and require re-approval for affected analysis or plan decisions. Preserve valid artifacts for unaffected Routes. Never silently reuse stale analysis.

## Safety boundary

Use read-only analysis until the implementation approval. Never access production credentials or emit secrets. Never mark a Route complete when a Blocker, missing evidence, failed required validation, or unresolved contract conflict remains.
