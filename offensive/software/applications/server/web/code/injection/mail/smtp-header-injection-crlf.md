---
title: "SMTP and email header injection: CRLF in headers, Bcc injection, and envelope confusion"
description: Newline injection in From, Subject, or custom headers when applications concatenate user input into RFC 5322 messages.
keywords:
  - email header injection
  - CRLF injection
---

# SMTP header injection

## Context

Unescaped `\r\n` in user fields **terminates** the current header and **starts** new headers or **body** separation, adding **Bcc**, changing **Subject**, or splitting **MIME** parts. Impact depends on MTA and whether the app builds **raw** SMTP vs API envelopes.

## Theory

Separate **envelope** recipients (SMTP `MAIL FROM` / `RCPT TO`) from **message** headers when using APIs that already isolate them.

## Practice

- Inject newline sequences into contact-form email fields in a mailhog or staging MTA and inspect the received message structure.
