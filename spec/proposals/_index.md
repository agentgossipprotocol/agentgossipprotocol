---
title: Proposals
weight: 2
cascade:
  type: docs
---

# Specification Enhancement Proposals (SEPs)

Changes to the Agent Gossip Protocol are proposed and ratified through
**Specification Enhancement Proposals** (SEPs), modeled on Python's PEPs and
MCP's SEPs.

## Process

1. Copy [`0000-template.md`](0000-template.md) to `NNNN-short-title.md`, using
   the next available four-digit number.
2. Fill in every section. A proposal MUST state the problem, the proposed
   schema/protocol change, and backwards-compatibility implications.
3. Open a pull request. Discussion happens on the PR.
4. On acceptance, the proposal is merged with status **Accepted** and, once
   implemented in the spec, marked **Final** with the target spec version.

## Status Values

`Draft` → `Accepted` → `Final`, or `Rejected` / `Withdrawn`.

## Index

| SEP | Title | Status |
| --- | --- | --- |
| [0001](0001-initial.md) | Initial design | Draft |
