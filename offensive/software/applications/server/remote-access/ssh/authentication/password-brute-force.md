---
title: "Password brute force: online password attacks against SSH"
description: "Where the effective SSH policy permits password authentication, authorized testing can assess password quality and attempt controls. Method availability may vary by user and source, while MaxAuthTries, MaxStartups, PerSourcePenalties, and external controls affect online attempts."
keywords:
  - ssh brute force
  - password spray
  - hydra
  - maxauthtries
  - credential reuse
---

# Password brute force

When the effective server policy permits password authentication for a particular user and source, an authorized assessment can test password quality and resistance to online guessing. `MaxAuthTries` limits attempts per connection, `MaxStartups` limits concurrent unauthenticated connections, and current OpenSSH versions can apply per-source penalties; external monitoring or blocking may add further controls. A valid password still may not provide a shell when another authentication method is required, the account is restricted, or the server forces a command.

```bash
# inspect the methods reported for this user and source without prompting
ssh -vvv -o BatchMode=yes -o PreferredAuthentications=password \
  -o PubkeyAuthentication=no user@<target> true 2>&1 \
  | sed -n 's/^debug1: Authentications that can continue: //p'
# spray one password across users (quiet), stop on success, few threads
hydra -L users.txt -p 'Autumn2025!' ssh://<target> -t 4 -f
nxc ssh <target> -u users.txt -p 'Autumn2025!'          # marks shell vs no-shell on hit
# brute one high-value account with a short, targeted list
hydra -l admin -P top-passwords.txt ssh://<target> -t 4 -f
```

## Exploitation notes

- Check the effective method chain first. A globally enabled `PasswordAuthentication` setting does not prove that it applies to every user or that password alone is sufficient.
- Keep threads low and the password list short and contextual: SSH logs every failure, `MaxAuthTries` closes the connection after a few tries, and IP-blocking tools react to bursts.
- Validate usernames first ([User enumeration](../enumeration/user-enumeration.md)) so attempts land on real accounts; derive passwords from the organisation (season/year, company name, breached creds) rather than a generic giant wordlist.
- Treat credential reuse as a separate, authorized hypothesis. A successful authentication may still be restricted to a subsystem, forced command, or non-interactive account.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec (ssh)](https://github.com/Pennyw0rth/NetExec)

## References

- [OpenSSH sshd_config](https://man.openbsd.org/sshd_config)
- [OpenSSH ssh_config](https://man.openbsd.org/ssh_config)
