---
title: Getting Started
weight: 1
---

# Getting Started

Agent Gossip Protocol (AGP) helps agents share planning context without
central coordination. A peer can announce capabilities it controls, report
subjective observations about action paths, and use gossiped context to decide
which path to try next.

AGP does not delegate work, carry prompts, or execute tasks. It answers a
narrower question: "What does this environment currently know about the
available ways to get work done?"

## Core Concepts

- **Agent**: a participant that consumes and produces gossiped context.
- **Peer**: another agent connected directly for gossip exchange.
- **Action path**: a route an agent may use to accomplish work, such as an
  external agent application, an inherent skill, or an MCP server.
- **Capability announcement**: a claim that an action path offers a
  capability.
- **Observation**: an attributed report about availability, degradation, or
  quality of an action path.

## Reading Path

1. Read the [Core Protocol](../../spec/core/) for the message model and
   conformance rules.
2. Review [Discovery](../../spec/discovery/) and
   [Transport](../../spec/transport/) for how peers connect and exchange
   messages.
3. Check [Security](../../spec/security/) before trusting or forwarding
   observations from other agents.
4. Use the versioned [JSON Schemas](../../schemas/v1/message.schema.json) when
   validating wire payloads.

## Implementation Shape

A minimal implementation usually needs to:

1. Maintain a local context store for capabilities and observations.
2. Validate incoming payloads against the versioned schemas.
3. Preserve observer attribution and timestamps when forwarding observations.
4. Expire stale context using TTLs and local freshness policy.
5. Compute rankings locally instead of expecting rankings to arrive over the
   wire.

The details are intentionally implementation-defined where local policy matters:
trust weighting, ranking formulas, peer selection, and retry behavior are left
to each agent.
