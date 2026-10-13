---
title: "Webmail: attacking the web front end over an IMAP and SMTP backend"
order: 3
description: "Webmail is an HTTP application sitting over an IMAP/SMTP mail store, so it is attacked as a web app (stored XSS in message rendering, SSRF, auth bypass, file write to RCE) and as a reachable mail backend behind it. Fingerprint the product from the login page, then navigate to Roundcube, Horde, or Zimbra."
keywords:
  - webmail
  - roundcube
  - horde
  - zimbra
  - stored xss webmail
---

# Webmail

A webmail product is a web application that renders mail from an IMAP store and sends through an SMTP relay. Everything that makes it useful is also the attack surface: it parses and displays attacker-supplied HTML mail, it fetches remote content, it stores per-user preferences and plugins on disk, and it holds long-lived session cookies to mailboxes. So a webmail host is attacked on two planes at once. The **web plane** is the PHP or Java application itself: stored cross-site scripting in message rendering that fires when the victim opens your mail, server-side request forgery from remote-image or calendar fetches, authentication and token-handling flaws, and file writes under the webroot that become code execution. The **mail plane** is the IMAP/SMTP backend the front end proxies to, reachable once you have a session or a backend credential.

The decisive property is that you deliver the payload by email. Unlike a normal web app where you need the victim to visit a link, a stored XSS in message rendering runs the moment the target reads a message you sent them, with their session, inside their mailbox origin. That turns a single crafted message into session theft, silent send-as, and mailbox exfiltration.

## Fingerprint which webmail

Identify the product and build before anything else: the exploitation path is entirely product-specific, and the version decides which message-rendering bypass or file-write chain applies.

```bash
# Login-page and path fingerprints
curl -sk https://<target>/ -i | grep -iE 'server:|x-powered-by|set-cookie'
curl -sk https://<target>/?_task=login -i        # Roundcube: redirects/renders its login
curl -sk https://<target>/horde/ -i              # Horde: /horde/ application root
curl -sk https://<target>/zimbra/ -i             # Zimbra web client
curl -sk https://<target>:7071/zimbraAdmin/ -i   # Zimbra admin console (separate port)
```

Read the signals:

- **Roundcube**: the login URL carries `?_task=login`, the session cookie is `roundcube_sessid` (and `roundcube_sessauth`), and the markup references `program/js/` and `skins/`. The build is in `CHANGELOG.md` at the webroot on many installs, or in the `?s=<hash>` asset query.
- **Horde**: an application root at `/horde/`, cookie `Horde`, login form posting to `/horde/login.php`, and the webmail module branded **IMP**. A `<meta name="generator">` or the `/horde/services/` tree confirms it.
- **Zimbra**: the client under `/zimbra/`, the admin console on **7071** at `/zimbraAdmin/`, the SOAP endpoints `/service/soap` and `/service/admin/soap`, cookies `ZM_AUTH_TOKEN` and `ZM_ADMIN_AUTH_TOKEN`. Build strings appear in `/zimbra/js/` asset paths.
- **Favicon and generator**: hash the favicon (`curl -sk https://<target>/favicon.ico | md5sum`) against a known-product table, and grep the HTML for `<meta name="generator">`, which Roundcube and Horde both emit.

Once the product and build are known, go to the matching page and start from its own fingerprint section.

## Pages

- **[Roundcube](roundcube.md)**: stored XSS in HTML message rendering (`rcube_washtml` bypasses), skin and plugin path traversal, and the file-write-to-PHP chains.
- **[Horde](horde.md)**: CSRF against admin and preference endpoints, preference and form injection, and the image-handler command injection to RCE.

## Subtopics

- **[Zimbra](zimbra/index.md)**: the full Collaboration Suite, from account enumeration and admin-token abuse to the `mboximport` web-shell write and the ProxyServlet SSRF chain to admin.

## References

- [HackTricks: pentesting web (webmail interfaces)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
- [PortSwigger Web Security Academy: cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
- [SonarSource research blog (webmail vulnerability analyses)](https://www.sonarsource.com/blog/)
