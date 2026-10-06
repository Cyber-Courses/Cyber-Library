---
title: "Sender authentication: spoofing past SPF, DKIM, and DMARC"
order: 1
description: "How receivers check SPF, DKIM, and DMARC on inbound mail, and how an attacker reads the published records to find the gap that lets a message spoof a target domain in the visible From: header and still pass. Orients the SPF, DKIM, DMARC, and sender-spoofing children."
keywords:
  - sender authentication
  - SPF
  - DKIM
  - DMARC
  - email spoofing
  - DMARC alignment
---

# Sender authentication

A receiving mail server decides whether an inbound message is really from the domain it claims by running three checks against DNS records the sending domain publishes. SPF asks whether the sending IP is authorized for the envelope `MAIL FROM` (and `HELO`) domain. DKIM verifies a cryptographic signature over selected headers and the body against a public key the domain publishes. DMARC ties the two to the one identity a human actually sees, the `From:` header, by requiring that a passing SPF or DKIM identity *aligns* with the `From:` domain, and it declares what to do when neither aligns.

The attacker goal is to deliver a message whose visible `From:` is a target domain (so the recipient trusts it) and that still clears these checks, or that is not subject to a blocking policy. Every path to that outcome is a gap in one of the three records, so the first move against any target is to pull all three and read them. The key asymmetry to keep in mind: SPF and DKIM authenticate identities the recipient never sees (the envelope sender, the signing domain), and only DMARC binds anything to the `From:` header. A domain with perfect SPF and DKIM but no enforcing DMARC is wide open to `From:` spoofing.

## Triage

Pull the three records and read the policy before anything else. The DKIM selector is not guessable in general, take it from the `DKIM-Signature: ... s=` tag of any mail you have received from the target, or brute common names (see [DKIM](dkim.md)).

```bash
dig +short TXT victim.com | grep -i spf                 # SPF: mechanisms + the all qualifier (-all / ~all / ?all / +all)
dig +short TXT _dmarc.victim.com                        # DMARC: p=, sp=, pct=, adkim/aspf alignment mode
dig +short TXT selector._domainkey.victim.com           # DKIM: public key for a known selector (k=, p=, key length)
```

Read and route:

- **No SPF record, or `+all` / `?all` / a sendable `include:`** → the envelope domain can be spoofed directly. Go to [SPF](spf.md).
- **DMARC missing or `p=none`** → the `From:` header is not enforced, spoofs are delivered regardless of SPF/DKIM. Go to [DMARC](dmarc.md).
- **`p=quarantine`/`p=reject` but `sp=none`, `pct<100`, or relaxed alignment** → subdomains, a fraction of traffic, or sibling identities slip through. Go to [DMARC](dmarc.md).
- **No DKIM, a short key, a `l=` body-length tag, or a stale selector** → the signature can be forged, appended to, or replayed. Go to [DKIM](dkim.md).
- **All three locked down** → fall back to tricks that need no record gap (display-name spoofing, lookalike domains, header mismatch). Go to [Sender spoofing](sender-spoofing.md).

Once the gap is identified, [Sender spoofing](sender-spoofing.md) is where you actually craft and send the message with `swaks`.

## Pages

- **[SPF](spf.md)**: the `v=spf1` record, its mechanisms and qualifiers, and the misconfigurations (`+all`, soft `~all`, over-broad `include:`, lookup-limit permerror) that authorize your mail.
- **[DKIM](dkim.md)**: the `DKIM-Signature` header and published key, and forging, appending to (`l=`), or replaying a valid signature.
- **[DMARC](dmarc.md)**: the `_dmarc` record, alignment, and the policy gaps (`p=none`, `pct`, missing `sp`, relaxed alignment, cousin domains) that let a `From:` spoof through.
- **[Sender spoofing](sender-spoofing.md)**: the payoff page, crafting and sending the spoofed message, plus header and lookalike tricks that work even against a hardened domain.

## References

- [RFC 7489 (DMARC)](https://datatracker.ietf.org/doc/html/rfc7489)
- [RFC 7208 (SPF)](https://datatracker.ietf.org/doc/html/rfc7208)
- [RFC 6376 (DKIM)](https://datatracker.ietf.org/doc/html/rfc6376)
- [HackTricks: phishing methodology](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/phishing-methodology/index.html)
