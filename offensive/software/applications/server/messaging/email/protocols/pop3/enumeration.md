---
title: "Enumeration: fingerprinting a POP3 service and its users"
description: "Enumerating POP3 before authentication: the greeting banner identifying qpopper, Courier, or Dovecot and exposing the APOP timestamp, the CAPA response listing STLS/SASL/UIDL, and username validation through differential USER/PASS responses and timing. Reads +OK and -ERR in a worked session."
keywords:
  - pop3 enumeration
  - capa
  - apop timestamp
  - pop3 user enumeration
  - stls
---

# Enumeration

POP3 leaks its identity in the first line and its abilities in one command. The greeting banner often names the implementation, and if the server supports APOP it embeds a per-connection timestamp token right there in the greeting. `CAPA` lists whether STARTTLS (`STLS`) is offered, which SASL mechanisms exist, and whether `UIDL`/`TOP` are available. The `USER`/`PASS` two-step is also a username oracle on servers that distinguish "no such user" from "bad password". All of this is unauthenticated.

## Fingerprint first

The server sends a `+OK` greeting on connect. On an APOP-capable server the greeting contains a `<...>` token, the timestamp used for the APOP digest.

```bash
nc <target> 110
+OK POP3 mail.corp.local v2020.1 server ready <1896.697170952@mail.corp.local>
CAPA
+OK Capability list follows
TOP
USER
UIDL
SASL PLAIN LOGIN
STLS
APOP
.
```

Read this:

- The greeting string frequently names the product (`qpopper`, `Courier`, `Dovecot ready`). A `<token@host>` present in the greeting means APOP is offered and gives you the challenge if you want to attack it.
- `STLS` means STARTTLS is available on 110; its absence on a cleartext port means `USER`/`PASS` will cross the wire in plaintext (useful to know for sniffing, and for whether the server forces TLS).
- `SASL` lists the `AUTH` mechanisms; `UIDL` and `TOP` confirm you can list unique IDs and peek at headers during [mailbox access](mailbox-access.md).

On 995 wrap it in TLS:

```bash
openssl s_client -connect <target>:995 -quiet
CAPA
```

## Username validation

Classic POP3 implementations answer `USER` by itself with `+OK` regardless, then reveal the account's existence at the `PASS` step or via timing. Where the server (or the SASL back end behind it) returns a distinct error for an unknown user, the login becomes a user oracle.

```bash
nc <target> 110
USER bob
+OK
PASS x
-ERR [AUTH] Authentication failed.          # account exists, bad password
USER nobody
+OK
PASS x
-ERR [SYS/PERM] No such user.                # account does not exist (older servers)
```

Modern Dovecot normalizes both to the same message and adds delay, killing the wording differential, so fall back to timing: a server that validates the user before hashing the password answers a known account more slowly than an unknown one.

```bash
for u in $(cat users.txt); do
  t=$( { /usr/bin/time -f %e sh -c \
    "printf 'USER $u\r\nPASS x\r\nQUIT\r\n' | openssl s_client -connect <target>:995 -quiet 2>/dev/null >/dev/null"; } 2>&1 )
  echo "$u $t"
done | sort -k2 -n
```

Legacy qpopper and older Courier are where explicit "no such user" wording survives; sort the timing output for the modern uniform servers.

## Follow-on

Carry the validated usernames and the APOP token/`SASL` list into [authentication](authentication.md), and the banner's product/version into [server exploitation](server-exploitation.md).

## Tools

- [nmap pop3-capabilities / pop3-ntlm-info](https://nmap.org/nsedoc/scripts/pop3-capabilities.html)

## References

- [RFC 1939: APOP and the greeting timestamp](https://datatracker.ietf.org/doc/html/rfc1939#section-7)
- [RFC 2449: POP3 CAPA command](https://datatracker.ietf.org/doc/html/rfc2449)
- [HackTricks: pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
