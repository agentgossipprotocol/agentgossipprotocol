---
title: Getting Started
weight: 1
---

# Getting Started

Agent Gossip Protocol (AGP) is a transport-independent protocol whose
preferred integration is an A2A extension. Agents exchange signed, expiring
**records** of operational evidence — self-reported lifecycle, directly
measured route quality, authorized evaluation scores, and unverified
referrals — and use that evidence to choose more effectively among action
paths they've already discovered through ordinary A2A/MCP discovery and are
already authorized to use.

AGP does not delegate work, carry prompts, or execute tasks. It does not
decide what exists or who may use it, either — those are Agent Cards,
A2A discovery, and authorization policy. AGP answers a narrower question:
"Given what officially exists and what I'm allowed to use, what has
recently been observed about it?"

## Core Concepts

- **Agent**: a participant that consumes and produces gossip records.
- **Peer**: another agent with which an agent runs a `Sync` session.
- **Action path**: a route an agent may use to accomplish work, such as an
  external agent application, an inherent skill, or an MCP server.
- **Record**: an immutable, signed, timestamped, expiring unit of evidence.
  Every record has an `issuer` (who observed it) and a `subject` (what it's
  about) — often different agents.
- **Record kind**: v1 defines four — `agent.lifecycle.v1` (self-reported
  state), `route.sli.v1` (measured success/latency/error rates),
  `capability.evaluation.v1` (authorized evaluation scores), and
  `peer.referral.v1` (an unverified hint to check another Agent Card).
- **Selector**: a description of which records a peer is willing to
  disclose, or wants to receive.

## Reading Path

1. Read the [Core Protocol](../../spec/core/) for the record model, the four
   v1 record kinds, and what gossip is explicitly excluded from carrying.
2. Review [Transport](../../spec/transport/) for the single, transport-independent
   `Sync` anti-entropy operation and the contract any binding must satisfy.
3. Read the [A2A Binding](../../spec/binding-a2a/) for how AGP is advertised,
   negotiated, and run as an A2A extension (Agent Card, wire mappings).
4. Read [Discovery](../../spec/discovery/) for how peers bootstrap and
   select one another — distinct from discovering action paths themselves.
5. Check [Security](../../spec/security/) before trusting, storing, or
   forwarding a record from another agent: signing, replay protection, and
   the selector/authorization intersection that governs what may even be
   disclosed between two peers.
6. Use the versioned [JSON Schemas](../../schemas/v1/record.schema.json)
   when validating wire payloads.

## Implementation Shape

A minimal implementation usually needs to:

1. Maintain a local record store, partitioned for anti-entropy
   reconciliation (tenant / domain / record kind / subject namespace).
2. Validate incoming records against the versioned schemas, verify their
   signatures, and reject replays using `(issuer, issuerEpoch, sequence)`.
3. Compute the directional selector/authorization intersection before
   disclosing anything to a peer — never trust a peer's own claim of what
   it's entitled to receive.
4. Deduplicate by content-addressed `recordId`; there is no "never revisit
   a peer" rule to implement, just dedup, TTLs, and per-peer summaries.
5. Expire stale records using their `expiresAt`, and treat corrections as
   new records referencing `supersedes` rather than mutating stored ones.
6. Feed accepted records into planning as a **read-only, advisory** input,
   applied only after authoritative capability and authorization checks —
   never as an authorization input itself.

The details are intentionally implementation-defined where local policy
matters: trust weighting across issuers, ranking formulas, peer selection,
and retry behavior are left to each agent.
