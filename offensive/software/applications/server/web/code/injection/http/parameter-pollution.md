---
title: "HTTP parameter pollution (HPP)"
description: "Supplying duplicate HTTP parameters that different components resolve inconsistently, to bypass validation and WAFs or override values after a check."
keywords:
  - HTTP parameter pollution
  - HPP
  - duplicate parameters
  - WAF bypass
  - parameter precedence
---

# Parameter pollution

HTTP parameter pollution (HPP) supplies the same parameter more than once (`?id=1&id=2`) and exploits the fact that platforms resolve duplicates differently. When two components in the stack pick different occurrences, a value that passes one check is the one that is not used downstream.

Resolution varies by platform: PHP/Apache and many frameworks take the **last** occurrence, others take the **first**, ASP.NET/IIS (and classic ASP) **concatenate** them with a comma (`1,2`), and JSP/servlets expose them as an **array** where code often reads index 0. Knowing the target's rule decides which copy to weaponize.

Two abuses follow. A WAF or input filter that inspects the first occurrence can be bypassed when the application uses the last, so a benign first value hides a malicious second (useful for slipping an injection payload past a filter):

```
/search?q=safe&q=' OR '1'='1
```

Logic bypass overrides a value after it has been validated: supply the allowed value where the validator reads it and the attacker value where the business logic reads it. Query-string versus body duplication is a related case, since many frameworks merge the two scopes and one may win over the other (`?role=user` in the URL and `role=admin` in the body).

HPP is a precedence bug, not a payload by itself, so it is usually combined with another technique (SQL injection, access control) whose payload rides the occurrence the vulnerable component reads. Confirm the target's duplicate-resolution rule first by observing which value takes effect.

## Tools

- **Burp Suite**: duplicate parameters across query and body and observe which occurrence takes effect.
- **Param Miner**: Burp extension for discovering hidden and duplicated parameters.
- **curl**: send repeated parameters and cross-scope duplicates from the command line.

## References

- OWASP Testing Guide: Testing for HTTP Parameter Pollution
- PortSwigger Web Security Academy: parameter pollution notes
