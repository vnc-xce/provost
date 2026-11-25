# Architecture

> Status: target design. Nothing described here is implemented yet. Items marked **To be decided** are open.

## Components

| Component | Package | Responsibility | Milestone |
|---|---|---|---|
| Gateway (PEP) | `internal/gateway` | MCP proxy. Intercepts every tool call, enforces the decision, injects tool credentials, forwards allowed calls. | 1 |
| Policy engine (PDP) | `internal/policy` | Evaluates deterministic rules on identity, tool and parameters. Default deny. | 2 |
| Identity | `internal/identity` | Issues and verifies short-lived signed agent identities bound to a user. Rotation and revocation. | 3 |
| Audit log | `internal/audit` | Appends requests, decisions and results to a hash-chained, signed log. | 1 (basic), 4 (tamper-evident) |
| Human approval | `internal/approval` | Holds irreversible or outbound actions until a human approves or rejects them. | 5 |
| Provenance | `internal/provenance` | Tags data from untrusted sources and prevents it from choosing the target of a sensitive action. | 5 |
| Gateway binary | `cmd/provost` | Entry point of the gateway. | 1 |
| Audit CLI | `cmd/provost-audit` | Verifies the integrity of an audit log. | 4 |

## Component diagram

```mermaid
flowchart TB
    subgraph CP[Control plane]
        GIT[(Policies in Git)]
    end

    subgraph DP[Data plane: N gateway instances]
        direction TB
        PEP[gateway: PEP]
        PDP[policy: PDP]
        ID[identity]
        PROV[provenance]
        APR[approval]
        AUD[audit]
        CRED[(Tool credentials)]
    end

    AGENT[AI agent] -->|MCP + signed identity| PEP
    PEP --> ID
    PEP --> PROV
    PEP --> PDP
    PEP --> APR
    PEP --> AUD
    CRED -.-> PEP
    PEP --> TOOLS[MCP tools]
    GIT -.->|distribution: To be decided| PDP
    AUD --> CLI[provost-audit CLI]
    APR <--> HUMAN([Approver])
```

## Life of a tool call

```mermaid
sequenceDiagram
    autonumber
    participant Ag as AI agent
    participant G as Gateway (PEP)
    participant I as Identity
    participant Pv as Provenance
    participant P as Policy (PDP)
    participant H as Human approver
    participant L as Audit log
    participant T as MCP tool

    Ag->>G: tools/call (tool, params) + signed identity
    G->>L: record request
    G->>I: verify signature, expiry, revocation
    alt identity invalid
        G->>L: record denial
        G-->>Ag: deny
    end
    G->>Pv: get provenance labels of params
    G->>P: decide(identity, user, tool, params, labels)
    P-->>G: allow / deny / require approval
    alt deny (default)
        G->>L: record denial
        G-->>Ag: deny
    else require approval
        G->>H: approval request
        H-->>G: approve / reject
        G->>L: record human decision
    end
    G->>G: inject tool credential
    G->>T: forward call
    T-->>G: result
    G->>Pv: tag result as untrusted if needed
    G->>L: record result
    G-->>Ag: result
```

Notes:

- The agent never sees the tool credential. It is added by the gateway after the decision (step "inject tool credential").
- A refused call never reaches the tool.
- The behaviour when the audit log cannot be written is **To be decided** (fail-closed is the default leaning, see [ADR 0002](docs/adr/0002-default-deny.md)).

## Trust boundaries

```mermaid
flowchart LR
    subgraph Untrusted
        A[AI agent and its model]
        EXT[External content:<br/>web pages, documents, emails]
    end
    subgraph Trusted[Trusted: Provost]
        G[Gateway + PDP + log]
    end
    subgraph Protected[Protected network]
        T[Tools and their credentials]
    end
    EXT -. may contain injections .-> A
    A -->|boundary 1| G
    G -->|boundary 2| T
    GIT[(Policy repo)] -->|boundary 3| G
    HUM([Human approver]) -->|boundary 4| G
```

| # | Boundary | What crosses it | Guarantee expected |
|---|---|---|---|
| 1 | Agent → gateway | MCP calls, signed identity | Every call authenticated and evaluated |
| 2 | Gateway → tools | Allowed calls with credentials | Only the gateway can reach tools (network isolation) |
| 3 | Policy repo → gateway | Policies | Policies are versioned and reviewed. Integrity of distribution: **To be decided** |
| 4 | Approver → gateway | Approval decisions | Approver authentication: **To be decided** |

The full analysis is in [docs/threat-model.md](docs/threat-model.md).

## Repository layout

| Folder | Role |
|---|---|
| `cmd/provost/` | Gateway binary (`main` only, wiring of `internal/` packages) |
| `cmd/provost-audit/` | Audit log verification CLI |
| `internal/gateway/` | MCP interception, enforcement, credential injection |
| `internal/policy/` | PDP: rule loading and evaluation |
| `internal/identity/` | Agent identity tokens: issuance, verification, rotation, revocation |
| `internal/audit/` | Hash-chained, signed append-only log |
| `internal/approval/` | Human-in-the-loop workflow |
| `internal/provenance/` | Data provenance labels and propagation |
| `policies/` | Policy files (control plane, versioned in Git) |
| `deploy/compose/` | Docker Compose demo on an isolated network |
| `examples/demo-agent/` | Demo agent used by the scenarios |
| `test/scenarios/` | End-to-end scenarios proving each milestone's acceptance criteria |
| `docs/adr/` | Architecture Decision Records |

Go's `internal/` directory prevents other modules from importing these packages. The public surface of Provost is its binaries and its policy format, not a Go library.

## Open choices

| Topic | Status |
|---|---|
| Policy language: Cedar or OPA/Rego | **To be decided** at milestone 2 ([ADR 0005](docs/adr/0005-policy-language-cedar-or-opa.md)) |
| MCP transports supported (stdio, Streamable HTTP) | **To be decided** at milestone 1 |
| Where tool credentials are stored (file, secret manager) | **To be decided** |
| Identity token format details (JWT claims, lifetime) | **To be decided** at milestone 3 |
| Revocation mechanism | **To be decided** at milestone 3 |
| Audit log storage backend and key management | **To be decided** at milestone 4 |
| Approval channel (CLI, web UI, chat) | **To be decided** at milestone 5 |
| How provenance labels are propagated through the agent | **To be decided** at milestone 5 |
| Policy distribution to instances (pull, push, signed bundles) | **To be decided** at milestone 6 |
| Behaviour when the log or the PDP is unavailable | **To be decided** (fail-closed leaning) |
