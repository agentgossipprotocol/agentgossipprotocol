---
title: Transport
weight: 4
---

# Transport

**Status:** Draft

This document defines wire-level transport expectations for the Agent Gossip
Protocol. The protocol is transport-agnostic where possible; this document
specifies the requirements a transport binding MUST satisfy.

RFC 2119 keyword conventions are inherited from [Core](core/).

## 1. Transport Requirements

A conforming transport binding MUST provide:

> **TODO:** Enumerate requirements (ordering, framing, reliability,
> confidentiality, bidirectionality).

## 2. Supported Bindings

> **TODO:** Specify concrete bindings (e.g., WebSockets, HTTP) and note which
> are REQUIRED vs. OPTIONAL for conformance.

## 3. Framing and Encoding

Messages MUST be encoded such that they validate against the version 1
schemas. 

> **TODO:** Define on-the-wire encoding (e.g., JSON, length-prefixed framing).

## 4. Connection Lifecycle

> **TODO:** Define connect, keep-alive, and teardown behavior.

## 5. Security Binding

Transport-level security requirements are defined in [Security](security/).
