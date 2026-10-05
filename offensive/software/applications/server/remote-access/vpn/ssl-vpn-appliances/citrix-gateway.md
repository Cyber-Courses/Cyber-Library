---
title: "Citrix Gateway: traversal-to-RCE and the Citrix Bleed token disclosure"
description: "Citrix Gateway and ADC (NetScaler) have had a path-traversal chain giving unauthenticated remote code execution, and the Citrix Bleed memory-disclosure flaw that leaks session tokens from the appliance, letting an attacker hijack authenticated sessions and bypass multi-factor authentication. Both were exploited at scale for initial access."
keywords:
  - citrix
  - netscaler
  - adc
  - citrix bleed
  - session token
---

# Citrix Gateway

Citrix Gateway and ADC (NetScaler) front remote access and applications, and two flaw classes made them prime targets. A path-traversal vulnerability chained to unauthenticated remote code execution on the appliance, giving device control. And Citrix Bleed is a memory-disclosure flaw: an unauthenticated request to the gateway returns uninitialised memory that contains valid session tokens, so an attacker repeatedly reads the endpoint, harvests session tokens, and replays them to hijack authenticated user sessions, bypassing both passwords and multi-factor authentication. Both were exploited at scale, Citrix Bleed notably by ransomware groups, for initial access.

```bash
# fingerprint Citrix Gateway / NetScaler ADC
curl -skI https://<gateway>/
curl -sk https://<gateway>/vpn/index.html | grep -i netscaler
# Citrix Bleed class: repeatedly request the vulnerable endpoint; the response leaks
# memory containing session tokens -> replay a token to hijack an authenticated session
for i in $(seq 1 200); do curl -sk 'https://<gateway>/<bleed-endpoint>' ; done | strings | grep -i 'sessionid\|token'
# traversal-to-RCE class: a crafted path reaches a writable/exec location -> RCE
```

## Exploitation notes

- Citrix Bleed is distinctive: it needs no credentials and defeats MFA, because the leaked session tokens are for already-authenticated sessions; replaying a harvested token drops the attacker into that user's authenticated session on the gateway.
- Harvesting is a repetition attack: each request leaks a chunk of memory, so many requests collect tokens; filter the responses for session-token patterns.
- The traversal-to-RCE class gives appliance code execution directly; identify which flaw the target build is exposed to.
- Both yield access that fronts internal applications and the network; Citrix Bleed in particular drove large ransomware intrusions via hijacked sessions.

## References

- [Citrix security bulletins](https://support.citrix.com/securitybulletins)
- [CISA: Citrix Bleed exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
