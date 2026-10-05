---
title: "Authentication: WinRM passwords, NTLM, and pass-the-hash"
description: "WinRM authenticates with Windows credentials over Negotiate/NTLM, Kerberos, or Basic. Because it accepts NTLM, an attacker authenticates with an NT hash (pass-the-hash) as well as a password, and a Kerberos ticket also works. Spraying and reusing credentials against WinRM is a standard lateral-movement step; only authorised accounts can connect."
keywords:
  - winrm authentication
  - pass-the-hash
  - ntlm
  - kerberos
  - spray
---

# Authentication

WinRM authenticates with Windows credentials over several mechanisms: Negotiate (which selects Kerberos or NTLM), raw NTLM, Kerberos, Basic (password in an HTTP header, used over HTTPS), and CredSSP. The important consequence is that WinRM accepts NTLM, so an attacker authenticates with an NT hash via pass-the-hash exactly as with a password, and a Kerberos ticket (from a prior compromise) works through Negotiate. The practical attack is therefore credential and hash reuse: spray passwords, replay captured or dumped NT hashes, or present a Kerberos ticket, against WinRM for lateral movement. The constraint is authorisation: only local admins and Remote Management Users may connect, so a valid credential is not always a usable one.

```bash
# password, NT hash (pass-the-hash), and ticket all work
nxc winrm <target> -u user -p 'Password1'
nxc winrm <target> -u user -H <nthash>                 # pass-the-hash
evil-winrm -i <target> -u user -H <nthash>             # hash auth into a shell
evil-winrm -i <target> -u user -p 'Password1'
# Kerberos: export a ticket and use Negotiate
export KRB5CCNAME=ticket.ccache; evil-winrm -i <target> -u user -r <REALM>
```

## Exploitation notes

- Pass-the-hash is the headline: WinRM's NTLM support means a dumped NT hash authenticates without cracking, so WinRM is a primary destination for hashes harvested elsewhere.
- Authorisation gates usage: the account must be a local administrator or in Remote Management Users on the target; `nxc winrm` distinguishes a valid-but-unauthorised account from a usable one.
- Spray and reuse as with any Windows service, respecting lockout; a hit that `nxc` marks `Pwn3d!` is admin via WinRM and gives an immediate privileged shell.
- Kerberos authentication avoids NTLM entirely and suits environments that log/limit NTLM; present a TGT/service ticket through Negotiate.

## References

- [HackTricks: WinRM authentication](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
