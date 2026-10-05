---
title: "SPF: abusing Sender Policy Framework records to spoof the envelope sender"
description: "Reading a v=spf1 record and its mechanisms and qualifiers to find the gap that authorizes your mail: a missing record, +all or ?all, an over-broad shared-sender include:, a permissive ~all softfail, or a ten-lookup permerror. Includes dig fingerprinting and a worked swaks delivery."
keywords:
  - SPF
  - Sender Policy Framework
  - v=spf1
  - all qualifier
  - SPF include
  - envelope spoofing
---

# SPF

SPF lets a domain publish, in a single DNS TXT record, the set of IP addresses allowed to send mail using that domain in the envelope. The receiver takes the domain from the SMTP `MAIL FROM` (the return-path, formally RFC5321.From) and from the `HELO`/`EHLO` name, looks up that domain's SPF record, and checks whether the connecting IP matches. The decisive fact for an attacker: SPF never looks at the `From:` header the recipient sees, so an SPF pass alone proves nothing about the visible sender. SPF only becomes a spoofing barrier when DMARC requires the SPF identity to align with `From:`.

## Mechanism

An SPF record is a TXT record on the domain apex beginning with `v=spf1`, followed by mechanisms evaluated left to right, each carrying a qualifier:

```text
v=spf1 ip4:198.51.100.0/24 a mx include:_spf.google.com ~all
```

- **Mechanisms** say which senders match: `ip4:`/`ip6:` (literal ranges), `a` (the domain's A records), `mx` (the domain's MX hosts), `include:` (import another domain's SPF record and pass if it passes), `all` (matches everything, always last).
- **Qualifiers** prefix a mechanism and set the result on a match: `+` pass (default if omitted), `-` fail, `~` softfail, `?` neutral. So `-all` means "anything not matched above fails", `~all` means "softfail", `?all` means "neutral", `+all` means "pass for everyone".

The first matching mechanism wins. The qualifier on the terminal `all` therefore decides the fate of every sender not explicitly listed.

## Fingerprint

Pull the record and read the terminating qualifier and the includes:

```bash
dig +short TXT victim.com | grep -i spf
# "v=spf1 include:_spf.google.com include:sendgrid.net ~all"
```

Decision points from what you see:

- **No line returned** → no SPF published. The envelope domain has no authorized-sender policy; SPF cannot fail, and whether the spoof lands depends entirely on DMARC/DKIM.
- **`+all` or `?all`** → any IP passes (`+all`) or is neutral (`?all`) for this domain. You can send as the domain from your own host and SPF will not object.
- **`~all`** → softfail. Non-listed senders "softly" fail, which a large fraction of receivers still deliver (often straight to the inbox, sometimes to spam). Worth sending to confirm.
- **`-all` with broad `include:`** → the terminal is strict, but a shared-sender include (`sendgrid.net`, `spf.mandrillapp.com`, `_spf.google.com`, `servers.mcsv.net` for Mailchimp) authorizes the *entire* provider's outbound pool. If you can send through that provider (a free or trial account on the same platform), your mail egresses from an IP inside the included range and SPF passes for the victim domain.
- **Many `include:`/`a`/`mx` terms** → count the DNS lookups (see below); exceeding ten yields a permerror.

## Worked: confirm and deliver

Count the lookups first when the record is long. Each `include`, `a`, `mx`, `ptr`, and `exists` term costs one DNS lookup, nested includes count cumulatively, and the limit is ten. Over ten is a `permerror`, which most receivers treat as *not a fail* (neutral), so a bloated record effectively disables SPF enforcement:

```bash
# a quick count of lookup-consuming terms across the record and its nested includes
dig +short TXT victim.com | tr ' ' '\n' | grep -E 'include:|^a$|^a:|^mx$|^mx:|ptr|exists:'
# pyspf resolves the full tree and reports permerror explicitly
python3 -c "import spf,sys; print(spf.check2(i='203.0.113.9', s='x@victim.com', h='mail.victim.com'))"
# ('permerror', 'Too many DNS lookups') => enforcement is effectively off
```

When the terminal is `~all`, `?all`, or `+all` (or the record permerrors), send a message with the victim domain in the envelope and the `From:` header and show it is accepted. `swaks` drives the full SMTP transaction so you can read the response code:

```bash
swaks --to target.user@recipient.com \
      --from notifications@victim.com \
      --helo victim.com \
      --header "Subject: Account notice" \
      --body "Test." \
      --server mx.recipient.com
# ... <-  MAIL FROM:<notifications@victim.com>
# ... -> 250 2.1.0 Ok
# ... <-  RCPT TO:<target.user@recipient.com>
# ... -> 250 2.1.5 Ok
# ... -> 250 2.0.0 Ok: queued           <- accepted; check the delivered Authentication-Results
```

Read the result in the delivered message's `Authentication-Results:` header: `spf=pass` on `~all`/`+all`/permerror confirms the envelope domain is yours to use. Here `swaks --from` sets both the envelope and the header `From:` to `victim.com`, so this is a direct envelope spoof that stands or falls on SPF alone (and then DMARC). To separate the two identities (SPF-pass on a domain you control while the header spoofs the victim), see [Sender spoofing](sender-spoofing.md).

## Follow-on

An SPF pass is only half the story. If the victim's SPF lets you send as its envelope domain, the spoof survives only when DMARC does not demand DKIM alignment too; read [DMARC](dmarc.md) next to confirm the policy. If SPF is strict but a shared `include:` is in play, pivot to abusing that provider's pool. Either way, the message is actually built and sent on the [Sender spoofing](sender-spoofing.md) page.

## Tools

- [swaks](https://github.com/jetmore/swaks)
- [pyspf](https://github.com/sdgathman/pyspf)

## References

- [RFC 7208 (Sender Policy Framework)](https://datatracker.ietf.org/doc/html/rfc7208)
- [RFC 7208 section 4.6.4 (DNS lookup limits)](https://datatracker.ietf.org/doc/html/rfc7208#section-4.6.4)
- [HackTricks: email spoofing (SPF)](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/phishing-methodology/index.html)
