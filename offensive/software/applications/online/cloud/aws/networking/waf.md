---
title: "WAF: bypassing web ACL rules to reach the application"
order: 10
description: "Bypassing AWS WAF rules through encoding, size limits, and rule gaps to reach the protected application."
keywords:
  - WAF
  - web ACL
  - rule
  - bypass
  - encoding
---

# WAF

AWS WAF applies a web ACL of managed and custom rules to CloudFront, API Gateway, or an ALB. Reading the web ACL shows which rules run and their request-size and match limits, and the attack is to shape a payload that slips through a gap: oversized bodies WAF stops inspecting, encodings the rule does not normalize, or a request path that reaches the origin without passing the ACL at all.

## Reading the web ACL

```bash
aws wafv2 list-web-acls --scope REGIONAL   # or CLOUDFRONT
aws wafv2 get-web-acl --name <name> --scope REGIONAL --id <id> \
  --query "WebACL.Rules[].[Name,Statement,Action]"
```

## Bypass levers

```
# oversized body: WAF inspects only the first N KB, so push the payload past the limit
# encoding/normalization gaps the managed rule misses
id=1%2f%2a%2a%2funion%2f%2a%2a%2fselect   # comment/whitespace obfuscation
# header or method the rule does not cover (e.g. only GET/POST inspected)
```

## Exploitation notes

- WAF enforced only at CloudFront is void if the [origin](cloudfront.md) accepts direct requests; the origin address is the cleanest bypass.
- The body-inspection size limit is a hard boundary: content past it is unfiltered, which suits large injection or upload payloads.
- Rate-based and IP-reputation rules are evaded with distributed source IPs; inspect the web ACL to see which rules are reputation-based.

## Tools

- **AWS CLI** (`wafv2 get-web-acl`): rule disclosure where you have read access.
- **Burp Suite** / **ffuf**: iterate encodings and sizes against the live ACL.
- **nuclei**: fingerprint the WAF and known bypasses.

## References

- [HackTricks: WAF bypass](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/waf-bypass.html)
- [AWS: WAF web ACL rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rules.html)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
- [CloudFox: finding origins exposed behind the edge](https://github.com/BishopFox/cloudfox)
