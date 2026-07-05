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

RFC 2119 keyword conventions are inherited from [Core](core/).

## 1. Threat Model Scope

> **TODO:** Define trust assumptions, adversary capabilities, and what is in
> and out of scope.

## 2. Sybil Attacks

An adversary that can cheaply create many identities can dominate gossip
fan-out and bias convergence.

> **TODO:** Define required or recommended Sybil-resistance measures and their
> interaction with [Discovery](discovery/).

## 3. Observation Poisoning

Because planning decisions are driven by gossiped observations
([Core §3.3](core/)), false observations are a direct attack on agent
behavior. An adversary may:

- Report a healthy competitor path as `degraded` or `unavailable` to divert
  traffic away from it.
- Forge or mutate *forwarded* observations, misattributing claims to honest
  observers.

Mitigations follow from the subjective-observation model: observations are
claims bound to an identified `observer`, receivers MUST preserve provenance
and MUST NOT merge claims across observers, and rank is computed locally.

> **TODO:** Define required observer authentication (signatures over
> observations) and recommended receiver-side weighting of untrusted
> observers.

## 4. Self-Promotion Attacks

An adversary may inflate its own capability announcements — claiming
capabilities it cannot deliver, or announcing many paths — to attract
selection during planning.

Core requires that agents only announce paths they control
([Core §3.2](core/)); everything else about a path is learned through other
agents' observations, which naturally correct inflated claims over time.

> **TODO:** Define whether announcements require proof of control, and how
> receivers SHOULD bound trust in unverified announcements.

## 5. Eclipse and Partitioning

> **TODO:** Address eclipse attacks and network partition handling. Note the
> planning impact: an eclipsed agent plans against attacker-chosen context.

## 6. Confidentiality, Integrity, and Authenticity

> **TODO:** Define required cryptographic guarantees on the wire (reference
> [Transport](transport/)).

## 7. Replay and Freshness

Replayed observations resurrect stale context (e.g., replaying an old
"degraded" report after a path recovers). The freshness rules in
[Core §3.4](core/) (mandatory timestamps, TTL expiry) are the first line of
defense.

> **TODO:** Define nonce/timestamp/sequence requirements to prevent replay.

## 8. User Presence Privacy (Deferred)

> **Deferred:** Gossiping user presence ([Core §3.5](core/)) broadcasts where
> a user identity is active — among the most privacy-sensitive data this
> protocol could carry. This section MUST be completed (identity
> representation, consent, minimization, retention) before the reserved
> `presence` message type is specified in a future SEP. Until then,
> implementations MUST NOT gossip user identity or presence data.
