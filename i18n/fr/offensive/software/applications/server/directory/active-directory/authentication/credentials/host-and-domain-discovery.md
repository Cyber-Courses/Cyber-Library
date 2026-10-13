---
order: 17
title: "Découverte des hôtes et du domaine : trouver les contrôleurs de domaine et les contextes de nommage"
description: "Découvrir le domaine et la forêt Active Directory, localiser les contrôleurs de domaine, et lire le rootDSE LDAP et les contextes de nommage qui ancrent chaque requête ultérieure."
keywords:
  - découverte de contrôleur de domaine
  - rootDSE
  - naming context
  - DNS SRV
  - nltest
---

# Découverte des hôtes et du domaine

Chaque requête ultérieure a besoin de trois choses : le nom du domaine, un contrôleur de domaine avec qui dialoguer, et les contextes de nommage LDAP qui enracinent l'arborescence de l'annuaire. Les découvrir est la première étape, que vous partiez d'une position réseau non authentifiée ou d'un hôte compromis.

## Trouver les contrôleurs de domaine

Les contrôleurs de domaine s'annoncent via des enregistrements DNS SRV et les ports de service Windows habituels :

```bash
# DNS SRV records point straight at the DCs and the domain name
dig +short _ldap._tcp.dc._msdcs.example.local SRV
nslookup -type=SRV _ldap._tcp.dc._msdcs.example.local

# From a domain-joined host
nltest /dsgetdc:example.local
nltest /dclist:example.local
```

Sur le réseau, un DC expose généralement les ports 53 (DNS), 88 (Kerberos), 389/636 (LDAP/LDAPS), 445 (SMB), 464 (kpasswd), et 3268/3269 (catalogue global). Kerberos sur 88 plus LDAP sur 389 constitue une empreinte de DC solide :

```bash
nmap -p 53,88,135,139,389,445,464,636,3268,3269 -sV <subnet>
```

## Lire le rootDSE et les contextes de nommage

Le **rootDSE** LDAP est lisible de manière anonyme sur la plupart des DC et révèle la structure de l'annuaire sans identifiants :

```bash
ldapsearch -x -H ldap://<dc-ip> -s base -b "" \
  defaultNamingContext rootDomainNamingContext configurationNamingContext \
  schemaNamingContext dnsHostName currentTime supportedSASLMechanisms
```

Valeurs clés :

- **`defaultNamingContext`** (par exemple `DC=example,DC=local`) est la base de recherche pour chaque requête d'objet.
- **`rootDomainNamingContext`** identifie la racine de la forêt, ce qui compte pour le travail inter-domaines et inter-forêts.
- **`configurationNamingContext`** et **`schemaNamingContext`** contiennent la configuration à l'échelle de la forêt (sites, services, les services d'inscription AD CS) et le schéma.
- **`dnsHostName`** confirme le nom du DC, et **`supportedSASLMechanisms`** liste les mécanismes d'authentification SASL que le DC propose (cela ne révèle pas la posture de signature ou de channel-binding, qui nécessite une vérification de politique ou comportementale séparée).

## Identifier le domaine depuis un point d'ancrage

Sur un hôte Windows compromis, le domaine et le DC proviennent de l'environnement :

```
echo %USERDNSDOMAIN%
echo %LOGONSERVER%
nltest /domain_trusts
systeminfo | findstr /i domain
```

Depuis l'outillage Linux contre le réseau, le fingerprinting SMB renvoie le domaine et le nom du DC d'un coup :

```bash
nxc smb <dc-ip>          # NetExec: shows domain, hostname, OS, signing
enum4linux-ng -A <dc-ip>
```

## Notes d'exploitation

- La lecture du rootDSE fonctionne généralement sans authentification, la découverte du domaine précède donc souvent la possession d'identifiants.
- Un décalage entre `defaultNamingContext` et `rootDomainNamingContext` signifie que vous êtes dans un domaine enfant, ce qui change les options de confiance et d'escalade (voir [énumération des approbations](../../trusts/trust-enumeration.md)).
- Notez le DC qui répond et si LDAPS est disponible, car la posture de signature et de channel-binding détermine si un relais vers LDAP sera viable plus tard.

## Outils

- **NetExec (nxc)** : fingerprinting SMB/LDAP du domaine, de l'hôte et de la signature.
- **ldapsearch** : lectures du rootDSE et des contextes de nommage.
- **nltest / nslookup / dig** : découverte des DC et des enregistrements SRV.

## Références

- [NetExec (Pennyw0rth)](https://github.com/Pennyw0rth/NetExec)
- [ldapdomaindump (dirkjanm)](https://github.com/dirkjanm/ldapdomaindump)
- [Microsoft: Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-domain-services)
- [The Hacker Recipes: Active Directory recon](https://www.thehacker.recipes/ad/recon/)
