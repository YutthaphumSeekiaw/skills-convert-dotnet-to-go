# .NET-to-Go Conversion Agents

This repository contains portable agent and skill instructions for converting selected .NET API routes into a Go target service.

## Install on another machine

Copy or clone this repository, then expose the repository's `.agents` directory to the agent runtime used by that machine. Keep the instruction files, schemas, and `CONTEXT.md` together. Do not copy secrets or production configuration into this repository.

The workflow is prompt-driven and produces deterministic artifacts through explicit schemas, input fingerprints, evidence references, approvals, and validation commands. Model output itself is not assumed to be byte-for-byte deterministic.

## Run

Invoke the `dotnet-to-go-converter` agent and provide:

1. `sourcePath`: the .NET API path.
2. `templatePath`: the Go template path.
3. `specPaths`: optional OpenAPI or documentation paths.
4. `outputPath`: a new workspace for the cloned template and route artifacts.
5. `routeSelection`: route IDs or a request to propose a selection.
6. `targetDatastore` decisions, including `sql-server-existing` or a named new datastore.

The agent pauses for approval after source analysis, conversion planning, and immediately before implementation. A route is complete only when its implementation, tests, parity fixtures, parity report, and validation results are recorded in the manifest.

## Layout

- `.agents/agents/`: orchestrator agent.
- `.agents/skills/`: stage-specific skills.
- `schemas/`: machine-readable manifest and artifact contracts.
- `docs/adr/`: durable design decisions.
