---
title: ".NET deserialization: BinaryFormatter, TypeNameHandling, and ViewState"
order: 5
description: "Exploiting .NET deserialization: BinaryFormatter/LosFormatter sinks, Json.NET TypeNameHandling, ASP.NET ViewState, and gadget generation with ysoserial.net."
keywords:
  - .NET deserialization
  - BinaryFormatter
  - TypeNameHandling
  - ViewState
  - ysoserial.net
---

# .NET deserialization

.NET deserializers that preserve type information can be steered to instantiate attacker-chosen types and run their reconstruction logic, giving gadget-chain **remote code execution**. The classic unsafe formatters embed types in the stream; the JSON libraries become unsafe when configured to honor a type hint.

## The sinks

- **`BinaryFormatter.Deserialize`**, **`LosFormatter`**, **`ObjectStateFormatter`**, **`SoapFormatter`**, **`NetDataContractSerializer`**: type-aware binary/XML formatters that are unsafe on untrusted input. Base64 BinaryFormatter output often begins `AAEAAAD/////`.
- **Json.NET (`Newtonsoft.Json`) with `TypeNameHandling`** set to anything other than `None`: the JSON carries a `$type` field naming the CLR type to instantiate, which is the injection point:

  ```json
  { "$type": "System.Windows.Data.ObjectDataProvider, PresentationFramework",
    "MethodName": "Start", "ObjectInstance": { "$type": "System.Diagnostics.Process, System", ... } }
  ```

- **`JavaScriptSerializer`** with a custom `SimpleTypeResolver`, and **`XmlSerializer`/`DataContractSerializer`** when the expected type is attacker-influenced.

## ViewState

ASP.NET **`__VIEWSTATE`** is a `LosFormatter`/`ObjectStateFormatter` blob. When it is not protected by a valid MAC (MAC disabled, or the `machineKey` is known or leaked), an attacker forges a ViewState that deserializes to a gadget. `ysoserial.net` has a dedicated ViewState plugin that takes the key and generator:

```
ysoserial.exe -p ViewState -g TextFormattingRunProperties \
  -c "cmd /c ping attacker" --apppath="/" --path="/page.aspx" \
  --decryptionalg="AES" --decryptionkey=... --validationalg="SHA1" --validationkey=...
```

A leaked or default `machineKey` (from config disclosure) turns ViewState into reliable RCE.

## Gadget generation

Use **ysoserial.net** to produce payloads for the formatter and gadget in play:

```
ysoserial.exe -f BinaryFormatter -g TypeConfuseDelegate -c "calc"
ysoserial.exe -f Json.Net -g ObjectDataProvider -c "cmd /c whoami"
```

Deliver in the formatter's expected encoding (base64 for most binary formatters).

## Exploitation notes

- Identify the formatter first; the gadget (`TypeConfuseDelegate`, `ObjectDataProvider`, `WindowsIdentity`, `TextFormattingRunProperties`) must match both the formatter and the assemblies available (WPF `PresentationFramework`, etc.).
- For Json.NET, you need `TypeNameHandling != None` on the deserializing side; probe by sending a `$type` and watching for type-load behavior or errors.
- ViewState requires defeating the MAC: hunt for a disclosed `web.config`/`machineKey`, a known default, or MAC-disabled pages.

## Tools

- **ysoserial.net**: formatters, gadgets, and the ViewState plugin.

## References

- PortSwigger Web Security Academy: Insecure deserialization
- ysoserial.net project
