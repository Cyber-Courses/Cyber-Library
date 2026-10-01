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

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

An attacker writes a value into a field (name, note, address) that the application later emits into a generated artifact. The exported file is then parsed by a second interpreter that treats part of the data as instructions: a spreadsheet evaluating a cell that begins with `=`, a TeX engine honoring a macro, or an HTML-to-PDF renderer fetching and executing injected markup. The web application may be perfectly safe; the execution lands in the victim's spreadsheet client, in the server-side PDF toolchain, or against internal endpoints the renderer can reach. The three pages cover CSV/spreadsheet formulas, LaTeX pipelines, and HTML-to-PDF renderers.
