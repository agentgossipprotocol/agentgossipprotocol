---
title: Core
weight: 1
---

# Core Protocol

**Status:** Draft

This document defines the core, normative rules of the Agent Gossip Protocol
(AGP). It is implementation-agnostic: it describes the messages exchanged on
the wire and the behavior peers MUST exhibit, never how a given peer is built.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as
described in RFC 2119 and RFC 8174.

## 1. Overview

In a multi-agent environment, an agent determines a course of action for its
tasks by choosing among **action paths**: external agent applications, its own
inherent skills, or MCP servers. That choice is only as good as the agent's
knowledge of its environment — which paths exist, which are degraded, and
which alternatives have appeared.

AGP keeps that knowledge fresh. Agents gossip two kinds of context:

1. **Capability announcements** — "action path *P* offers capability *C*".
2. **Observations** — "at time *T*, I observed path *P* to be available /
   degraded / unavailable, with these measurements".

Receivers accumulate this context and use it to plan. AGP itself never carries
a task, a delegation, or an instruction: it is a context layer that
*complements* execution protocols such as MCP and A2A rather than competing
with them.

## 2. Terminology

- **Agent** — an autonomous participant that both consumes gossiped context
  for planning and produces context for others.
- **Peer** — an agent to which another agent maintains a direct gossip
  connection.
- **Action Path** — a means by which an agent can accomplish (part of) a
  task. An action path is one of: an **external agent application**, an
  **inherent skill** of the agent itself, or an **MCP server**.
- **Capability** — a declared function that an action path offers (e.g.,
  "summarize documents", "query inventory").
- **Observation** — a timestamped, *subjective* report by one agent about one
  action path: its availability, degradation, and measured quality.
- **Rank** — a preference ordering over action paths for a given task. Rank
  is **computed locally** by each agent from the observations it holds. Rank
  is NOT gossiped and has no wire representation.
- **Message** — a unit of gossip conforming to `message.schema.json`.
- **Context State** — an agent's accumulated view of capabilities and
  observations, conforming to `state.schema.json`.

## 3. Context Model

### 3.1 Action Paths

An action path is identified by its `kind` (`agent`, `skill`, or `mcp`) and a
stable identifier. Identifiers MUST be stable across restarts of the path so
that observations accumulate meaningfully.

> **TODO:** Define the identifier format and (for `agent`/`mcp` kinds) the
> address/endpoint representation.

### 3.2 Capability Announcements

An agent MAY announce capabilities for action paths it controls or hosts.
An announcement binds a capability to an action path.

- An agent MUST NOT announce capabilities for action paths it does not
  control. (Reporting on *other* agents' paths is done through observations,
  never announcements.)
- Announcements MUST be attributed to the announcing agent.
- An agent SHOULD withdraw (via a `withdraw` message) capabilities that are
  no longer offered.

### 3.3 Observations

Observations are the mechanism by which "degraded or low-ranked path"
knowledge spreads. They are deliberately **subjective**:

- An observation MUST identify the observing agent (`observer`), the observed
  action path, a `status` of `available`, `degraded`, or `unavailable`, and
  the time of observation (`observedAt`).
- An observation MAY carry quantitative metrics (e.g., latency, success
  rate).
- A receiver MUST treat every observation as a *claim by its observer*, not
  as fact. Receivers MUST NOT merge observations from different observers
  into a single record; provenance is preserved.
- Receivers compute ranking locally from the observations they hold. How a
  receiver weighs observers, recency, and metrics is implementation-defined.

### 3.4 Freshness

Stale context is worse than no context.

- Every capability announcement and observation MUST carry a timestamp.
- Producers SHOULD attach a TTL; receivers MUST discard context whose TTL has
  expired.

> **TODO:** Define default TTLs and the maximum permitted clock skew.

### 3.5 User Presence (Deferred)

> **Deferred:** A future SEP will specify gossiping *user presence* — an
> agent's knowledge of where a user identity is active elsewhere in the
> environment, as an input to planning. The `presence` message type is
> reserved for this purpose (§4). It MUST NOT be used until specified; the
> privacy requirements in [Security §7](security/) MUST be resolved first.

## 4. Message Model

Every message MUST validate against the base gossip payload schema
(`message.schema.json`, version 1).

| Type | Payload | Purpose |
| --- | --- | --- |
| `announce` | Capability | Declare a capability offered by an action path. |
| `observe` | Observation | Report a subjective observation of an action path. |
| `withdraw` | Capability reference | Retract a previous announcement. |
| `presence` | *(reserved)* | Reserved for user presence. Deferred to a future SEP; MUST NOT be sent. |

> **TODO:** Define the encoding rules and de-duplication window for message
> `id`s.

## 5. Gossip Semantics

> **TODO:** Define fan-out, rounds, anti-entropy, and convergence
> expectations. Specify what a conforming peer MUST, SHOULD, and MAY do on
> receipt of each message type — including forwarding rules for observations
> it did not author (forwarded observations MUST preserve the original
> `observer` and `observedAt`).

## 6. State Propagation

> **TODO:** Define how context state is reconciled between peers
> (anti-entropy over `state.schema.json`), so that a newly joined agent can
> acquire the current capability and observation set.

## 7. Error Handling

Peers MUST report protocol errors using the standardized error object defined
in `error.schema.json`.

> **TODO:** Enumerate error conditions and the required peer behavior for each.

## 8. Conformance

An implementation is **conformant** if and only if it satisfies every "MUST"
and "REQUIRED" clause in this document and validates all payloads against the
version 1 schemas.

> **TODO:** Define conformance levels, if any (e.g., core vs. extended).
