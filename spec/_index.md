---
title: Specification
weight: 1
cascade:
  type: docs
---

# Agent Gossip Protocol Specification

**Version:** 0.1.0 (draft)
**Status:** Draft — not yet ratified.

The Agent Gossip Protocol (AGP) is a **shared-context layer** for multi-agent
environments. Agents gossip the availability, health, and observed quality of
**action paths** — external agent applications, inherent skills, and MCP
servers — so that each agent can plan how to accomplish its tasks against
fresh, decentralized knowledge of its environment.

AGP carries *planning context*, not work: when the environment changes — a
similar agent becomes available, a path degrades, a capability is withdrawn —
that context propagates by gossip, and each agent independently re-evaluates
its options.

This specification describes the *what* and the *how* of the protocol on the
wire. It does not mandate any particular implementation, runtime, or
programming language.

## Scope and Non-Goals

AGP is the context layer **only**. The following are explicit non-goals:

- **Task delegation or execution.** Agents act on gossiped context through
  other channels (e.g., MCP, A2A, or native invocation). AGP never carries a
  task.
- **Orchestration.** AGP does not assign work, schedule agents, or arbitrate
  between them.
- **Global ranking.** Agents gossip *subjective observations*; every receiver
  computes its own ranking locally. There is no protocol-level "true" rank.

> **Deferred:** Gossiping *user presence* — where a user is active elsewhere
> in the known environment, as a planning signal — is intentionally deferred
> and will be specified in a future SEP. Hooks are reserved in the message
> model and threat model.

## Conventions and Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they
appear in all capitals, as shown here.

## Document Set

| Document | Purpose |
| --- | --- |
| [Core](core/) | The definitive, RFC-style protocol rules. |
| [Discovery](discovery/) | How agents find and verify peers. |
| [Security](security/) | Threat models (e.g., Sybil attacks, state poisoning). |
| [Transport](transport/) | Wire-level transport expectations. |

## Machine-Readable Schemas

Normative payload definitions are published as JSON Schema and served at
stable, versioned URLs:

- `https://agentgossipprotocol.org/schemas/v1/message.schema.json`
- `https://agentgossipprotocol.org/schemas/v1/state.schema.json`
- `https://agentgossipprotocol.org/schemas/v1/error.schema.json`

Where prose and schema disagree, the **schema is normative** for payload
structure; the prose is normative for protocol behavior.

## Versioning

This specification uses [Semantic Versioning](https://semver.org/). Changes
are proposed and ratified through the process described in the
[proposals](../proposals/) directory.

> **TODO:** Ratify version 1.0.0. Everything under 1.0.0 is subject to
> breaking change without notice.
