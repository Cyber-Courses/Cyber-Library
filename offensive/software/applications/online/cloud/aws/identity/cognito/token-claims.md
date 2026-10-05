---
title: "Token claims: abusing Cognito ID and access token claims"
description: "Tampering with or abusing Cognito ID and access token claims to assume a more privileged identity."
keywords:
  - Cognito
  - token claims
  - ID token
  - access token
  - JWT
---

# Token claims

Cognito issues JWT ID and access tokens whose claims (groups, `custom:` attributes, scopes) applications and identity pools consume for authorization. Where an app trusts a claim the user can influence, or validates tokens weakly, manipulating claims elevates the session.

## Where claims are trusted

- **Identity-pool role mapping**: a pool can map users to roles by a token claim (a group or attribute), so controlling that claim selects a more privileged role at `get-credentials-for-identity`.
- **App-side authorization**: an app that reads `cognito:groups` or a `custom:` claim and trusts it grants whatever the claim says.

```bash
# decode a held token to see the claims in play
echo "<jwt>" | cut -d. -f2 | base64 -d 2>/dev/null
```

## Exploitation notes

- Group membership drives `cognito:groups`; if you can join a group (admin-set or self-service), the next token carries it.
- Role-mapping rules that key on a writable attribute combine directly with [user pool](user-pool.md) attribute abuse.
- Weak signature or audience validation on the app side lets a crafted token through; test whether the app verifies the Cognito JWKS and `aud`.

## Tools

- **jwt_tool**: inspect and test the token's claims and validation.
- **AWS CLI** (`cognito-identity get-credentials-for-identity`) to turn a mapped token into role credentials.

## References

- [HackTricks Cloud: AWS Cognito](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [AWS: Cognito identity-pool role mapping](https://docs.aws.amazon.com/cognito/latest/developerguide/role-based-access-control.html)
- [cognito-scanner](https://pypi.org/project/cognito-scanner/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
