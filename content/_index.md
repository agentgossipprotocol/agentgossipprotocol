---
title: Agent Gossip Protocol
layout: hextra-home
---

{{< hextra/hero-badge >}}
  Draft specification
{{< /hextra/hero-badge >}}

<div class="agp-hero-title">
{{< hextra/hero-headline >}}
  Agent Gossip Protocol
{{< /hextra/hero-headline >}}
</div>

<div class="agp-hero-subtitle">
{{< hextra/hero-subtitle >}}
  A shared-context protocol for multi-agent systems. Agents gossip capability,
  availability, and health signals so each participant can plan against fresh,
  decentralized knowledge.
{{< /hextra/hero-subtitle >}}
</div>

<div class="agp-actions">
  <a class="agp-button agp-button-primary" href="docs/">
    Get started
  </a>
  <a class="agp-button agp-button-secondary" href="spec/">
    Read the spec
  </a>
  <a class="agp-button agp-button-secondary" href="proposals/">
    View proposals
  </a>
</div>

<div class="agp-card-grid">
  <a class="agp-card" href="spec/core/">
    <h2>Protocol core</h2>
    <p>
      Normative message rules, context state, freshness, and conformance language.
    </p>
  </a>
  <a class="agp-card" href="spec/discovery/">
    <h2>Discovery</h2>
    <p>
      How peers find one another, verify identities, and decide what context to exchange.
    </p>
  </a>
  <a class="agp-card" href="schemas/v1/message.schema.json">
    <h2>JSON Schemas</h2>
    <p>
      Versioned payload definitions for messages, state snapshots, and protocol errors.
    </p>
  </a>
</div>

<div class="agp-note">
  <p>
    AGP carries planning context, not tasks. It complements execution protocols
    such as MCP and A2A by helping agents understand which action paths exist,
    which are degraded, and which alternatives have appeared.
  </p>
</div>
