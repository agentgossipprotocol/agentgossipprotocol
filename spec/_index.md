---
title: Specification
weight: 1
cascade:
  type: docs
---

# Agent Gossip Protocol Specification

**Version:** 0.1.0 (draft)
**Status:** Draft — not yet ratified.

The Agent Gossip Protocol (AGP) is a **transport-independent protocol** that
propagates **expiring, attributable operational evidence** among agents:
self-reported runtime state, directly measured route quality, authorized
evaluation scores, and unverified peer referrals. Each agent uses that
evidence to choose more effectively among **action paths** — external agent
applications, inherent skills, and MCP servers — that already exist and are
already authorized.

AGP is defined in two layers. The **core protocol** (record model, interest
model, `Sync` anti-entropy, security requirements) is independent of any
host protocol. A **binding** maps that core onto a concrete transport; the
normative binding in this specification is the [A2A Binding](/spec/binding-a2a/),
which is the preferred and best-integrated way for A2A-compatible agents to
run AGP. The core deliberately does not depend on any A2A object (`Task`,
`Message`, `Artifact`, or `AgentCard`); those appear only in the binding.

AGP does not discover what exists, decide who may talk to whom, or carry
work. Those questions are answered by the host protocol's discovery (Agent
Cards and A2A discovery under the A2A binding), by authorization policy, and
by MCP/A2A/native invocation, respectively. AGP answers a narrower question:
*given what officially exists and what I'm allowed to use, what has recently
been observed about it?*

This specification describes the *what* and the *how* of the protocol on the
wire. It does not mandate any particular implementation, runtime, or
programming language, and it deliberately does not prescribe a policy engine,
storage layer, or ranking formula.

> **Naming note:** "AGP" here is this project's Agent Gossip Protocol. It is
> unrelated to the experimental, non-normative "Agent Gateway Protocol"
> sample extension published under `a2a-samples` (a BGP-inspired hierarchical
> routing layer). Because A2A extensions are identified by URI, not by
> acronym, the two never collide on the wire: AGP's A2A binding is always
> addressed as `https://agentgossipprotocol.org/a2a/extensions/gossip/v1`
> (see [A2A Binding §1](/spec/binding-a2a/)). Avoid the bare acronym "AGP" in
> contexts where either protocol could plausibly be meant.

## Scope and Non-Goals

AGP is an **evidence layer only**. The following are explicit non-goals:

- **Discovery.** AGP does not decide what agents, capabilities, or endpoints
  officially exist. That is the job of Agent Cards and A2A discovery
  ([Discovery](/spec/discovery/)). A gossiped `peer.referral.v1` record is a hint,
  never a fact, until the receiver independently fetches and validates the
  referenced Agent Card.
- **Authorization or identity.** AGP MUST NOT create permissions, establish
  identity, or modify an Agent Card. Whether two agents may communicate, or
  whether a given piece of evidence may even be exchanged between them, is
  decided entirely by policy outside this protocol ([Security §3](/spec/security/)).
- **Configuration.** Authoritative operational parameters (SLAs, quotas,
  routing tables) are configuration, not evidence. Gossip may inform a
  preference; it never overrides configuration.
- **Task delegation or execution.** Agents act on gossiped evidence through
  other channels (e.g., MCP, A2A, or native invocation). AGP never carries a
  task, a prompt, tool input/output, or business data.
- **Orchestration.** AGP does not assign work, schedule agents, or arbitrate
  between them.
- **Global ranking or truth.** Every record is a *subjective, attributed
  observation* with its own evidence and freshness window; every receiver
  computes its own ranking locally. There is no protocol-level "true" rank
  and no unqualified assertions like "Agent B is unreliable."

> **Excluded from v1:** User activity/usage profiling, prompts or task
> histories, tool inputs/outputs, business records, raw exception strings,
> chain-of-thought or model-generated explanations, secrets/credentials, and
> free-form reputation assertions. If an agent acts on behalf of a user, user
> identity is handled through on-behalf-of authorization and local policy —
> never propagated as gossip. See [Core §3.6](/spec/core/) and
> [Security §9](/spec/security/).

## Conventions and Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they
appear in all capitals, as shown here.

## Document Set

**Core protocol** (transport-independent):

| Document | Purpose |
| --- | --- |
| [Core](/spec/core/) | The record model, v1 record types, exclusions, deduplication, and conformance rules. |
| [Discovery](/spec/discovery/) | How agents bootstrap and select gossip peers. |
| [Security](/spec/security/) | Threat model, record signing, and the selector/authorization intersection. |
| [Transport](/spec/transport/) | The `Sync` anti-entropy operation and the abstract contract a binding MUST satisfy. |

**Bindings:**

| Document | Purpose |
| --- | --- |
| [A2A Binding](/spec/binding-a2a/) | AGP as an A2A extension: Agent Card declaration, negotiation, identity/auth mapping, and the gRPC / HTTP+JSON realizations of `Sync`. |

## Machine-Readable Schemas

Normative payload definitions are published as JSON Schema and served at
stable, versioned URLs:

- `https://agentgossipprotocol.org/schemas/v1/record.schema.json` — the immutable, signed gossip record.
- `https://agentgossipprotocol.org/schemas/v1/frame.schema.json` — the `GossipFrame` envelope exchanged during `Sync`.
- `https://agentgossipprotocol.org/schemas/v1/selector.schema.json` — interest selectors used for scoping.
- `https://agentgossipprotocol.org/schemas/v1/error.schema.json` — the normalized error model.

Where prose and schema disagree, the **schema is normative** for payload
structure; the prose is normative for protocol behavior.

## Versioning

The **core protocol** and each **binding** are versioned independently,
each using [Semantic Versioning](https://semver.org/): a change to the A2A
binding does not by itself bump the core version, and a backward-compatible
core change may only widen the core versions a binding advertises (see
[A2A Binding §7](/spec/binding-a2a/)). Changes are proposed and ratified through
the process described in the [proposals](../proposals/) directory.

> **TODO:** Ratify core version 1.0.0 and A2A binding v1. Everything under
> 1.0.0 is subject to breaking change without notice.
