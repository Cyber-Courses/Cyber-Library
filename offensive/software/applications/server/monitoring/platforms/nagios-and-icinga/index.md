---
title: "Nagios and Icinga: attacking the monitoring servers"
description: "Nagios (Core and the commercial XI) and the Icinga fork monitor hosts through plugins and the NRPE remote executor. The surface is the web interface authentication, the NRPE agent on 5666 which runs check plugins and is prone to argument and command injection, injection into Nagios command and macro definitions, and the recurring Nagios XI authentication-bypass-to-RCE chains."
keywords:
  - nagios
  - nagios xi
  - icinga
  - nrpe
  - monitoring
---

# Nagios and Icinga

Nagios is a long-standing monitoring system in two forms: the open-source Nagios Core (configured by files, with a CGI web interface) and the commercial Nagios XI (a full PHP web application on top). Icinga is a fork with Icinga 2 and the Icinga Web 2 interface. They monitor hosts by running check plugins locally and, for remote hosts, through NRPE (the Nagios Remote Plugin Executor) on TCP 5666. The attack surface: the web interface authentication, the NRPE agent which executes check commands and is a classic command-injection target, injection into the Nagios command and macro definitions that run on the monitoring server, and the long list of Nagios XI web-application vulnerabilities that chain authentication bypass to remote code execution.

```bash
# web interfaces and the NRPE agent
curl -sk https://<target>/nagios/ ; curl -sk https://<target>/nagiosxi/
nmap -p5666 -sV <target>                         # NRPE
```

## Subtopics

- **[Enumeration](enumeration.md)**: product and version fingerprinting.
- **[Authentication](authentication.md)**: default and weak web credentials.
- **[NRPE abuse](nrpe-abuse.md)**: the remote plugin executor on 5666.
- **[Command and plugin injection](command-and-plugin-injection.md)**: injection into command definitions.
- **[Nagios XI exploits](nagios-xi-exploits.md)**: the auth-bypass-to-RCE chains.

## References

- [Nagios documentation](https://www.nagios.org/documentation/)
- [NRPE documentation](https://github.com/NagiosEnterprises/nrpe)
- [HackTricks: 5666 NRPE](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
