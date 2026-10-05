---
title: "Services and operator abuse: taking nicks, channels, and IRC operator status"
description: "IRC services (NickServ, ChanServ) guard account and channel ownership with a password, so identifying or brute-forcing them takes over registered nicks and channels. The OPER command promotes a session to IRC operator, unlocking server commands (KILL, SAMODE, REHASH, DIE, RESTART) and, on some ircds, module loading and host command execution. Weak O:lines and SASL make both reachable."
keywords:
  - NickServ
  - ChanServ
  - OPER
  - IRC operator
  - SASL
---

# Services and operator abuse

IRC splits privilege into two systems you can attack separately. Services (NickServ and ChanServ, backed by Anope or Atheme) own nick and channel registration and gate it with a user-chosen password, so guessing or capturing that password hands you the account or the channel. Operator status is server-level: the `OPER <name> <password>` command checks your session against an O:line (an operator block in the ircd config that pairs a name and password hash with a host mask and a privilege set) and, on success, grants network-control commands and on many daemons the ability to load modules or run commands on the host. Both are password problems, and both are frequently weak.

## Preconditions

A registered session (numeric `001`). For services abuse, NickServ/ChanServ must be running (confirmed in [enumeration](enumeration.md) via a `services.` node in `MAP`). For operator escalation, you need a candidate operator name and password; `STATS o` where readable names the O:lines and their host masks, and operator passwords are often reused from channel topics, service accounts, or config leaks.

## Identifying and taking service accounts

```text
PRIVMSG NickServ :IDENTIFY admin Summer2026!
-NickServ- Password incorrect.
PRIVMSG NickServ :IDENTIFY admin CorrectHorse
-NickServ- You are now identified for admin.
PRIVMSG ChanServ :SET #ops FOUNDER recon
```

A successful `IDENTIFY` binds the session to that registered nick; once identified as a channel founder you can reassign the founder, grant yourself operator in the channel, or drop other users. SASL does the same at connect time and is scriptable for brute force against the plaintext port:

```bash
# brute-force a NickServ account over SASL PLAIN during the handshake
# authzid\0authcid\0password, base64-encoded, sent in an AUTHENTICATE line
printf '\0admin\0CorrectHorse' | base64
# => AGFkbWluAENvcnJlY3RIb3JzZQ==
# in the handshake: CAP REQ :sasl / AUTHENTICATE PLAIN / AUTHENTICATE <base64>
# numeric 903 = SASL authentication successful; 904 = failed
```

Interpret `903` as a valid credential (the nick is yours on every reconnect) and `904` as a wrong password; iterate the base64 line for a wordlist.

## Escalating to IRC operator

```text
OPER netadmin Autumn2026!
:irc.target.lan 464 recon :Password incorrect
OPER netadmin Sup3rSecret
:irc.target.lan 381 recon :You are now an IRC operator
```

Numeric `381` is the whole game: the session is now an operator. `464` is a wrong password (or a host mask that does not match the O:line, in which case no password will work from your address). Once at `381`, the operator command set is available:

```text
MODE recon +o
SAMODE #ops +o recon          # force channel-op in any channel (InspIRCd/UnrealIRCd)
KILL jdoe :cleanup            # forcibly disconnect a user to seize their nick
REHASH                        # reload server config from disk
```

`SAMODE` (or `SAJOIN`/`SANICK` on daemons that ship them) lets an operator seize control of any channel regardless of ChanServ, and `KILL` disconnects a target so you can grab their nick before they reconnect. On UnrealIRCd and InspIRCd an operator with the right privileges can load a server module, and some builds expose a config path or command block that runs on the host, turning operator status into execution on the ircd machine. `DIE` and `RESTART` stop or restart the whole server.

## Follow-on

- Service or founder takeover gives durable control of high-value channels (`#ops`, bot-control channels) and the ability to impersonate trusted users to the rest of the network.
- Operator status (`381`) is network control: seize any channel with `SAMODE`, deanonymize every user, and on module-capable daemons pivot to [server exploitation](server-exploitation.md) and a shell as the ircd user.
- A captured operator or service password is frequently reused on the host's shell, the services database, or other infrastructure; test it broadly.

## References

- [RFC 2812 (IRC Client Protocol, OPER)](https://datatracker.ietf.org/doc/html/rfc2812#section-3.1.4)
- [IRCv3 SASL authentication extension](https://ircv3.net/specs/extensions/sasl-3.1)
- [Atheme IRC services](https://github.com/atheme/atheme)
