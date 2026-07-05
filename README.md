# Agent Gossip Protocol

A protocol for decentralized agent-to-agent gossip communication.

This repository is the **source of truth** for the Agent Gossip Protocol (AGP):
the human-readable specification, the machine-readable schemas, and the process
for evolving them. The rendered site is published at
<https://agentgossipprotocol.org>.

## Repository layout

```
.
├── spec/              # Human-readable specification (RFC-style, RFC 2119)
│   ├── _index.md      #   Overview + conventions + document set
│   ├── core.md        #   Definitive core rules
│   ├── discovery.md   #   Peer discovery and verification
│   ├── security.md    #   Threat model (Sybil, poisoning, ...)
│   ├── transport.md   #   Wire-level transport expectations
│   └── proposals/     #   Specification Enhancement Proposals (SEPs)
│       ├── 0000-template.md
│       └── 0001-initial.md
│
├── schemas/           # Machine-readable definitions (versioned)
│   └── v1/
│       ├── message.schema.json
│       ├── state.schema.json
│       └── error.schema.json
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
