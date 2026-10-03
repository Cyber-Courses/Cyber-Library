---
title: "Net-NTLM capture and poisoning: harvesting authentication from the wire"
description: "Capturing NetNTLMv2 challenge-responses by poisoning broadcast name resolution (LLMNR, NBT-NS, mDNS) and answering for names that fail to resolve, forcing hosts to authenticate to an attacker-controlled listener."
keywords:
  - LLMNR poisoning
  - NBT-NS
  - Responder
  - NetNTLMv2
  - mDNS
---

# Net-NTLM capture and poisoning

When a Windows host tries to reach a name that DNS cannot resolve (a typo, a decommissioned share, a WPAD lookup), it falls back to **broadcast** name-resolution protocols: LLMNR, NBT-NS, and mDNS. These have no authentication: any host on the segment can answer "that's me". By answering, you make the victim connect to you and authenticate, handing you a **NetNTLMv2** challenge-response you can crack offline or relay.

**Lineage.** The poisoning surface is a relic of WINS and NetBIOS name resolution, whose broadcast fallback LLMNR and NBT-NS inherited. Responder made abusing it routine from 2012, and mDNS and IPv6 (mitm6, rogue DHCPv6 and WPAD) are the same idea carried onto newer stacks.

## Why it works

- **LLMNR** (UDP 5355) and **NBT-NS** (UDP 137) are multicast/broadcast fallbacks with no source validation.
- **mDNS** (UDP 5353) behaves similarly for `.local` names.
- A mistyped share, a stale drive mapping, or **WPAD** proxy auto-discovery produces a constant stream of failed lookups to poison.
- When the victim connects to your fake SMB/HTTP service, Windows sends a NetNTLMv2 response automatically, often for a logged-on user and sometimes for a privileged account.

## Capturing

```bash
# Responder: answer LLMNR/NBT-NS/mDNS and run rogue SMB/HTTP/etc. to capture NetNTLMv2
responder -I eth0 -wv

# Responder writes captured hashes to per-module files under logs/ (not the
# session log), e.g. logs/SMB-NTLMv2-SSP-<ip>.txt, in hashcat -m 5600 format
hashcat -m 5600 logs/SMB-NTLMv2-SSP-10.0.0.5.txt wordlist.txt -r rules/best64.rule
```

Target WPAD specifically: many networks never set a real `wpad` DNS record, so every browser's proxy lookup can be answered, capturing authentication broadly.

## From capture to impact

A captured NetNTLMv2 response has two uses:

- **Crack it** (`-m 5600`) to recover the plaintext, then reuse the credential for [spraying](../credentials/password-spraying.md) and authenticated access. Feasible only if the password is weak.
- **[Relay](relay.md) it** without cracking: forward the authentication in real time to another service and act as the victim there. This is the higher-value path, because it works regardless of password strength, as long as the target does not enforce signing.

## Exploitation notes

- NetNTLMv2 is **not** reusable like an NT hash: it is bound to a server challenge, so it only cracks or relays, never pass-the-hash.
- Machine-account responses (`HOST$`) are common and useful: a machine account's NetNTLM can be relayed even though its password is random and uncrackable.
- `mitm6` abuses IPv6 autoconfiguration (rogue DHCPv6 + DNS) to redirect name resolution more reliably than broadcast poisoning in IPv6-enabled networks, and pairs with relay.

## Tools

- **Responder**: LLMNR/NBT-NS/mDNS poisoning with rogue capture servers.
- **mitm6**: IPv6 DNS takeover to redirect and capture/relay authentication.
- **Inveigh**: PowerShell/C# poisoner for Windows attack hosts.

## References

- [Responder (lgandx): the maintained LLMNR/NBT-NS/mDNS poisoner](https://github.com/lgandx/Responder)
- [Inveigh (Kevin-Robertson): .NET IPv4/IPv6 MITM poisoner](https://github.com/Kevin-Robertson/Inveigh)
- [mitm6 (dirkjanm): IPv6 DNS takeover](https://github.com/dirkjanm/mitm6)
- [The Hacker Recipes: LLMNR/NBT-NS/mDNS poisoning](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/llmnr-nbtns-mdns-spoofing)
