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
  A transport-independent protocol for multi-agent systems, with a preferred
  A2A binding. Agents gossip expiring, attributed operational evidence —
  lifecycle, route quality, evaluation scores, referrals — so each
  participant can choose better among paths it has already discovered and is
  already authorized to use.
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
      The gossip record model, the four v1 record kinds, exclusions, and conformance rules.
    </p>
  </a>
  <a class="agp-card" href="spec/transport/">
    <h2>Sync operation</h2>
    <p>
      The single resumable anti-entropy operation that exchanges records, and its A2A binding.
    </p>
  </a>
  <a class="agp-card" href="schemas/v1/record.schema.json">
    <h2>JSON Schemas</h2>
    <p>
      Versioned payload definitions for records, gossip frames, selectors, and protocol errors.
    </p>
  </a>
</div>

<div class="agp-note">
  <p>
    AGP carries evidence, not tasks. It never discovers what exists, never grants
    access, and never overrides configuration — it only helps an agent choose
    among action paths it has already discovered and is already authorized to
    use, based on fresh, attributed, expiring observations.
  </p>
</div>
