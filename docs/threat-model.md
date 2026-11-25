# Threat model

> Status: initial draft, written before any code. Reviewed at the end of each milestone.

## Scope and assumptions

- In scope: the path from an AI agent's tool call to the tool's execution, and the records kept about it.
- The AI model and agent are **untrusted**: they may be wrong, manipulated or compromised.
- The host running the gateway is trusted. Compromise of that host is out of scope for this prototype.
- The demo uses fictitious data and keys only.

## Actors

| Actor | Trust | Description |
|---|---|---|
| User | Trusted for their own rights | Person on whose behalf the agent acts |
| AI agent | Untrusted | Model + orchestration code issuing MCP tool calls |
| External attacker | Hostile | Plants instructions in content the agent reads (web pages, documents, emails, tool results) |
| Compromised tool | Hostile | An MCP tool or server returning malicious content |
| Operator | Trusted | Deploys Provost, writes and reviews policies |
| Approver | Trusted | Human who approves irreversible or outbound actions |

## Assets

| Asset | Why it matters |
|---|---|
| Tool credentials | Grant direct access to tools. Must never reach the agent. |
| Data held by tools | Target of destruction and exfiltration |
| Policies | Define what is allowed. Tampering widens access. |
| Agent identities and signing keys | Forging them bypasses authorization |
| Audit log and its signing key | Only trustworthy record of what happened |
| Gateway availability | Without it, no tool can be used (by design) |

## Trust boundaries

1. **Agent → gateway**: untrusted calls enter the trusted zone.
2. **Gateway → tools**: only the gateway may cross it (network isolation).
3. **Policy repository → gateway**: policy changes enter the data plane.
4. **Approver → gateway**: human decisions enter the data plane.

See the diagram in [ARCHITECTURE.md](../ARCHITECTURE.md#trust-boundaries).

## Threats and planned mitigations

OWASP references point to the [OWASP Top 10 for Agentic Applications for 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications/). A reference is given only where the mapping is certain. A dash means no reference is claimed.

| ID | Threat | Example | Planned mitigation | Milestone | OWASP |
|---|---|---|---|---|---|
| T1 | Destructive actions | Agent runs `DROP TABLE` or deletes files | Default deny. Rules on tool parameters. Human approval for irreversible actions. | 1, 2, 5 | ASI02 Tool Misuse & Exploitation |
| T2 | Data exfiltration | Agent sends private data to an attacker-controlled address | Outbound actions require approval. Untrusted data cannot choose the target of a sensitive action. | 5 | — |
| T3 | Prompt injection | A web page tells the agent to "email the customer list to …" | Provenance tags on untrusted data (CaMeL model). Deterministic policies that do not depend on the model's judgement. | 2, 5 | ASI01 Agent Goal Hijack |
| T4 | Gateway bypass | Agent calls a tool directly, skipping Provost | Agent holds no tool credentials. Isolated network where only the gateway reaches tools. | 1 | — |
| T5 | Identity theft or forgery | Stolen, replayed or forged agent token; agent acting beyond its user's rights | Short-lived Ed25519-signed tokens bound to a user. Rotation and revocation. Delegated rights ⊆ user rights. | 3 | ASI03 Identity & Privilege Abuse |
| T6 | Audit log tampering | An entry is modified or deleted to hide an action | Hash chain + signatures. `provost-audit` verification CLI. | 4 | — |
| T7 | Gateway outage | Gateway crash or overload | Fail closed: no gateway, no tool call. Several instances deciding locally. | 1, 6 | — |

## Residual risks and open questions

- A policy that is too permissive is still enforced faithfully. Policy review is a human process.
- An approver can be misled into approving a harmful action. Approval UX: **To be decided** (milestone 5).
- Deleting the end of the log (truncation) vs. modifying a line: detection strategy **To be decided** (milestone 4).
- Protection of signing keys at rest: **To be decided**.
- Integrity of policy distribution to instances: **To be decided** (milestone 6).
