---
title: "Ruby deserialization: Marshal and YAML universal gadget chains"
order: 4
description: "Exploiting Ruby deserialization: Marshal.load and YAML.load/Psych on untrusted input, and the universal gadget chains that reach code execution without application-specific classes."
keywords:
  - ruby deserialization
  - Marshal.load
  - YAML.load
  - Psych
  - universal gadget chain
---

# Ruby deserialization

Ruby's `Marshal.load` and, historically, `YAML.load` (Psych) reconstruct arbitrary Ruby objects from untrusted input. Reconstruction runs callbacks such as `init_with` and `marshal_load`, and the Ruby ecosystem has well-known **universal gadget chains** built from the standard library and Rails/Gem dependencies, so exploitation often does not need application-specific classes.

## The sinks

- **`Marshal.load(data)`** on attacker bytes. Marshal is a binary format beginning `\x04\x08`; it appears in caches, cookies, and session stores (Rails once defaulted cookie sessions to Marshal). Marshal is explicitly unsafe for untrusted data.
- **`YAML.load(data)`** (older Psych) builds arbitrary objects via tags like `!ruby/object:` and `!ruby/hash:`. `YAML.safe_load` (and modern defaults) restrict this; test which is in use.
- **`Oj.load`** in compat/object mode, and other object-mode JSON/MessagePack loaders, reconstruct Ruby objects the same way.

## Universal gadget chains

Because the chain can be assembled from gems present in almost every app, generic payloads exist. A classic Psych/YAML gadget uses `Gem::Requirement` with an embedded `Gem::Installer`/`Gem::Package` sequence that reaches `Kernel.system`, and equivalent Marshal chains target `ActiveSupport`/`ERB` to evaluate attacker code. The practical workflow is to recognize the sink, then apply a known universal chain for the Ruby/Rails/psych version in play rather than hand-building gadgets.

A YAML payload shape against a vulnerable loader:

```yaml
--- !ruby/object:Gem::Requirement
requirements:
  !ruby/object:Gem::Package::TarReader
  io: &1 !ruby/object:Net::BufferedIO
  ...
```

(Full chains are version-specific; use a maintained chain for the target's gem versions.)

## Exploitation notes

- Fingerprint the sink: Marshal (`\x04\x08` binary) versus YAML (text with `!ruby/...` tags) determines the payload family.
- Rails apps: check the session/cookie serializer and any `Marshal.load` on cache entries; a secret-key-signed cookie blocks tampering, so look for unsigned caches and queues instead.
- As with other engines, a successful load executes before the app inspects the object, so post-load validation does not help the defender or hinder you.

## Tools

- Maintained universal-gadget payloads for Marshal and Psych/YAML (version-matched).

## References

- Ruby docs: Marshal (security note), Psych/YAML
- PortSwigger Web Security Academy: Insecure deserialization
