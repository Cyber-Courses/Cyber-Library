---
title: "Ivanti Connect Secure: auth-bypass and command-injection chains"
order: 2
description: "Ivanti Connect Secure (formerly Pulse Connect Secure) has had pre-authentication chains combining an authentication-bypass flaw with a command-injection flaw to reach unauthenticated remote code execution on the gateway. Exploited at scale, these give control of the appliance and a route into the internal network it fronts."
keywords:
  - ivanti
  - pulse connect secure
  - connect secure
  - auth bypass
  - command injection
---

# Ivanti Connect Secure

Ivanti Connect Secure (previously Pulse Connect Secure) is a widely deployed remote-access gateway, and it has been exploited through pre-authentication chains that combine two flaws: an authentication-bypass that reaches endpoints or functionality meant to require login, and a command-injection that executes OS commands on the appliance. Chained, they give unauthenticated remote code execution on the gateway. These have been exploited at scale, including by state actors, because the appliance is internet-facing and holds the route inside; the appliance's own integrity checker has also been targeted to evade detection after compromise.

```bash
# fingerprint Ivanti/Pulse Connect Secure
curl -skI https://<gateway>/
curl -sk 'https://<gateway>/dana-na/' | grep -iE 'ivanti|pulse'
# the chain: reach a protected endpoint via the auth-bypass, then invoke the
# command-injection sink to run OS commands on the appliance (unauthenticated RCE).
# match the exact Connect Secure build to the advisory for the specific chain.
```

## Exploitation notes

- The power is the chain: the auth-bypass alone reaches restricted functionality, and feeding the command-injection through it gives unauthenticated code execution, so identify both components for the target build.
- Post-compromise, attackers have tampered with the appliance's Integrity Checker Tool to hide, so the device cannot be trusted to report its own compromise; this matters for persistence and detection evasion considerations.
- The outcome is appliance control plus the internal-network pivot and any cached/returned credentials; Connect Secure compromise has driven major intrusions.
- Version-specific; fingerprint the Connect Secure build (the `dana-na` portal paths and resources) and match the advisory for the exact chain.

## References

- [Ivanti security advisories](https://www.ivanti.com/blog)
- [CISA: Ivanti Connect Secure exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
