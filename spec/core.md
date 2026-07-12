---
title: Core
weight: 1
---

# Core Protocol

**Status:** Draft

This document defines the core, normative rules of the Agent Gossip Protocol
(AGP). It is implementation-agnostic: it describes the records and behavior
peers MUST exhibit, never how a given peer is built. The wire operation that
exchanges these records is defined in [Transport](/spec/transport/).

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as
described in RFC 2119 and RFC 8174.

## 1. Overview

In a multi-agent environment, an agent determines a course of action for its
tasks by choosing among **action paths**: external agent applications, its
own inherent skills, or MCP servers. Which paths *officially exist*, and
which the agent is *authorized* to use, is decided entirely outside AGP — by
the host protocol's authoritative discovery (Agent Cards and A2A discovery
under the [A2A Binding](/spec/binding-a2a/)) and by authorization policy. AGP's
only job is to keep the agent's picture of the *already-discovered,
already-authorized* subset fresh: which paths are currently healthy, which
are degraded, and what has recently been measured about them.

AGP propagates **gossip records** — immutable, signed, expiring statements
of evidence. A record is never authoritative and never self-sufficient: it
is one attributed observation, valid for a bounded window, that a receiver
weighs alongside everything else it knows.

## 2. Terminology

- **Agent** — an autonomous participant that both consumes gossip records
  for planning and produces records for others.
- **Peer** — an agent with which another agent runs a `Sync` session
  ([Transport §1](/spec/transport/)).
- **Action Path** — a means by which an agent can accomplish (part of) a
  task. An action path is one of: an **external agent application**, an
  **inherent skill** of the agent itself, or an **MCP server**.
- **Issuer** — the agent that authored and signed a record. The issuer is
  not necessarily the record's subject; an issuer routinely reports on
  paths it invoked, not just paths it hosts.
- **Subject** — the action path, capability, or agent that a record is
  *about*.
- **Record** — an immutable, signed, timestamped, expiring unit of gossip
  evidence conforming to `record.schema.json` (§3).
- **Record Kind** — the typed shape of a record's `value` (§4). Kinds are
  versioned independently (e.g. `route.sli.v1`).
- **Selector** — a receiver-defined description of the records it is
  interested in and/or willing to disclose ([Security §3](/spec/security/)).
- **Partition** — a stable subdivision of the record space (by tenant,
  domain, record kind, and subject namespace) used to make anti-entropy
  reconciliation tractable ([Transport §4](/spec/transport/)).
- **Rank** — a preference ordering over action paths for a given task. Rank
  is **computed locally** by each agent from the records it holds, alongside
  authoritative capability and authorization data. Rank is NOT gossiped and
  has no wire representation.

## 3. Record Model

### 3.1 Structure

Every record MUST validate against `record.schema.json` (version 1) and
MUST contain, at minimum:

| Field | Meaning |
| --- | --- |
| `recordId` | Content address: the SHA-256 hash of the canonical (JSON Canonicalization Scheme) unsigned record. |
| `issuer` | The agent that observed and signed the record. |
| `issuerEpoch` | An opaque token that changes whenever the issuer's sequence counter resets (e.g. on restart), so `sequence` is unambiguous across issuer lifetimes. |
| `sequence` | A monotonically increasing counter scoped to `(issuer, issuerEpoch)`. |
| `subject` | The action path, capability, or agent the record is about. May differ from `issuer` (§3.2). |
| `kind` | The versioned record type governing the shape of `value` (§4). |
| `scope` | Tenant/domain/partition context used for authorization and reconciliation. |
| `observationWindow` | The time range over which the evidence in `value` was collected, where applicable. |
| `value` | The kind-specific evidence payload (§4). |
| `evidence` | How the observation was made (e.g. `method`, `sampleCount`) — never a bare confidence score (§3.3). |
| `observedAt` | When the issuer made the observation. |
| `expiresAt` | When the record MUST be treated as stale and discarded (§3.5). |
| `supersedes` | OPTIONAL. The `recordId` of a prior record this one corrects or retracts (§3.4). |
| `signature` | The issuer's signature over the canonical unsigned record. |

### 3.2 Issuer and Subject Are Distinct

A record's `issuer` is whoever made and signed the observation; its
`subject` is whoever or whatever the observation is about. They are
frequently different agents — e.g. agent A issues a `route.sli.v1` record
whose subject is agent B's capability, based on A's own measurements of
calling B.

- A record MUST identify `issuer` and `subject` as separate fields.
- A receiver MUST NOT collapse or merge records from different issuers about
  the same subject into a single record. Provenance per issuer is preserved;
  weighting across issuers is a local, implementation-defined policy.

### 3.3 Evidence, Not Confidence

Records MUST NOT carry a bare, unitless confidence score (e.g.
`confidence: 0.82`). Instead, `evidence` MUST describe how the observation
was made — at minimum an observation `method` (e.g.
`direct-runtime-observation`) and, where applicable, a `sampleCount` and the
`observationWindow`. Receivers derive their own trust weighting from this
concrete evidence; the protocol does not define one.

### 3.4 Immutability, Corrections, and Retractions

Records are immutable once signed. A correction or retraction MUST be a new,
independently signed record whose `supersedes` field references the
`recordId` of the record it replaces. Implementations MUST NOT mutate a
stored record in place.

### 3.5 Freshness and Expiry

Stale evidence is worse than no evidence.

- Every record MUST carry `observedAt` and `expiresAt`.
- A receiver MUST discard (or mark stale and stop surfacing) a record once
  `expiresAt` has passed.
- A receiver SHOULD reject records whose `observedAt` is further in the
  future than the receiver's permitted clock skew, and MUST reject records
  whose `(issuer, issuerEpoch, sequence)` it has already seen a higher
  sequence for (replay protection; see [Security §7](/spec/security/)).

> **TODO:** Define default TTL ranges per record kind and the maximum
> permitted clock skew.

### 3.6 What Is Never a Record

The following MUST NOT be represented as a gossip record in v1: user
activity or usage profiles; prompts or task histories; tool inputs or
outputs; business records; raw exception strings or stack traces; chain of
thought or other model-generated explanations; secrets or credentials;
unverified claims about internal skills; or free-form reputation assertions
about an agent ("Agent B is unreliable"). See also [Security §9](/spec/security/).

If an agent acts on behalf of a user, user identity MUST be handled through
on-behalf-of authorization and local policy evaluation, never propagated as
gossip.

## 4. Record Kinds (v1)

v1 defines exactly four record kinds. A conforming peer MAY support a
strict subset; it MUST NOT invent unregistered kinds and expect peers to
interpret them (unrecognized kinds MUST be treated per
[Transport §3](/spec/transport/), not guessed at).

### 4.1 `agent.lifecycle.v1`

Self-reported runtime state of the issuer itself (`issuer == subject`):
`ready`, `draining`, `degraded`, or `unavailable`; the issuer's current
deployment epoch; and, where known, an expected recovery time. This is
**advisory** and does not replace health checks — a receiver MUST NOT treat
the absence of a lifecycle record as evidence of health.

### 4.2 `route.sli.v1`

Direct measurements made while the issuer invoked the subject capability:
attempt count, success count, normalized error classes (never raw error
text — see [Security §9](/spec/security/)), and a latency distribution, all
bound to `observationWindow`.

### 4.3 `capability.evaluation.v1`

An evaluation result issued by an authorized evaluator, identifying: the
capability under evaluation, the evaluator's identity, the evaluation suite
ID and version, the dataset class, the score, the sample size, the
evaluation timestamp, and expiry. Scores from different suites, suite
versions, or dataset classes MUST NOT be treated as directly comparable by
a receiver.

### 4.4 `peer.referral.v1`

A hint that another endpoint or capability may exist. A referral is
**never** sufficient on its own: a receiver MUST fetch and validate the
referenced subject's authoritative descriptor through the host protocol's
discovery — under the [A2A Binding](/spec/binding-a2a/), the referenced Agent
Card — before treating the referral as anything more than a hint to check
([Discovery §3](/spec/discovery/)). A referral MUST NOT be used to bypass that
validation or authorization.

## 5. Deduplication and Bounded Propagation

Earlier drafts of this protocol proposed a path-based invariant: "gossip
cannot come to a previously touched agent." That rule is **not** part of
this specification. Enforcing it would require every record to carry its
full traversal path (or a probabilistic path filter), which leaks topology,
grows record size at every hop, and can incorrectly block legitimate
reconciliation in a mesh with cycles.

Instead, a conforming peer MUST use content-addressed deduplication:

- `recordId` is a content address (§3.1); a peer that has already stored a
  given `recordId` MUST treat re-receipt as a no-op — it MUST NOT reprocess
  or duplicate-store the record, but receiving it again is not an error.
- A peer MUST maintain a local seen-record cache (bounded by the freshness
  rules in §3.5) sufficient to recognize duplicates.
- A peer SHOULD maintain per-peer summaries of what it has already sent to
  or acknowledged from each peer, to avoid re-offering records that peer
  already has ([Transport §4](/spec/transport/)).
- A transfer envelope MAY carry a hop budget as an optional, non-normative
  optimization hint. Hop budgets bound wasted work; they are not a
  correctness mechanism, and their absence or exhaustion MUST NOT be
  interpreted as evidence about the record's validity.

Cycles in the underlying peer topology are harmless: once two peers'
partition digests converge (§ [Transport §4](/spec/transport/)), further sessions
between them transfer nothing.

## 6. Planner Consumption

A planner or router MAY consume gossip records, but only as a read-only,
advisory input alongside authoritative capability and authorization data.

- Gossip MUST NOT be used as an authorization input. A record MUST NOT grant
  access to an action path the receiver is not already authorized to use.
- Gossip MUST NOT independently trigger a destructive or irreversible
  action, and MUST NOT be used to permanently block a user or path — it may
  only influence routing preference, retry policy, and exploration.
- A planner MUST apply authoritative capability and authorization data
  first, and only then apply fresh gossip evidence to prefer among the
  remaining, permitted options. Evidence that is missing or stale MUST fall
  back to authoritative defaults rather than blocking a decision.
- A planner consuming multiple record kinds about the same subject MUST
  keep them distinguishable by issuer, kind, and freshness rather than
  collapsing them into a single unqualified statement.

## 7. Conformance

An implementation is **conformant** if and only if it satisfies every "MUST"
and "REQUIRED" clause in this document and in [Transport](/spec/transport/), and
validates all payloads against the version 1 schemas.

> **TODO:** Define conformance levels, if any (e.g., core vs. extended).
