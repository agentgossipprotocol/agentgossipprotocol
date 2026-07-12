# Agent Gossip Protocol

A transport-independent protocol that propagates expiring, attributable
operational evidence — self-reported lifecycle, measured route quality,
authorized evaluation scores, and unverified referrals — so agents can
choose more effectively among action paths they've already discovered and
are already authorized to use. Its preferred integration is an A2A
extension. Gossip never carries tasks, prompts, business data, or user
content, and never grants access on its own.

This repository is the **source of truth** for the Agent Gossip Protocol (AGP):
the human-readable specification, the machine-readable schemas, and the process
for evolving them. The rendered site is published at
<https://agentgossipprotocol.org>.

## Repository layout

```
.
├── spec/              # Human-readable specification (RFC-style, RFC 2119)
│   ├── _index.md      #   Overview + conventions + document set
│   ├── core.md        #   Core protocol: record model & rules (transport-independent)
│   ├── discovery.md   #   Peer discovery and verification
│   ├── security.md    #   Threat model (Sybil, poisoning, ...)
│   ├── transport.md   #   The Sync operation + abstract binding contract
│   ├── binding-a2a.md #   The normative A2A binding (extension, Agent Card, wire mappings)
│   └── proposals/     #   Specification Enhancement Proposals (SEPs)
│       ├── 0000-template.md
│       └── 0001-initial.md
│
├── schemas/           # Machine-readable definitions (versioned)
│   └── v1/
│       ├── record.schema.json     # the immutable, signed gossip record
│       ├── frame.schema.json      # the GossipFrame envelope for Sync
│       ├── selector.schema.json   # interest selectors
│       └── error.schema.json      # normalized protocol error model
│
├── content/           # Hugo site pages (home, docs landing)
├── LICENSES/          # Full license texts (Apache-2.0, CC-BY-4.0)
└── ...                # Hugo config and build tooling
```

The `spec/` directory is rendered into the docs site via Hugo module mounts.
`spec/proposals/` is published at `/proposals/`, while `schemas/` is served at
`https://agentgossipprotocol.org/schemas/...`.

## Versioning

The specification follows [Semantic Versioning](https://semver.org/). Changes
are proposed and ratified through the [proposals](spec/proposals/) process.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md), [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md),
and [GOVERNANCE.md](GOVERNANCE.md). To report a security vulnerability, see
[SECURITY.md](SECURITY.md).

## License

Dual-licensed: specification/documentation under **CC-BY-4.0**, code and schemas
under **Apache-2.0**. See [LICENSE](LICENSE) for details.

## Building the site locally

```sh
hugo server
```
