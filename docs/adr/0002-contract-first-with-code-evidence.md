---
status: accepted
---

# Contract-First With Code Evidence

OpenAPI or other declared specifications define the public contract when available, while source code supplies behavioral evidence. A conflict is recorded as a Blocker and requires explicit resolution before conversion; the agent must not silently choose one interpretation.

## Consequences

The workflow can support projects without OpenAPI by deriving a provisional contract from code, but that contract must be labeled as inferred and approved before implementation.
