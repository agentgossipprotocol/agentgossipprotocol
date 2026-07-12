---
title: Discovery
weight: 2
---

# Peer Discovery and Verification

**Status:** Draft

This document specifies how agents bootstrap and select **gossip peers**.
Terminology and RFC 2119 keyword conventions are inherited from [Core](/spec/core/).

**Two kinds of discovery — only one lives here.** Discovering *gossip peers*
(who to run a `Sync` session with) is the subject of this document.
Discovering *action paths* (which agents, skills, or MCP servers can
perform a task) is not this protocol's job at all: it is the job of the host
protocol's discovery (Agent Cards and A2A discovery under the
[A2A Binding](/spec/binding-a2a/)). AGP does not invent a second, competing
discovery system, and a `peer.referral.v1` record ([Core §4.4](/spec/core/)) is
never a substitute for validating that authoritative descriptor.

## 1. Discovery Goals

Gossip-peer discovery MUST let an agent find at least one eligible,
authorized peer to `Sync` with. It is explicitly **not** required to
achieve full network membership visibility, global topology knowledge, or
guaranteed connectivity — AGP tolerates partial, changing connectivity
between independent connected subgraphs (§4).

## 2. Bootstrapping

A conforming implementation MAY bootstrap its initial peer set from any
combination of:

- **Direct configuration** — a statically configured list of peer
  endpoints.
- **An enterprise registry or catalog** of known agents.
- **A DHT-based capability index**, where deployed.
- **A2A Agent Card discovery** — well-known URLs, registries/catalogs, or
  direct configuration, as A2A itself defines. A2A does not standardize a
  single global registry, and neither does AGP.
- **Previously validated peer referrals** — a `peer.referral.v1` record
  ([Core §4.4](/spec/core/)) whose target Agent Card has already been fetched and
  validated MAY be promoted to a candidate peer.

None of these bootstrap mechanisms is authoritative on their own for
*action-path* discovery; they only seed the set of agents an agent will
attempt to run `Sync` sessions with. Whether the resulting peer actually
offers a usable capability is decided by that peer's Agent Card and by
authorization policy, not by how the peer was found.

## 3. Peer Verification

Before opening a `Sync` session, an agent MUST verify a candidate peer's
identity using the binding's peer-authentication mechanism
([Transport §5](/spec/transport/); under the [A2A Binding §4](/spec/binding-a2a/), the
security scheme the peer's Agent Card declares). An agent MUST also confirm
the candidate peer advertises AGP support via the binding's capability
advertisement (under A2A, `AgentCapabilities.extensions` —
[A2A Binding §2](/spec/binding-a2a/)) before attempting to negotiate a session.

A `peer.referral.v1` record MUST be treated as unverified until this step
completes. Fetching and validating the referenced subject's authoritative
descriptor — under the A2A binding, the referenced Agent Card, including
confirming its own signature where present — is REQUIRED before an agent
relies on the referral for anything beyond "check this out."

## 4. Peer Selection

A conforming agent SHOULD select peers for each gossip interval using a
weighted mix of:

- **Random selection**, for network mixing.
- **Interest overlap** — peers whose advertised or previously observed
  selectors ([Security §3](/spec/security/)) intersect meaningfully with the
  agent's own.
- **Recency** — preferring peers not contacted recently, to spread
  freshness.
- **Zone or topology diversity**, where topology metadata is available.
- **Reliability and backoff state** — deprioritizing peers with recent
  failed or truncated sessions.

Each `Sync` session ([Transport §1](/spec/transport/)) is one-to-one and mutual:
both sides may disclose and both sides may request, subject to their own
independently evaluated authorization ([Security §3](/spec/security/)).

Randomized push-pull gossip can converge in O(log N) rounds in a connected,
sufficiently well-mixed graph. This is **not** a universal guarantee.
Authorization boundaries and selector-based interest filtering partition a
real deployment into smaller dissemination subgraphs; convergence
properties apply separately within each connected subgraph, and there is no
protocol-level guarantee of convergence *across* subgraphs that never share
an authorized, interest-overlapping session.

## 5. Membership and Liveness

Membership is not tracked as a separate, authoritative construct. An agent
observes another agent's liveness only through:

- Successful `Sync` sessions (§4).
- `agent.lifecycle.v1` records it receives ([Core §4.1](/spec/core/)), which are
  self-reported and advisory, not a substitute for a health check.

An agent that has not been reachable for an implementation-defined period
SHOULD be deprioritized by peer selection (§4) rather than removed from any
authoritative registry — AGP has no authority to remove an agent from
anything.

## 6. Interaction with Security

Discovery is a primary attack surface. Implementations MUST consider the
threats enumerated in [Security](/spec/security/), in particular Sybil attacks and
eclipse/partitioning.
