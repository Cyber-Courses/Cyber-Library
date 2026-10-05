---
title: "DMARC: policy and alignment gaps that let a From: spoof through"
description: "Reading a _dmarc record and deciding the spoofing path by its policy: no record, p=none, pct below 100, a missing or weak sp= that leaves subdomains open, relaxed alignment that lets a sibling identity pass, and cousin domains outside DMARC scope entirely. Includes dig fingerprinting and a worked subdomain spoof."
keywords:
  - DMARC
  - DMARC alignment
  - p=none
  - sp subdomain policy
  - pct
  - relaxed alignment
---

# DMARC

DMARC is the only one of the three checks that protects the identity the recipient actually reads, the `From:` header. It works on top of SPF and DKIM: a message passes DMARC when SPF or DKIM passes *and* the identity that passed *aligns* with the `From:` domain. It then tells the receiver what to do when neither aligns, and where to send reports. Because DMARC is the binding step, its policy record is the single most important thing to read against a target: a domain with strict SPF and DKIM but a weak DMARC policy is still spoofable in the `From:` header.

## Mechanism

DMARC is a TXT record at `_dmarc.<domain>`:

```text
v=DMARC1; p=reject; sp=none; adkim=r; aspf=r; pct=100; rua=mailto:dmarc@victim.com
```

- `p=` is the policy for the organizational domain: `none` (take no action, just report), `quarantine` (treat as suspicious, usually spam folder), `reject` (refuse at SMTP).
- `sp=` is the policy for subdomains. When `sp=` is absent, subdomains inherit `p=`. When present, it overrides `p=` for subdomains only.
- `pct=` is the percentage of messages the policy is applied to (default 100). The rest are treated as the next-lower policy.
- `adkim=`/`aspf=` set alignment strictness: `r` relaxed (the default) matches on the organizational domain, so `mail.victim.com` aligns with `victim.com`; `s` strict requires an exact domain match.

Alignment is what connects SPF/DKIM to `From:`. SPF alignment compares the envelope `MAIL FROM` domain to the `From:` domain; DKIM alignment compares the signature `d=` to the `From:` domain. Only an aligned pass counts for DMARC.

## Fingerprint

Pull the record and read every tag:

```bash
dig +short TXT _dmarc.victim.com
# "v=DMARC1; p=reject; sp=none; adkim=r; aspf=r; pct=100; rua=mailto:dmarc@victim.com"
```

Decide the path from what the policy says:

- **No record returned** → the domain publishes no DMARC. Receivers have nothing binding the `From:` header, so an unaligned message is judged on SPF/DKIM alone (and many deliver it). Spoof the `From:` directly.
- **`p=none`** → monitor-only. The domain is collecting reports but asks receivers to take no action, so unaligned spoofs are delivered. This is the most common live gap on real domains.
- **`pct=` below 100** → only that fraction of mail gets `p`; the remainder drops to the lower policy (for `p=reject; pct=25`, three quarters of messages get at most `quarantine`). Resend the same spoof repeatedly; a share lands under the softer treatment.
- **`sp=none` (or `sp=quarantine` while `p=reject`)** → the organizational domain is enforced but subdomains are not. Spoof a subdomain (below).
- **`adkim=r`/`aspf=r` (relaxed, the default)** → any subdomain identity aligns with the parent. If you can obtain an SPF pass or a DKIM signature for *any* subdomain (a marketing subdomain delegated to an email provider you can send through, a forgotten host with its own `+all` SPF), a `From: anything@victim.com` aligns under relaxed and passes.
- **`p=reject; sp=reject; adkim=s; aspf=s`** → fully locked. Move to lookalike and display-name tactics on [Sender spoofing](sender-spoofing.md), which DMARC cannot touch.

## Worked: spoof a subdomain under sp=none

When `sp=none` (or the parent is `p=none`), a subdomain of the target is unprotected, including subdomains that do not exist, because the receiver walks up to the organizational `_dmarc` record and applies `sp`. Send with a subdomain in both the envelope and the `From:` header:

```bash
# parent enforced, subdomains not: pick a plausible or non-existent subdomain
swaks --to target@recipient.com \
      --from alerts@notifications.victim.com \
      --header "From: \"IT Service Desk\" <helpdesk@noexist.victim.com>" \
      --header "Subject: Mailbox migration" \
      --body "..." \
      --server mx.recipient.com
```

Read the delivered `Authentication-Results:`. The message still shows `dmarc=fail`, because neither SPF nor DKIM aligns with the `From:` domain; what changes is the disposition. The subdomain's applicable policy is `none` (set explicitly by `sp=none`, or inherited when the parent is `p=none`), so the receiver's requested action is to deliver anyway. The spoof lands not because DMARC passed but because the policy for that subdomain asks for no action on failure. The recipient sees a `victim.com` subdomain in `From:`, and because relaxed alignment is the default it still reads as the trusted organization to a human.

## Worked: ride relaxed alignment from a sibling

Under relaxed alignment you do not need the exact `From:` domain to pass SPF or DKIM, only its organizational parent. If the target delegates a subdomain to a bulk sender (common for `email.victim.com`, `mktg.victim.com`) and you can send through that platform, your mail gets an aligned SPF/DKIM pass for the subdomain, which under `adkim=r`/`aspf=r` aligns with a `From: ceo@victim.com`:

```bash
# confirm a delegated subdomain with its own permissive SPF or a provider include you can use
dig +short TXT email.victim.com | grep -i spf     # e.g. "v=spf1 include:sendgrid.net -all"
# send through that provider so egress IP + envelope land inside the subdomain's authorized set,
# then set the visible From: to the bare parent domain; relaxed alignment makes it pass
```

Cousin or lookalike domains (`victim-support.com`, a punycode homoglyph of `victim.com`) are a different organizational domain entirely, so DMARC does not apply to them at all; those belong to the display tactics on [Sender spoofing](sender-spoofing.md).

## Follow-on

The policy you read here decides which earlier gap you actually need: `p=none` or no record means any `From:` spoof is delivered with no SPF/DKIM work at all; an enforced `p` sends you back to [SPF](spf.md) or [DKIM](dkim.md) to obtain an *aligned* pass. Once the path is clear, build and send the message on [Sender spoofing](sender-spoofing.md).

## Tools

- [swaks](https://github.com/jetmore/swaks)
- [checkdmarc](https://github.com/domainaware/checkdmarc)

## References

- [RFC 7489 (DMARC)](https://datatracker.ietf.org/doc/html/rfc7489)
- [RFC 7489 section 3.1 (identifier alignment)](https://datatracker.ietf.org/doc/html/rfc7489#section-3.1)
- [RFC 7489 section 6.3 (policy record tags)](https://datatracker.ietf.org/doc/html/rfc7489#section-6.3)
