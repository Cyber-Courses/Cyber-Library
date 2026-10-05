---
title: "User enumeration: validating recipients over SMTP"
description: "SMTP tells you which accounts exist. VRFY and EXPN confirm and expand recipients directly, and RCPT TO after a null MAIL FROM validates addresses even when VRFY and EXPN are disabled, with response codes and timing separating valid from unknown users. Validated addresses feed spraying and spoofing."
keywords:
  - smtp user enumeration
  - VRFY
  - EXPN
  - RCPT TO
  - smtp-user-enum
---

# User enumeration

SMTP leaks the recipient namespace because the server has to decide, per address, whether it will accept mail for it. Three verbs expose that decision. `VRFY <user>` asks the server to confirm a local user: `250`/`252` means it accepts the address, `550` means it does not. `EXPN <list>` expands a mailing list or alias into its members. Both are frequently disabled, but the acceptance decision still happens during a real transaction: after `MAIL FROM:<>` (a null sender, which bounces cannot loop on), `RCPT TO:<user@domain>` returns `250` for a deliverable recipient and `550` for an unknown one. That `RCPT TO` path cannot be switched off without refusing mail, so it is the reliable enumerator. When codes are made uniform, response timing often still differs between a real mailbox lookup and an instant rejection.

## Preconditions

- Reach port 25 (or 587). Read the EHLO capabilities: `250-VRFY` and `250-EXPN` mean the easy verbs are available; their absence just moves you to `RCPT TO`.
- Know the mail domain. Pull it from the banner, the MX record, or the PTR: `dig mx victim.com +short`.
- Have a candidate list (first.last, flast, employee names from OSINT). Enumeration validates candidates, it does not invent them.

## Worked session

```bash
nc mail.victim.com 25
```
```text
220 mail.victim.com ESMTP Postfix
EHLO attacker.test
250-mail.victim.com
250-VRFY
250 8BITMIME
VRFY root
250 2.1.5 root <root@victim.com>          # 250 = valid local user
VRFY nosuchuser
550 5.1.1 <nosuchuser>: Recipient address rejected: User unknown
# VRFY disabled? fall back to the transaction path:
MAIL FROM:<>
250 2.1.0 Ok
RCPT TO:<jsmith@victim.com>
250 2.1.5 Ok                               # 250 = jsmith exists / is deliverable
RCPT TO:<notreal@victim.com>
550 5.1.1 <notreal@victim.com>: Recipient address rejected: User unknown
```

Interpret the codes: `250` accepted (valid), `550` rejected (unknown), `252` "cannot VRFY but will attempt delivery" (ambiguous: the server is telling you it accepts-all and will not confirm, so VRFY is useless here and you must judge by `RCPT TO` or timing instead).

## Automating it

```bash
# RCPT mode: most reliable, works when VRFY/EXPN are off
smtp-user-enum -M RCPT -U users.txt -D victim.com -t mail.victim.com
# VRFY mode: faster where the verb is enabled
smtp-user-enum -M VRFY -U users.txt -t mail.victim.com
# Metasploit equivalent
msfconsole -q -x "use auxiliary/scanner/smtp/smtp_enum; set RHOSTS mail.victim.com; set USER_FILE users.txt; run; exit"
```

Read the output as a valid/invalid split keyed on the `250` vs `550` difference. `smtp-user-enum` prints each username with `exists` or nothing; a run where everything "exists" means the server accepts-all (`252`-style), so the result is noise, not a user list.

## Variants and follow-on

- Exchange and Microsoft 365 do not behave like a classic MTA here: `RCPT TO` is often uniform and VRFY is gone. Enumerate those through their web endpoints and login timing instead, see [Exchange enumeration](../../../exchange/enumeration.md) and [Entra ID account enumeration](../../../../directory/entra-id/authentication/enumeration.md).
- A catch-all domain accepts every `RCPT TO`, defeating the method. Confirm by probing an obviously-bogus random address first; if it returns `250`, stop trusting `RCPT TO`.
- Feed every validated address straight into [authentication](authentication.md) for password spraying, and into any [open relay](open-relay.md) or [SMTP smuggling](smtp-smuggling.md) payload as real internal recipients.

## Tools

- [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum)
- [Metasploit smtp_enum](https://github.com/rapid7/metasploit-framework/blob/master/modules/auxiliary/scanner/smtp/smtp_enum.rb)

## References

- [RFC 5321 section 3.5 (VRFY and EXPN)](https://datatracker.ietf.org/doc/html/rfc5321#section-3.5)
- [pentestmonkey: smtp-user-enum](https://pentestmonkey.net/tools/user-enumeration/smtp-user-enum)
- [HackTricks: SMTP username enumeration](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
