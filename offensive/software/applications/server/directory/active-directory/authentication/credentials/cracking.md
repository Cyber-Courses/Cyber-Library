---
title: "Cracking: turning recovered hashes into passwords offline"
description: "Cracking Active Directory credential material offline: NTLM hashes, Kerberos roasting tickets (etype 23 RC4 and etype 17/18 AES), cached domain credentials (DCC2), and NetNTLMv2, with hashcat modes and strategy."
keywords:
  - hashcat
  - NTLM cracking
  - kerberoast hash
  - DCC2
  - NetNTLMv2
---

# Cracking

Many of the secrets you recover are hashes, not passwords. Cracking is the offline step that turns a hash into the plaintext you can type, spray, or reuse where the hash itself is not directly usable (roasting tickets, cached logons, captured NetNTLM). It is done on your own hardware, so it is undetectable to the target and bounded only by the password's strength and your wordlists.

## What cracks, and how fast

The hash format decides feasibility. From fastest to slowest:

- **NTLM** (`hashcat -m 1000`): unsalted and fast; weak passwords fall quickly. But a strong NTLM hash is usually better *used* (pass-the-hash) than cracked.
- **Kerberoast TGS, RC4 (etype 23)** (`-m 13100`): fast, the reason Kerberoasting is so effective against service accounts.
- **Kerberoast TGS, AES (etype 17/18)** (`-m 19600 / 19700`): much slower; a domain that forces AES on service accounts blunts roasting.
- **AS-REP roast** (`-m 18200`): similar speed to RC4 Kerberoast.
- **NetNTLMv2** (`-m 5600`): captured from [poisoning/relay](../ntlm/net-ntlm-capture-and-poisoning.md); crackable but slower than NTLM.
- **DCC2 / MSCache2** (`-m 2100`): cached domain logons, deliberately slow and salted, so only weak passwords are realistic.

```bash
hashcat -m 13100 kerberoast.hash wordlist.txt -r rules/best64.rule     # Kerberoast RC4
hashcat -m 18200 asrep.hash wordlist.txt                                # AS-REP roast
hashcat -m 5600 netntlmv2.hash wordlist.txt                             # captured NetNTLMv2
```

## Strategy

- **Wordlists plus rules** beat brute force for human passwords; large real-world lists (rockyou and breach compilations) with a rule set recover the bulk of weak passwords.
- **Target the organization**: company name, season-year, and locale-specific patterns (mutated with rules) catch policy-compliant-but-predictable passwords that generic lists miss.
- **Spend effort where the hash is fast and the account is privileged**: an RC4 Kerberoast of a Domain Admin service account is the highest-value crack; do not burn cycles on AES hashes or DCC2 unless the password looks weak.

## Exploitation notes

- Cracking is unlimited and silent, so a captured roast or hash costs the defender nothing to issue but can be worked indefinitely offline.
- A cracked service-account or admin password is reusable everywhere that account is valid and feeds directly into [password spraying](password-spraying.md) of related accounts.
- If AES is enforced, pivot from cracking to *using* the material (pass-the-hash, pass-the-key, ticket reuse) rather than fighting the slow hash.

## Tools

- **hashcat**: GPU cracking across all the modes above.
- **John the Ripper**: alternative, with `--format` equivalents and good rule support.
- **hcxtools / name-that-hash**: identify unknown hash formats before cracking.

## References

- The Hacker Recipes: cracking
- hashcat: example hashes and modes
