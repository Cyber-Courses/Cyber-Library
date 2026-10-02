---
title: "AJP and Ghostcat: file read and inclusion through the Tomcat AJP connector"
description: "Exploiting an exposed Tomcat AJP connector (Ghostcat): reading or including files under the web application via crafted AJP attributes, disclosing source and configuration, and escalating to RCE with an uploadable file."
keywords:
  - ghostcat
  - AJP connector
  - tomcat 8009
  - file read
  - LFI to RCE
---

# AJP and Ghostcat

Tomcat ships an **AJP connector** (Apache JServ Protocol), classically on port `8009`, intended for a front Apache/nginx to forward requests over a trusted link. AJP lets the caller set request attributes that Tomcat normally trusts, including the servlet that will process the request and the file it maps to. When the AJP port is reachable, an attacker sets those attributes directly. This is the Ghostcat class of bug.

## File read and inclusion

By crafting AJP attributes (`javax.servlet.include.request_uri`, `...servlet_path`, `...path_info`), an attacker makes Tomcat's default servlet read an arbitrary file under the web application root and return it, or *include* it as if it were a servlet resource:

- **Read**: disclose any file under the web app, including `WEB-INF/web.xml` (the deployment descriptor, servlet map, and often credentials) and compiled/class and config files normally protected under `WEB-INF`.
- **Include**: process a file as JSP. If the attacker can get a file with JSP content onto the server (an upload stored under the web root, even as a `.txt`/image), including it executes the JSP, giving **RCE**.

```bash
# Metasploit: auxiliary/admin/http/tomcat_ghostcat  (file read)
# or the standalone PoC:
python3 ghostcat.py -p 8009 -f WEB-INF/web.xml target
```

## Finding it

- Port-scan for `8009/tcp` (AJP). An open AJP connector that is not firewalled to the front proxy is the precondition.
- Confirm by reading `WEB-INF/web.xml`; a successful read proves the file-read primitive and maps the app's servlets and any embedded secrets.

## Exploitation

- Read `WEB-INF/web.xml`, then the referenced config and class files, to recover credentials (DB, management app), servlet mappings, and secrets.
- For RCE, find any feature that stores attacker content under the web root (upload, avatar, log, cache), then use the AJP *include* to execute it as JSP.
- Credentials recovered from `web.xml`/config frequently unlock the [Manager app](manager-and-host-manager.md) for a cleaner WAR-deploy RCE.

## Tools

- **ghostcat** PoC, Metasploit `tomcat_ghostcat`, **nmap** AJP scripts.

## References

- Apache Tomcat: AJP connector documentation and security guidance
- Ghostcat technical writeups
