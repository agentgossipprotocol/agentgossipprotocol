---
title: Security
weight: 3
---

# Security Considerations and Threat Model

**Status:** Draft

This document describes the threat model for the Agent Gossip Protocol. It is
a *design* document about protocol-level threats and mitigations. To **report a
vulnerability**, see the repository's [SECURITY.md](https://github.com/agentgossipprotocol/agentgossipprotocol/blob/main/SECURITY.md)
policy instead.

RFC 2119 keyword conventions are inherited from [Core](/spec/core/).

## 1. Threat Model Scope

AGP assumes a mesh of mutually distrustful agents, some of whom may be
compromised, misconfigured, or actively adversarial. It assumes the binding
in use supplies transport security (TLS) and peer authentication
([Transport §5](/spec/transport/)); AGP does not re-specify transport security, it
relies on the binding's — under the [A2A Binding](/spec/binding-a2a/), A2A's own
TLS and authentication scheme.

In scope: forgery or tampering of records; replay of stale records; Sybil
identities; over-disclosure of evidence across authorization boundaries;
and denial-of-service against a peer's `Sync` session handling.

Out of scope: compromise of an agent's underlying task execution, prompts,
or business logic — AGP carries none of that data ([Core §3.6](/spec/core/)) and
so cannot be a vector for it.

## 2. Sybil Attacks

An adversary that can cheaply create many identities can dominate gossip
fan-out and bias convergence, or manufacture apparent consensus about a
subject's health.

Mitigations:

- Every record MUST be signed by its `issuer` ([Core §3.1](/spec/core/), §6
  below); an unsigned or invalidly signed record MUST be rejected.
- A receiver SHOULD weight issuers by an implementation-defined trust score
  informed by identity provenance (e.g., how the issuer's key was
  provisioned) rather than treating all issuers as equally credible.
- Peer selection ([Discovery §4](/spec/discovery/)) SHOULD bound how much of an
  agent's peer set can be filled by identities sharing a suspiciously
  similar provisioning path or endpoint range.

## 3. Selector Disclosure and the Authorization Intersection

Because a mutual `Sync` session carries evidence in both directions, "peers
can talk to each other" does not imply "peers may disclose the same things
to each other." Each direction of a session has its own authorized scope.

An **interest selector** describes what a peer is willing to disclose, or
what it wants to receive:

```json
{
  "selectorId": "finance.invoice-routes",
  "subjectNamespaces": ["agent://bank-a/reconciliation/*"],
  "capabilities": ["invoice.reconcile/v2"],
  "recordKinds": ["agent.lifecycle.v1", "route.sli.v1"],
  "tenant": "bank-a"
}
```

For a session between A and B, the effective exchange scope in the
A-to-B direction is:

```
Scope(A→B) = Publish(A) ∩ Subscribe(B) ∩ Authorization(A,B) ∩ SupportedTypes(A,B)
```

That is: what A is willing to publish, intersected with what B is
subscribed to, intersected with what an authorization decision permits for
this specific (A, B) pair, intersected with the record kinds both sides
support. The B-to-A direction is computed independently and is not assumed
to be symmetric — a peer being allowed to *receive* something does not
imply it is allowed to *send* the analogous thing back.

- A peer MUST compute and enforce this intersection before transferring any
  record; the intersection MUST NOT be computed by, or trusted from, the
  remote peer.
- For sensitive environments, peers SHOULD exchange preauthorized
  `selectorId` values rather than disclosing their complete
  `subjectNamespaces` or dependency graph.
- Standing relationships ("this peer may gossip with that peer at all") and
  runtime constraints ("only for this tenant, this record kind, this
  freshness window, this environment") are two different kinds of policy
  and MAY be evaluated by different mechanisms (e.g. a relationship-based
  authorization store for the former, a runtime policy engine for the
  latter). AGP does not mandate a specific product for either; it only
  requires that both be evaluated per (issuer, subject, receiver) before
  disclosure.

## 4. Record Poisoning

Because planning decisions are influenced by gossiped records
([Core §6](/spec/core/)), false or manipulated records are a direct attack on
agent behavior. An adversary may:

- Issue a false `route.sli.v1` or `agent.lifecycle.v1` record about a
  healthy competitor path to divert traffic away from it.
- Attempt to alter a record in transit — including appending path/hop
  information to it — to change its meaning or invalidate its signature.

Mitigations follow from the record model: every record is a signed claim
bound to an identified `issuer` ([Core §3.1](/spec/core/)); receivers MUST
preserve per-issuer provenance and MUST NOT merge claims across issuers
into a single record ([Core §3.2](/spec/core/)); relay/hop metadata belongs only
in the *per-hop transfer envelope*, never inside the signed record itself,
so relaying a record can never invalidate its origin signature
([Transport §2](/spec/transport/)); and rank is computed locally
([Core §2](/spec/core/)), so no single record can unilaterally demote a path
network-wide.

## 5. Self-Promotion and Unverified-Capability Attacks

An adversary may inflate its own state — issuing favorable
`agent.lifecycle.v1` records about itself, or fabricating
`capability.evaluation.v1` records — to attract routing preference.

Mitigations:

- `capability.evaluation.v1` records MUST identify the evaluator, and a
  receiver MUST verify that the identified evaluator is authorized to issue
  evaluations for the claimed suite before trusting the score
  ([Core §4.3](/spec/core/)). Self-issued evaluations from an unauthorized
  evaluator MUST be disregarded.
- `agent.lifecycle.v1` is explicitly advisory ([Core §4.1](/spec/core/)); a
  receiver MUST NOT treat it as a substitute for its own health checks or
  for `route.sli.v1` evidence it has directly measured.
- `peer.referral.v1` records MUST trigger independent Agent Card validation
  before any claimed capability is trusted ([Core §4.4](/spec/core/),
  [Discovery §3](/spec/discovery/)) — a referral cannot itself manufacture a
  capability.

## 6. Confidentiality, Integrity, and Authenticity

- **Transport.** AGP MUST run over a binding that provides confidentiality
  and integrity (e.g., TLS) and authenticates peers
  ([Transport §5](/spec/transport/)). A binding MUST NOT bypass the host
  protocol's primary security controls (under A2A, see
  [A2A Binding §4](/spec/binding-a2a/)).
- **Record signing.** Every record's `signature` MUST be computed over the
  canonical form of the unsigned record, using [JSON Canonicalization
  Scheme (JCS)](https://www.rfc-editor.org/rfc/rfc8785). `recordId` MUST be
  the SHA-256 hash of that same canonical unsigned form (a content
  address). Record signing is part of the core protocol and is independent
  of any host-protocol card signing: under A2A, an Agent Card signature
  authenticates only the *card* and says nothing about gossip records, which
  is why records carry their own, separate signature.
- A receiver MUST verify a record's signature against the issuer's known
  key before storing or forwarding it, and MUST reject records that fail
  verification.

## 7. Replay and Freshness

Replayed records resurrect stale evidence (e.g., replaying an old
`degraded` lifecycle record after the path has recovered). Defenses:

- Mandatory `observedAt`/`expiresAt` and TTL expiry
  ([Core §3.5](/spec/core/)) bound how long a replay can matter.
- `issuerEpoch` and a monotonically increasing `sequence`
  ([Core §3.1](/spec/core/)) let a receiver detect and reject a replayed or
  out-of-order record: a record with a `sequence` at or below one already
  seen for the same `(issuer, issuerEpoch)` MUST be rejected.
- Content-addressed `recordId` (§6) makes exact-duplicate replay a cheap,
  safe no-op rather than a reprocessing event ([Core §5](/spec/core/)).

## 8. Eclipse and Partitioning

An adversary controlling all of a victim's gossip peers can eclipse it,
feeding it fabricated or selectively withheld evidence. Because rank is
computed locally from received records, an eclipsed agent plans against
attacker-chosen evidence without knowing it.

Mitigations SHOULD include peer-selection diversity (topology/zone
diversity, random mixing — [Discovery §4](/spec/discovery/)) and SHOULD avoid
relying on a single peer for all evidence about a given subject.

> **TODO:** Define minimum peer-diversity requirements and detection
> heuristics for suspected eclipse.

## 9. User Data and Free-Form Content Exclusion

User activity/usage profiles, prompts, task histories, tool inputs/outputs,
business records, raw exception strings, chain-of-thought or other
model-generated explanations, secrets/credentials, and free-form reputation
assertions MUST NOT be gossiped in v1 ([Core §3.6](/spec/core/)). This is a
protocol-level exclusion, not an implementation preference: a conformant
peer MUST reject an otherwise well-formed record whose `kind` is not one of
the four v1 kinds ([Core §4](/spec/core/)) if that record's `value` contains
free-form natural-language content.

If an agent acts on behalf of a user, user identity and activity MUST be
handled through on-behalf-of authorization and local policy evaluation, not
propagated as gossip. Gossiping user presence remains deferred pending a
dedicated SEP that resolves identity representation, consent, minimization,
and retention; until then, the `presence` concept has no wire
representation anywhere in this specification.
