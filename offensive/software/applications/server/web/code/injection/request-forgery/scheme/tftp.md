---
title: "tftp:// URLs and SSRF: UDP-based TFTP in custom or embedded outbound clients (rare)"
description: Rare tftp:// URL support in outbound clients; UDP-based and uncommon in typical HTTP libraries.
keywords:
  - SSRF
  - TFTP
---

# TFTP (SSRF)

TFTP over `tftp://` is uncommon in application HTTP clients. Document the scheme for completeness when a **custom** or **embedded** client lists it in supported protocols. Test only in isolated environments.
