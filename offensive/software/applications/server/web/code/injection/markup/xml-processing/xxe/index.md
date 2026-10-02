---
title: "XML external entity injection"
description: "XML parsers that resolve DTDs and external entities let an attacker read local files, reach internal services, and amplify input into denial of service."
keywords:
  - XXE
  - XML external entity
  - DTD injection
  - entity expansion
  - out-of-band XXE
---

# XXE

XML external entity injection abuses the parts of the XML specification that most applications never need: the **Document Type Definition** (DTD) and its **entities**. When a parser accepts attacker-controlled XML with entity resolution left enabled, a declaration such as `<!ENTITY xxe SYSTEM "file:///etc/passwd">` makes the parser fetch an external resource and splice its contents into the document wherever `&xxe;` appears. The input stays valid XML, so it passes through SOAP endpoints, REST bodies, SVG and DOCX uploads, SAML, and any other XML-backed interface.

Three outcomes follow from that single primitive. **Disclosure** reads local files and returns them in the response or through a side channel. **SSRF and out-of-band** retrieval point the entity at internal hosts, cloud metadata, or an attacker server, reaching resources the parser's network can see. **Amplification** nests entities so a tiny document expands into gigabytes, exhausting memory for denial of service.

This subtree covers in-band file disclosure, out-of-band exfiltration, SSRF, entity-expansion attacks, and the blind variants that leak data through parser errors when nothing is reflected.

## Tools

- **XXEinjector**: automating XXE exploitation including OOB and error-based file retrieval.
- **Burp Suite**: crafting XXE payloads and scanning parameters.
- **Burp Collaborator**: confirming blind and out-of-band XXE.
- **oxml_xxe**: embedding XXE payloads into OOXML and other file uploads.

## References

- PortSwigger Web Security Academy: XML external entity (XXE) injection
- OWASP: XML External Entity Prevention Cheat Sheet
