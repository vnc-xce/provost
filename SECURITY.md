# Security policy

## ⚠️ Prototype, not for production

Provost is an **educational prototype**. It has not been audited and must **not** be used to protect real systems, real data or real credentials. All data and keys shipped with the project are fictitious.

## Supported versions

No release exists yet. Only the `main` branch is considered.

## Reporting a vulnerability

Please **do not** open a public issue for a security problem.

- Private reporting channel: **To be decided** (for example, GitHub private vulnerability reporting or a dedicated address).
- Include: affected component, steps to reproduce, expected and observed behaviour, and impact.
- Acknowledgement and fix timelines: **To be decided**. As a prototype maintained on a best-effort basis, no response time is guaranteed.

## Scope

Of particular interest:

- Any way for an agent to reach a tool without going through the gateway.
- Any call that is allowed when no rule allows it (default deny violation).
- Forged, expired or revoked identities being accepted.
- Audit log modifications not detected by `provost-audit`.
- Untrusted data choosing the target of a sensitive action.

See [docs/threat-model.md](docs/threat-model.md) for the threat model.
