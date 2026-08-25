---
status: accepted
---

# Preserve Template Baseline

A conversion output is a full clone of the supplied Go template. Root tooling and configuration, build files, `scripts/`, `sqlc.yaml`, `third_party/`, module metadata, and all `kkpfg-kkpb` libraries remain available so the target service keeps the template's build, generation, testing, and integration contract. Route pruning is limited to mutable application areas: `cmd/`, `db/`, `docs/`, and `internal/`, and only after dependency-closure analysis.

## Consequences

A selected route can leave unused template tooling or private dependencies in the output. This is intentional: preserving the template baseline is more important than producing a minimal standalone module, and the scope report must distinguish preserved baseline files from pruned mutable files.
