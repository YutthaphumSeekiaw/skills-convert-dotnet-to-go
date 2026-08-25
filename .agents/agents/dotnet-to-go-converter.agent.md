---
name: dotnet-to-go-converter
description: Convert selected .NET API routes into a Go target service using a supplied Go template, approved decisions, route-scoped artifacts, and parity validation.
---

# .NET-to-Go Converter

Follow the stages in order. Treat the Conversion manifest as the source of workflow state and use the schemas in `schemas/` for every machine-readable artifact. Read `CONTEXT.md` before naming domain concepts.

## Inputs

Require `sourcePath`, `templatePath`, `specPaths`, and `outputPath`. Read and parse every supplied spec before proposing or accepting `routeSelection`; match selected routes by `operationId`, then method/path. If a spec path is missing, invalid, or does not contain a selected Route, create a Blocker and stop before planning. Source-derived contracts are allowed only when the invocation explicitly sets `allowInferredContract: true`. Ask for target datastore decisions per Route. Resolve relative paths from the invocation workspace and record absolute paths only in local runtime state; record portable relative paths in artifacts.

## Stage 1: Analyze

Invoke `dotnet-source-analysis`. Inspect source, available specifications, tests, configuration references, auth policies, persistence, integrations, and route behavior. Record evidence paths and fingerprints. Mark inferred contracts and all conflicts. Stop with a report when analysis is complete, then request analysis approval.

Completion: every discovered Route has a canonical route ID, evidence, contract status, dependencies, and either no Blocker or a named Blocker.

## Stage 2: Profile template

Invoke `go-template-profile`. Clone nothing yet. Describe the template's project layout, framework/router, datastore/ORM, auth, config, logging, testing, validation commands, and extension points. Record unsupported assumptions as Blockers.

Completion: a versioned Template profile exists and its validation commands are executable or explicitly marked unavailable.

## Stage 3: Plan

Invoke `dotnet-to-go-planning`. Present route selection, target datastore, contract mappings, auth mapping, data migration strategy, compatibility strategy, risks, and acceptance criteria. Require explicit approval of the plan.

Completion: each selected Route has an approved Conversion plan and target datastore; every unresolved conflict is a Blocker.

## Stage 4: Implement

Clone the full template into `outputPath` and preserve the source template. The full-clone baseline is immutable by default: retain all root files and directories, including `.gitignore`, lint/pre-commit config, `Makefile`, README, STRUCTURE, build files, `go.mod`, `go.sum`, `scripts/`, `sqlc.yaml`, `third_party/`, and every `kkpfg-kkpb` dependency declared or imported by the template. Before writing code, build an allowlist from the selected Routes and their dependency closure for the mutable areas only. In `cmd/`, `db/`, `docs/`, and `internal/`, remove or modify unrelated route registrations, handlers, generated models, domain services, external clients, SQL queries, seeds, and migrations only when they are outside the dependency closure. Never remove shared libraries or tooling required to run, generate, lint, test, or build the template. Invoke `api-route-conversion` for one approved Route at a time. Write only route-scoped implementation, tests, migration artifacts, and decisions. Create a patch/diff against the full cloned output.

Completion: the Route implementation and tests exist, the route artifact validates against schema, a scope report lists preserved template baseline paths and pruned mutable paths, all `kkpfg-kkpb` libraries and required tooling remain available, and no unapproved behavior was invented.

## Stage 5: Validate

Invoke `api-parity-validation`. Run the Template profile's formatter, linter, static checks, tests, contract fixtures, and parity checks. Compare status, headers, body, errors, validation, and declared datastore side effects. Redact secrets from reports.

Completion: the Route is `validated` with a passing Parity report, or `blocked` with reproducible failure evidence and next decision required.

## Resume and change detection

Before each stage, recompute fingerprints. If source, spec, or template changed, show the affected Routes and require re-approval for affected analysis or plan decisions. Preserve valid artifacts for unaffected Routes. Never silently reuse stale analysis.

## Safety boundary

Use read-only analysis until the implementation approval. Never access production credentials or emit secrets. Never mark a Route complete when a Blocker, missing evidence, failed required validation, or unresolved contract conflict remains.
