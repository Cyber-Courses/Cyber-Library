---
title: "Export injection"
description: "Attacker-controlled data stored by a web app is later interpreted by a downstream program that opens the exported file, moving code execution off the web server."
keywords:
  - export injection
  - csv injection
  - formula injection
  - latex injection
  - html to pdf
---

# Exports

Export injection flips the direction of the attack: the payload is stored as ordinary data, and the harm happens when a **downstream program** opens the file the application exports.

An attacker writes a value into a field (name, note, address) that the application later emits into a generated artifact. The exported file is then parsed by a second interpreter that treats part of the data as instructions: a spreadsheet evaluating a cell that begins with `=`, a TeX engine honoring a macro, or an HTML-to-PDF renderer fetching and executing injected markup. The web application may be perfectly safe; the execution lands in the victim's spreadsheet client, in the server-side PDF toolchain, or against internal endpoints the renderer can reach. The three pages cover CSV/spreadsheet formulas, LaTeX pipelines, and HTML-to-PDF renderers.

## Pages

- **[CSV](csv.md)**: Exported cells beginning with =, +, -, or @ are evaluated as formulas by spreadsheet software, enabling command execution via DDE and data exfiltration via w...
- **[LaTeX](latex.md)**: Attacker-controlled fields compiled by a server-side TeX engine allow file reads via \\input and shell command execution via \\write18 when shell-escape is e...
- **[PDF](pdf.md)**: Injected HTML and JavaScript in server-side PDF renderers read local files via file://, reach internal and cloud-metadata endpoints through SSRF, and exfiltr...

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for submitting payloads into exported fields.
- **LibreOffice / office app**: open generated files to verify downstream execution.

## References

- [OWASP: CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [PayloadsAllTheThings: CSV Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSV%20Injection)
