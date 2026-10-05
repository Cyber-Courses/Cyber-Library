---
title: "Enumeration: reading an IRC network from a registered session"
description: "Once a NICK/USER handshake completes, a normal IRC session can read the entire network: the ircd and version via VERSION, user and channel counts via LUSERS, the channel list via LIST, user details via WHO and WHOIS, channel membership via NAMES, server internals via STATS, and the network topology via MAP, plus service state through NickServ and ChanServ queries."
keywords:
  - IRC enumeration
  - LIST
  - WHOIS
  - VERSION
  - ircd version
---

# Enumeration

A registered IRC session is a full read interface to the network. There is no separate authentication for most queries: after the `NICK`/`USER` handshake the server answers numeric replies to a handful of commands that disclose the ircd build, every channel and its topic, every connected user and their host, the server links, and the services in use. The ircd version from `VERSION` is the single most important datum, because it decides whether a known server flaw applies, and the channel and user listing maps the human targets and any bots.

## Preconditions

A completed handshake (numeric `001`). If the server sent `464`, supply the connection password first with `PASS <pass>` before `NICK`/`USER`. Some networks hide `LIST` output or user hosts until you have joined a channel or set a mode, and services only answer if NickServ/ChanServ are actually running.

## Worked session

```text
$ nc irc.target.lan 6667
NICK recon
USER recon 0 * :recon
:irc.target.lan 001 recon :Welcome to the TargetNet IRC Network recon
:irc.target.lan 004 irc.target.lan UnrealIRCd-6.1.0 iowrsxzdHtIRcaAqOWS ...
VERSION
:irc.target.lan 351 recon UnrealIRCd-6.1.0. irc.target.lan :build details
LUSERS
:irc.target.lan 251 recon :There are 42 users and 3 invisible on 2 servers
:irc.target.lan 252 recon 1 :operator(s) online
LIST
:irc.target.lan 322 recon #ops 4 :deployment keys here, do not paste outside
:irc.target.lan 322 recon #general 31 :TargetNet general chat
:irc.target.lan 323 recon :End of /LIST
WHO *
:irc.target.lan 352 recon #ops ~jdoe 10.0.0.5 irc.target.lan jdoe H :0 John Doe
NAMES #ops
:irc.target.lan 353 recon = #ops :@admin jdoe backupbot
WHOIS admin
:irc.target.lan 311 recon admin ~admin 10.0.0.2 * :Site Admin
:irc.target.lan 313 recon admin :is an IRC Operator
```

Reading the numerics: `351` gives the exact ircd build (feed straight to [server exploitation](server-exploitation.md)); `251`/`252` count users and operators; each `322` is one channel with member count and topic (topics routinely leak secrets, as `#ops` does here); `352` from `WHO` gives each user's ident and real IP (`10.0.0.5`), which deanonymizes operators and bots; `353` from `NAMES` lists channel members, where `@` marks a channel operator; `311` and `313` from `WHOIS` confirm `admin` is an IRC operator and expose its host.

Server internals and topology:

```text
MAP
:irc.target.lan 015 recon :irc.target.lan (42 clients)
:irc.target.lan 015 recon :  `- services.target.lan (0 clients)
STATS o
:irc.target.lan 243 recon O *@10.0.0.2 * admin 0 :operator block
```

`MAP` reveals linked servers (including the `services.` pseudo-server, confirming Anope/Atheme services), and `STATS o` where permitted dumps the O:lines, which show exactly which host masks and operator names can run `OPER`, naming your target for operator escalation.

Querying services directly:

```text
PRIVMSG NickServ :INFO admin
PRIVMSG ChanServ :INFO #ops
PRIVMSG ChanServ :ACCESS #ops LIST
```

`NickServ INFO` shows whether a nick is registered, when it was last seen, and sometimes the email on file; `ChanServ INFO`/`ACCESS LIST` shows channel founders and the access list, naming who to impersonate or take over.

## Exploitation notes

- Record the `004`/`351` version verbatim before anything else; it is the pivot to [server exploitation](server-exploitation.md).
- `WHO *` and `WHOIS` leak real client IPs unless the network enforces host cloaking, giving you internal addresses to pivot toward even without touching the chat.
- Channel topics and `#ops`-style channels are where operators paste keys, tokens, and bot commands; join read-only and scroll back.
- A `services.` node in `MAP` plus answers from NickServ/ChanServ tells you account takeover is in play; move to [services and operator abuse](services-and-operator-abuse.md).

## References

- [RFC 1459 (IRC, numeric replies)](https://datatracker.ietf.org/doc/html/rfc1459#section-6)
- [RFC 2812 (IRC Client Protocol)](https://datatracker.ietf.org/doc/html/rfc2812)
- [Nmap irc-info script](https://nmap.org/nsedoc/scripts/irc-info.html)
