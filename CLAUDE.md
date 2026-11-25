# Instructions for AI coding agents

This file guides any AI agent (Claude Code or other) working on the Provost repository. Read [README.md](README.md), [ARCHITECTURE.md](ARCHITECTURE.md) and [ROADMAP.md](ROADMAP.md) before making changes.

## Non-negotiable rules

1. **Never weaken a deny rule to make a test pass.** If a test fails because a call is denied, fix the test, the fixture or the allow rule's scope. Never broaden or remove a deny. If unsure, stop and ask.
2. **No real data, no real keys.** All data, users, tokens and keys in code, tests, fixtures, examples and docs are fictitious and clearly labelled as such. Never commit a real secret, even temporarily.
3. **Every feature ships with its tests.** No code is merged without tests covering the allowed path **and** the denied path.
4. **Default deny everywhere.** Any error in identity verification, policy evaluation or approval results in a denial (see [ADR 0002](docs/adr/0002-default-deny.md)).
5. **Structural decisions need an ADR.** Any decision that changes the architecture, a trust boundary, a dependency of significance or a data format requires a new ADR in `docs/adr/` (copy `0000-template.md`) **and** an update of `ARCHITECTURE.md`.
6. **Stay in scope.** Do not add features beyond the current milestone of [ROADMAP.md](ROADMAP.md). Do not invent performance figures.

## Go conventions

- Code formatted with `gofmt`, passes `go vet`. Linter configuration: **To be decided** at milestone 1.
- Standard project layout: binaries in `cmd/`, private packages in `internal/`.
- Errors are returned, wrapped with context (`fmt.Errorf("...: %w", err)`), never ignored. No `panic` outside `main` and test helpers.
- Pass `context.Context` as the first parameter of any function that does I/O or may block.
- Small interfaces, defined where they are consumed.
- Prefer the standard library. Each new dependency must be justified in the pull request.
- Use `crypto/rand`, never `math/rand`, for anything security-related.
- Use constant-time comparison (`crypto/subtle`) for secrets and signatures.
- Never log secrets or tool credentials.

## Tests

- **Table-driven tests** are the default:

  ```go
  tests := []struct {
      name string
      // inputs
      // expected outputs
  }{
      {name: "deny when no rule matches", /* ... */},
      {name: "allow authorized read", /* ... */},
  }
  for _, tt := range tests {
      t.Run(tt.name, func(t *testing.T) { /* ... */ })
  }
  ```

- Test names describe the expected behaviour (`deny when token expired`).
- Run tests with the race detector: `go test -race ./...`.
- End-to-end scenarios proving milestone acceptance criteria go in `test/scenarios/`.

## Repository map

| Path | Content |
|---|---|
| `cmd/provost/` | Gateway binary |
| `cmd/provost-audit/` | Audit verification CLI |
| `internal/gateway/` | MCP interception (PEP) |
| `internal/policy/` | Decision engine (PDP) |
| `internal/identity/` | Agent identities |
| `internal/audit/` | Hash-chained, signed log |
| `internal/approval/` | Human approval |
| `internal/provenance/` | Provenance tags |
| `policies/` | Policy files |
| `deploy/compose/` | Docker Compose demo |
| `examples/demo-agent/` | Demo agent |
| `test/scenarios/` | End-to-end scenarios |
| `docs/` | Threat model and ADRs |

## Before finishing a task

- [ ] Tests added or updated, `go test -race ./...` passes.
- [ ] `gofmt` and `go vet` are clean.
- [ ] No deny rule was weakened.
- [ ] No real data or key was introduced.
- [ ] `ARCHITECTURE.md` and an ADR updated if the change is structural.
- [ ] `docs/threat-model.md` updated if a threat or mitigation changed.
