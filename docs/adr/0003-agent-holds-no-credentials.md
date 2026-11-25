# ADR 0003: The agent holds no credentials

- **Status:** Accepted
- **Date:** 2026-11-25
- **Milestone:** 1 (credential injection), 3 (agent identity)

## Context

If an agent holds an API key or a database password, anything that manipulates the agent can use that secret directly, bypassing any control. Secrets in prompts or agent memory can also leak through outputs and logs.

## Decision

- Tool credentials live **only in the gateway**. The gateway injects them into the outgoing call after the policy decision.
- The agent only holds a **short-lived signed identity** (Ed25519-signed JWT, inspired by SPIFFE), bound to the user it acts for.
- The agent's effective rights are always a subset of that user's rights.
- The network is arranged so that only the gateway can reach the tools.

## Consequences

- **Positive:**
  - A compromised agent cannot reach tools directly: it has neither the secret nor the network path.
  - A stolen agent token is only useful for a short time and can be revoked.
  - Credentials can be rotated without touching agents.
- **Negative:**
  - The gateway becomes a high-value target and a single point of failure. Mitigated by isolation and, at milestone 6, several instances.
  - Tools that require end-user OAuth flows need extra design. **To be decided**.
- **Follow-up:** credential storage, token lifetime and revocation mechanism, **To be decided** (milestones 1 and 3).

## Alternatives considered

| Option | Why not chosen |
|---|---|
| Agent holds scoped API keys | Long-lived secrets in an untrusted component |
| Agent holds the user's own session token | Agent would have *all* the user's rights, with no per-action control |
| Full SPIFFE/SPIRE deployment | Heavier than needed for a prototype. We keep its ideas (short-lived, verifiable identities), not its infrastructure. |
