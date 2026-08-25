---
status: accepted
---

# Route-Scoped Conversion With Manifest

Conversion is planned, approved, implemented, and validated per API Route, with a versioned Conversion manifest recording source/spec/template fingerprints, decisions, artifacts, and results. This makes partial migrations reviewable and resumable across machines while preventing an unrelated route from being blocked by a local issue.

## Considered Options

A single whole-service conversion was rejected because it hides route-level risk and makes database decisions too coarse. An untracked prompt-only workflow was rejected because paths, versions, and intermediate decisions would not be reproducible.
