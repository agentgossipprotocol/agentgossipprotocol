---
title: A2A Binding
weight: 5
---

# A2A Binding

**Status:** Draft

AGP is a **transport-independent protocol**: its records
([Core](/spec/core/)), interest model and authorization intersection
([Security §3](/spec/security/)), and `Sync` anti-entropy operation
([Transport](/spec/transport/)) are defined without reference to any host
protocol. This document defines the **A2A binding** — the preferred and
best-integrated way for A2A-compatible agents to advertise, negotiate, and
run AGP sessions. It is one binding that satisfies the abstract binding
contract in [Transport §5](/spec/transport/); it is not part of the AGP core, and
the core MUST NOT be understood to depend on any A2A object (`Task`,
`Message`, `Artifact`, or `AgentCard`).

RFC 2119 keyword conventions are inherited from [Core](/spec/core/).

## 1. Extension Identity

Under A2A, AGP is a standard A2A extension identified by the URI:

```
https://agentgossipprotocol.org/a2a/extensions/gossip/v1
```

This URI is distinct from, and MUST NOT be confused with, any other
extension that happens to abbreviate to "AGP" — A2A extensions are
identified by URI, not by acronym (see the naming note in
[the specification overview](../)).

The `/v1` in the URI is the version of **this A2A binding**, which evolves
independently of the AGP **core protocol** version (§7).

## 2. Agent Card Declaration

An agent advertises AGP support by adding an `AgentExtension` to its
`AgentCapabilities.extensions`:

```json
{
  "supportedInterfaces": [
    {
      "url": "https://reconciler.example.com/a2a/v1",
      "protocolBinding": "HTTP+JSON",
      "protocolVersion": "1.0"
    }
  ],
  "capabilities": {
    "extensions": [
      {
        "uri": "https://agentgossipprotocol.org/a2a/extensions/gossip/v1",
        "description": "Mutual anti-entropy exchange of operational gossip records",
        "required": false,
        "params": {
          "agpCoreVersion": "1.0",
          "recordKinds": [
            "agent.lifecycle.v1",
            "route.sli.v1",
            "capability.evaluation.v1",
            "peer.referral.v1"
          ],
          "digestAlgorithms": ["manifest-sha256-v1", "merkle-sha256-v1"],
          "maxFrameBytes": 262144
        }
      }
    ]
  }
}
```

- `required` MUST normally be `false`: an agent can still perform ordinary
  A2A work without gossip, consistent with A2A's own guidance that data-only
  extensions should not be marked required.
- `agpCoreVersion` declares the AGP **core protocol** version the agent
  speaks; the extension URI declares the **binding** version. A peer MUST
  treat these two versions independently when negotiating (§3, §7).
- `recordKinds` declares the subset of v1 record kinds
  ([Core §4](/spec/core/)) the agent will exchange.

Modern A2A Agent Cards use `supportedInterfaces` (not a single top-level
`url`) to declare transport endpoints; the example above reflects that.

## 3. Negotiation and Activation

Extension activation follows A2A's standard mechanism on every A2A
transport:

- The client includes the extension URI in the `A2A-Extensions` request
  header (for gRPC, the equivalent metadata value).
- The server activates the extension if supported and echoes the activated
  URIs back in the `A2A-Extensions` response header.
- Version negotiation combines the URI (binding version) with the
  `HELLO` frame's `version` and the peer's `agpCoreVersion` (core version).
  A peer MUST refuse to proceed if it shares no compatible core version
  with its peer, returning the `unsupported_version` error
  (`error.schema.json`).

Before opening a session, an agent MUST confirm the peer advertises this
extension in its Agent Card ([Discovery §3](/spec/discovery/)).

## 4. Identity and Authorization Mapping

AGP's abstract notions of peer and issuer identity map, under this binding,
onto A2A's authenticated identities:

| AGP concept | A2A mapping |
| --- | --- |
| Peer / issuer identity | The authenticated A2A agent identity of the connecting party. |
| Authoritative descriptor of a subject | The subject's A2A Agent Card. |
| Capability reference in a `subject` or selector | An A2A skill/capability identifier. |
| Peer authentication scheme ([Transport §5](/spec/transport/)) | The security scheme the peer's Agent Card declares. |

- A peer MUST authenticate its counterpart using the security scheme the
  counterpart's Agent Card declares. AGP over A2A MUST NOT define or accept
  an alternate, weaker authentication path.
- Extension methods MUST NOT bypass A2A's primary security controls; A2A's
  own requirement that servers authenticate incoming requests applies
  unchanged to AGP sessions.
- A `peer.referral.v1` record ([Core §4.4](/spec/core/)) is resolved, under this
  binding, by fetching and validating the referenced **Agent Card** before
  the referral is trusted for anything beyond "check this out"
  ([Discovery §3](/spec/discovery/)).

## 5. Transport Mappings

Every A2A transport maps to the same `Sync` state machine
([Transport §1](/spec/transport/)) and MUST NOT weaken the abstract binding
requirements in [Transport §5](/spec/transport/).

**gRPC.** A native bidirectional streaming RPC:

```protobuf
service AgentGossipSync {
  rpc Sync(stream GossipFrame) returns (stream GossipFrame);
}
```

**HTTP+JSON.** A resumable `POST`:

```
POST /extensions/gossip/v1:sync
A2A-Extensions: https://agentgossipprotocol.org/a2a/extensions/gossip/v1
```

Each request and response body carries a `GossipFrame`-shaped JSON object
conforming to `frame.schema.json`:

```json
{
  "sessionId": "01JZ...",
  "hello": {},
  "scopes": [],
  "summaries": [],
  "wants": [],
  "records": [],
  "acks": [],
  "continuationToken": null,
  "done": false
}
```

The first request implicitly opens the session (its `sessionId` is
generated by the client and echoed by the server); subsequent requests
carry the same `sessionId` until `done: true` or session expiry.

**JSON-RPC.**

> **TODO:** Define the JSON-RPC method name and request/response envelope
> for a `Sync` exchange, and specify the resumption/continuation timeout
> defaults for each transport.

## 6. Do Not Carry Gossip Through `SendMessage`

Routine gossip MUST use the `Sync` operation defined above, never A2A's
ordinary message-sending path. Mixing operational control traffic into A2A
tasks, conversations, or task history would violate [Core's](/spec/core/) scope
boundary and make gossip records indistinguishable from task content.

## 7. Versioning

This binding and the AGP core protocol are versioned independently:

```
AGP core protocol:   1.0   (declared as params.agpCoreVersion)
AGP A2A binding:      v1    (the extension URI path segment)
A2A protocol:         compatible with 1.x
```

A change to this binding (for example, adding a JSON-RPC mapping) does not
by itself change the core protocol version. Likewise, a backward-compatible
core improvement may only widen the range of `agpCoreVersion` values this
binding advertises. Breaking changes to either layer follow
[Semantic Versioning](https://semver.org/) for that layer.
