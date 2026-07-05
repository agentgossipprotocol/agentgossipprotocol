---
title: Discovery
weight: 2
---

# Peer Discovery and Verification

**Status:** Draft

This document specifies how agents discover and verify **gossip peers**.
Terminology and RFC 2119 keyword conventions are inherited from [Core](core/).

**Two kinds of discovery — only one lives here.** Discovering *gossip peers*
(who to exchange context with) is the subject of this document. Discovering
*action paths* (which agents, skills, or MCP servers can perform a task) is
not a separate mechanism at all — it is what the gossiped content itself
provides, via capability announcements and observations
([Core §3](core/)). An agent needs only a few verified gossip peers to learn
about the entire environment's action paths.

## 1. Discovery Goals

> **TODO:** State what gossip-peer discovery must achieve (bootstrapping,
> liveness, membership) and its non-goals.

## 2. Bootstrapping

> **TODO:** Define how a new agent joins the network (seed peers, well-known
> endpoints, etc.). Specify MUST/SHOULD/MAY behavior.

## 3. Peer Verification

Before exchanging state, an agent MUST verify a candidate peer's identity.

> **TODO:** Define the identity model (keys, certificates, proofs) and the
> verification handshake.

## 4. Membership and Liveness

> **TODO:** Define how membership changes (join/leave/fail) are detected and
> propagated. Cross-reference the Security document for Sybil resistance.

## 5. Interaction with Security

Discovery is a primary attack surface. Implementations MUST consider the
threats enumerated in [Security](security/), in particular Sybil attacks.
