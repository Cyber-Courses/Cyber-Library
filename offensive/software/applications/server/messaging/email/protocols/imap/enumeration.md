---
title: "Enumeration: fingerprinting an IMAP service and its users"
description: "Enumerating IMAP before authentication: the untagged CAPABILITY response exposing LOGINDISABLED, STARTTLS, and AUTH= mechanisms, the greeting banner identifying Dovecot, Cyrus, or Courier, and username validation through differential LOGIN and AUTHENTICATE responses. Reads tagged OK/NO/BAD replies in a worked session."
keywords:
  - imap enumeration
  - imap capability
  - dovecot banner
  - imap user enumeration
  - loginDisabled
---

# Enumeration

IMAP tells you most of what you need before you authenticate. The greeting banner frequently names the implementation and build, the `CAPABILITY` response lists whether cleartext login is disabled, whether STARTTLS is required, and which SASL mechanisms are offered, and the way the server answers a login attempt for a known versus unknown user can leak valid usernames. All of this is unauthenticated and each result selects the next step: a mechanism list to spray, a version to match against implementation flaws, or a user list to feed authentication.

## Fingerprint first

The server sends an untagged greeting (`* OK`) the moment the connection opens, often with the product string. Then a client-tagged `CAPABILITY` lists the server's abilities. The tag is any short token you pick; the server echoes it in the final line.

```bash
nc <target> 143
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE STARTTLS LOGINDISABLED AUTH=PLAIN] Dovecot ready.
A1 CAPABILITY
* CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE STARTTLS LOGINDISABLED AUTH=PLAIN
A1 OK Pre-login capabilities listed, post-login capabilities have more.
```

Read this closely:

- `Dovecot ready.` identifies the implementation; Cyrus answers with `* OK ... Cyrus IMAP ...`, Courier with `Courier-IMAP ready`.
- `LOGINDISABLED` means the `LOGIN` command (cleartext user/pass) is refused on this cleartext channel; you must negotiate `STARTTLS` first, or connect to 993.
- `AUTH=PLAIN`, `AUTH=LOGIN`, `AUTH=CRAM-MD5`, `AUTH=NTLM` enumerate the SASL mechanisms accepted by `AUTHENTICATE`. `AUTH=NTLM` (or `imap-ntlm-info`) leaks the domain and host names from the NTLM type-2 challenge.

On 993 wrap the same exchange in TLS:

```bash
openssl s_client -connect <target>:993 -quiet
a CAPABILITY
```

## Username validation

Where the server returns a materially different response for a valid versus invalid username, the login surface becomes a user oracle. Two signals recur: wording of the `a NO` text, and timing. A server that validates the user before checking the password can take measurably longer (or phrase the rejection differently) when the account exists.

```bash
openssl s_client -connect <target>:993 -quiet
a LOGIN alice wrongpass
a NO [AUTHENTICATIONFAILED] Authentication failed.
a LOGIN nosuchuser wrongpass
a NO [AUTHENTICATIONFAILED] Authentication failed.
```

Dovecot is deliberately uniform here: identical text and artificial delay for both cases, so the differential is usually dead. Older Courier and misconfigured SASL back ends (for example an IMAP front end proxying to a back end that answers "user unknown" faster than "bad password") are where timing and wording splits appear. Script the timing delta rather than eyeballing it:

```bash
for u in $(cat users.txt); do
  t=$( { /usr/bin/time -f %e sh -c \
    "printf 'a LOGIN $u x\r\n' | openssl s_client -connect <target>:993 -quiet 2>/dev/null >/dev/null"; } 2>&1 )
  echo "$u $t"
done | sort -k2 -n        # outliers at either end are candidate valid users
```

A SASL `AUTHENTICATE` probe is the other oracle: some stacks answer an unknown user at the mechanism step rather than after the credential, so an unknown user fails earlier in the exchange than a known one.

## Follow-on

Feed the validated usernames and the `AUTH=` mechanism list into [authentication](authentication.md). If the banner named a specific Dovecot, Cyrus, or Courier build, carry it to [server exploitation](server-exploitation.md).

## Tools

- [nmap imap-capabilities / imap-ntlm-info](https://nmap.org/nsedoc/scripts/imap-capabilities.html)

## References

- [RFC 3501: CAPABILITY command](https://datatracker.ietf.org/doc/html/rfc3501#section-6.1.1)
- [RFC 3501: LOGINDISABLED](https://datatracker.ietf.org/doc/html/rfc2595)
- [HackTricks: pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
