# ADR 0002: Deny by default

- **Status:** Accepted
- **Date:** 2026-11-25
- **Milestone:** 1

## Context

An agent can be manipulated (prompt injection) or simply wrong. A rule set that lists what is forbidden (deny list) fails open: any action nobody thought of is allowed. New tools and new parameters appear constantly.

## Decision

Every tool call is **denied unless a rule explicitly allows it**. This also applies to error cases: if identity verification, policy evaluation or any required step fails, the call is denied (fail closed).

## Consequences

- **Positive:**
  - Forgotten cases are refused rather than allowed.
  - Adding a tool requires an explicit, reviewable policy change.
  - Easy to test: "no rule" must mean "deny".
- **Negative:**
  - More upfront work to write allow rules.
  - Legitimate actions are blocked until a rule exists, which can frustrate users.
  - An outage of a required component (PDP, log) blocks all tool calls. This trade-off favours safety over availability.
- **Follow-up:** whether a failure to write the audit log must block the call, **To be decided** (fail-closed leaning).

## Alternatives considered

| Option | Why not chosen |
|---|---|
| Default allow with a deny list | Fails open on anything unforeseen. Unacceptable for a security gateway. |
| Model-based judgement ("ask an LLM if this is safe") | Non-deterministic and itself vulnerable to prompt injection. May complement, never replace, deterministic rules. |
