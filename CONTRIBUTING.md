# Contributing to the Agent Gossip Protocol

Thank you for your interest in improving the Agent Gossip Protocol (AGP).

## Ways to contribute

- **Fix or clarify prose** in `spec/` — open a pull request directly.
- **Propose a substantive change** to the protocol — file a Specification
  Enhancement Proposal (SEP). See below.
- **Improve schemas** in `schemas/` — changes to normative schemas MUST be
  accompanied by a SEP.

## Proposing changes (SEPs)

Substantive changes go through the SEP process described in
[`proposals/_index.md`](proposals/_index.md):

1. Copy `proposals/0000-template.md` to `proposals/NNNN-short-title.md`.
2. Fill in every section, including **backwards-compatibility** implications
   and the **SemVer** impact.
3. Open a pull request for discussion.

## Specification style

- Use [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) keywords
  (MUST, SHOULD, MAY, ...) in all capitals only when stating requirements.
- Describe the protocol on the wire, not any particular implementation.
- Keep prose and schemas consistent; the schema is normative for payload
  structure.

## Licensing and IP of contributions

By contributing you agree that:

- Documentation/specification contributions are licensed under **CC-BY-4.0**.
- Code and schema contributions are licensed under **Apache-2.0**, including
  its royalty-free patent grant.

This means anyone may implement the specification royalty-free. Do not submit
material encumbered by patents you are unwilling to license on these terms.

## Developer setup

The site is built with [Hugo](https://gohugo.io/) (extended). To preview:

```sh
hugo server
```
