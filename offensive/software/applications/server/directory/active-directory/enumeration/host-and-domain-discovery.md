---
title: "Host and domain discovery: finding domain controllers and naming contexts"
description: "Discovering the Active Directory domain and forest, locating domain controllers, and reading the LDAP rootDSE and naming contexts that anchor every later query."
keywords:
  - domain controller discovery
  - rootDSE
  - naming context
  - DNS SRV
  - nltest
---

# Host and domain discovery

Every later query needs three things: the domain name, a domain controller to talk to, and the LDAP naming contexts that root the directory tree. Discovering them is the first step whether you start from an unauthenticated network position or a compromised host.

## Finding domain controllers

Domain controllers advertise themselves through DNS SRV records and the usual Windows service ports:

```bash
# DNS SRV records point straight at the DCs and the domain name
dig +short _ldap._tcp.dc._msdcs.example.local SRV
nslookup -type=SRV _ldap._tcp.dc._msdcs.example.local

# From a domain-joined host
nltest /dsgetdc:example.local
nltest /dclist:example.local
```

On the network, a DC typically exposes 53 (DNS), 88 (Kerberos), 389/636 (LDAP/LDAPS), 445 (SMB), 464 (kpasswd), and 3268/3269 (Global Catalog). Kerberos on 88 plus LDAP on 389 is a strong DC fingerprint:

```bash
nmap -p 53,88,135,139,389,445,464,636,3268,3269 -sV <subnet>
```

## Reading the rootDSE and naming contexts

The LDAP **rootDSE** is readable anonymously on most DCs and reveals the directory's structure without credentials:

```bash
ldapsearch -x -H ldap://<dc-ip> -s base -b "" \
  defaultNamingContext rootDomainNamingContext configurationNamingContext \
  schemaNamingContext dnsHostName currentTime supportedSASLMechanisms
```

Key values:

- **`defaultNamingContext`** (for example `DC=example,DC=local`) is the search base for every object query.
- **`rootDomainNamingContext`** identifies the forest root, which matters for cross-domain and cross-forest work.
- **`configurationNamingContext`** and **`schemaNamingContext`** hold the forest-wide configuration (sites, services, the AD CS enrollment services) and the schema.
- **`dnsHostName`** confirms the DC's name, and **`supportedSASLMechanisms`** lists the SASL authentication mechanisms the DC offers (it does not reveal signing or channel-binding posture, which needs a separate policy or behavioral check).

## Identifying the domain from a foothold

On a compromised Windows host, the domain and DC come from the environment:

```
echo %USERDNSDOMAIN%
echo %LOGONSERVER%
nltest /domain_trusts
systeminfo | findstr /i domain
```

From Linux tooling against the network, SMB fingerprinting returns the domain and DC name at once:

```bash
nxc smb <dc-ip>          # NetExec: shows domain, hostname, OS, signing
enum4linux-ng -A <dc-ip>
```

## Exploitation notes

- The rootDSE read usually works unauthenticated, so domain discovery often precedes having any credentials.
- A mismatch between `defaultNamingContext` and `rootDomainNamingContext` means you are in a child domain, which changes trust and escalation options (see [trust enumeration](trust-enumeration.md)).
- Record the DC that answers and whether LDAPS is available, since signing and channel-binding posture decides whether relay to LDAP is viable later.

## Tools

- **NetExec (nxc)**: SMB/LDAP fingerprinting of domain, host, and signing.
- **ldapsearch**: rootDSE and naming-context reads.
- **nltest / nslookup / dig**: DC and SRV-record discovery.

## References

- Microsoft: LDAP rootDSE and naming contexts
- The Hacker Recipes: Active Directory reconnaissance
