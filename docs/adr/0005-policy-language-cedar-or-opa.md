# ADR 0005: Policy language, Cedar or OPA/Rego

- **Status:** Proposed
- **Date:** 2026-11-25
- **Milestone:** 2

## Context

The PDP must evaluate deterministic rules on the agent identity, the delegating user, the tool and its parameters (for example, deny SQL containing `DROP`). Rules are versioned in Git and reviewed by humans, so readability matters as much as expressiveness. The engine must be embeddable in a Go binary.

## Options

### Cedar

Policy language created by AWS, open source (Apache 2.0). Policies are `permit` / `forbid` statements over principal, action, resource and context.

- Default deny and "forbid wins over permit" are built into the language.
- Designed for analysis: policies can be validated against a schema.
- Concise and readable for authorization rules.
- Reference implementation in Rust. A Go implementation exists (`cedar-go`). Its feature coverage compared with the Rust version must be checked.
- Less suited to arbitrary data processing (e.g. parsing an SQL string).

### OPA / Rego

General-purpose policy engine, CNCF graduated project, written in Go.

- Native Go library: embedding is straightforward.
- Very expressive: can inspect any JSON input, with rich built-in functions.
- Large ecosystem (Kubernetes, Envoy, Terraform).
- Rego has a steeper learning curve. Expressiveness makes policies harder to review.
- Default deny must be written explicitly (`default allow := false`).

## Comparison

| Criterion | Cedar | OPA / Rego |
|---|---|---|
| Default deny | Built in | By convention |
| Readability for reviewers | High | Medium |
| Expressiveness | Focused on authorization | General purpose |
| Go integration | `cedar-go` (coverage to check) | Native |
| Schema validation | Built in | Optional |
| Inspecting parameter content (SQL) | Limited, may need pre-processing in Go | Possible in Rego |

No performance figures are claimed here. If needed, they will be measured on Provost's own policies.

## Decision

**To be decided at milestone 2.**

Proposed method: write the same two milestone 2 policies (deny SQL containing `DROP`, allow an authorized read) in both languages, then compare readability, test ergonomics and Go integration.

## Consequences

To be written once the decision is made. Either way, the PDP will sit behind a Go interface in `internal/policy` so that the gateway does not depend on the engine.
