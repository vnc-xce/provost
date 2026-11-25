# ADR 0001: Use Go as the implementation language

- **Status:** Accepted
- **Date:** 2026-11-25
- **Milestone:** 1

## Context

Provost is a network proxy sitting on the critical path of every tool call. It must be easy to audit, easy to deploy, and safe to run concurrently. It is also an educational project.

## Decision

Implement Provost in Go.

## Consequences

- **Positive:**
  - Single static binary per command: simple Docker images and deployment.
  - Concurrency built in (goroutines, channels), well suited to a proxy.
  - Strong standard library for networking, HTTP, JSON and cryptography (`crypto/ed25519`, `crypto/sha256`).
  - Memory safe (garbage collected), which removes a class of vulnerabilities common in C/C++.
  - Common language of cloud-native security tooling (OPA, SPIRE, Kubernetes), which eases integration and learning.
  - Simple language: code is easy to review, which matters for a security component.
- **Negative:**
  - Garbage collection adds latency variance compared with Rust or C++. Its impact is to be measured at milestone 6.
  - Less expressive type system than Rust: some invariants must be enforced by tests rather than by the compiler.
- **Follow-up:** minimum Go version, **To be decided** at milestone 1.

## Alternatives considered

| Option | Why not chosen |
|---|---|
| Rust | Stronger guarantees, but steeper learning curve and slower iteration for an educational prototype |
| Python | Rich AI ecosystem, but weaker deployment story and performance for a proxy on the hot path |
| TypeScript / Node.js | Official MCP SDK available, but runtime dependencies and weaker static guarantees |
