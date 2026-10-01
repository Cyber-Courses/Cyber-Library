---
title: "Local DTD reuse for blind XXE with no egress"
description: "Repurposing a DTD file already on disk lets a blind XXE redefine an internal entity and trigger an error-based leak when outbound network is blocked."
keywords:
  - local DTD
  - blind XXE
  - no outbound network
  - entity redefinition
  - error-based leak
---

# Local DTD

The portable blind techniques fetch an external DTD from an attacker server. When outbound network access is blocked entirely, no HTTP or FTP, and the internal subset forbids the parameter-entity-in-markup-declaration trick the attacker needs, those approaches stall. The **local DTD** technique removes the network requirement by reusing a DTD file that already exists on the target's own filesystem.

## The idea

Most systems ship DTD files as part of installed packages: documentation toolchains, GNOME, Linux distributions, and application runtimes all drop `.dtd` files on disk. A well-known example is the DocBook DTD at `/usr/share/yelp/dtd/docbookx.dtd`, present on many GNOME installations. These files declare parameter entities internally. The attack loads one of them with a `SYSTEM "file://..."` reference, which is a purely local read, and then **redefines** one of the parameter entities it declares.

XML allows a parameter entity to be redefined, and the first definition wins; by declaring the target entity before invoking the local DTD, the attacker's value takes effect while the rest of the local DTD still parses normally. Pick a parameter entity that the local DTD references somewhere, and hijack it to carry the error-based leak.

## Example payload

`docbookx.dtd` internally declares, among others, a parameter entity used in its content model. The injected document loads the file and redefines that entity so that resolving it reads the target file and feeds it into an invalid path:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
  <!ENTITY % ISOamso '
    <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM
      &#x27;file:///nonexistent/&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;error;
  '>
  %local_dtd;
]>
<foo>bar</foo>
```

The flow: `%local_dtd;` pulls in the on-disk DocBook DTD, which internally references `%ISOamso;`. Because the attacker declared `%ISOamso;` first, the redefined version runs. Inside it, `%file;` reads `/etc/passwd`, `%eval;` builds `%error;` as a reference to a nonexistent path with the file contents appended, and resolving `%error;` throws a parse error whose message carries the file. The numeric references (`&#x25;` for `%`, `&#x26;#x25;` for an escaped `%`, `&#x27;` for a quote) keep the nested declarations well-formed so each layer is defined at the right moment.

## Finding a usable DTD

The technique needs a DTD that is present on disk and whose internal parameter entities can be redefined into the leak chain. Candidates vary by platform; probe for ones likely to exist:

```
file:///usr/share/yelp/dtd/docbookx.dtd        # GNOME / yelp, entity %ISOamso;
file:///usr/share/xml/fontconfig/fonts.dtd     # fontconfig
file:///usr/share/xml/scrollkeeper/dtds/scrollkeeper-omf.dtd
```

On Windows and Java stacks, runtime and application directories hold their own DTDs. Confirm a file exists first with a plain local read that succeeds or fails distinguishably, then match the redefined entity name to one the chosen DTD actually declares and references. Once a present DTD and a valid entity name are found, the leak proceeds with no outbound connection at all.

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
