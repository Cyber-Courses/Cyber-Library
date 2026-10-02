---
title: "gRPC reflection, metadata headers, and protobuf dispatch surfaces"
description: Exposed reflection APIs, large protobuf fields, and metadata trusted as security decisions in gRPC services.
keywords:
  - gRPC
  - protobuf
---

# gRPC reflection and metadata

## Context

**Server reflection** exposes service definitions in production if left enabled. **Metadata** (`:authority`, custom headers) may be **trusted** like HTTP headers for tenant routing, same **[Trust boundary](../../../access-control/trust-boundary/index.md)** pitfalls.

## Theory

Map unary vs streaming handlers; unbounded **message** size and **streaming** fan-out can amplify abuse.
