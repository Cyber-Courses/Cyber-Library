---
order: 12
title: "SSP personnalisé : journaliser les identifiants en clair sur l'hôte"
description: "Enregistrer un Security Support Provider malveillant sur un contrôleur de domaine ou un serveur pour que le sous-système d'authentification consigne le mot de passe en clair de chaque ouverture de session, y compris les mots de passe de comptes de service et machine, dans un fichier local."
keywords:
  - SSP personnalisé
  - mimilib
  - memssp
  - security packages
  - persistance de domaine
---

# SSP personnalisé

L'authentification Windows est enfichable : les **Security Support Providers (SSP)** sont des DLL chargées dans LSASS qui gèrent les échanges d'authentification. Enregistrer un **SSP malveillant** fait que LSASS remet à votre code les authentifications qui se produisent **sur cet hôte**, ouvertures de session interactives et de service, lequel enregistre le mot de passe en **clair** fourni pendant celles-ci. Il ne révèle **pas** le mot de passe derrière chaque authentification de domaine : un utilisateur se connectant depuis une autre machine ne donne au DC qu'un blob de pré-authentification Kerberos ou une réponse NTLM, pas son texte clair. Ce qu'il attrape bien sur un DC, ce sont les comptes qui s'y connectent réellement, les sessions administratives interactives et les comptes de service et machine démarrant localement, en clair.

## L'installer

Deux formes courantes, nécessitant toutes deux admin/SYSTEM sur la cible :

```text
# Persistent: drop mimilib.dll and register it as a Security Package (survives reboot)
#   copy mimilib.dll to C:\Windows\System32\
#   add "mimilib" to HKLM\System\CurrentControlSet\Control\Lsa\Security Packages (REG_MULTI_SZ)

# In-memory: patch LSASS to add a logging SSP without touching disk (not reboot-persistent)
mimikatz: privilege::debug ; misc::memssp
```

Les identifiants capturés sont écrits en clair dans un journal local :

```text
# mimilib.dll (registered SSP) logs to:
C:\Windows\System32\kiwissp.log
# misc::memssp (in-memory) logs to:
C:\Windows\System32\mimilsa.log
```

## Pourquoi c'est précieux

- Il fournit des mots de passe en **texte clair**, pas des hash, aucun cracking n'est donc nécessaire, et il attrape des identifiants qui ne sont jamais sous une forme de hash extractible au repos.
- Sur un **DC**, il attrape tout de même les comptes de service et machine qui se connectent localement, et toute ouverture de session administrative interactive, en clair, ce qui est de grande valeur même s'il ne s'agit pas de chaque authentification de domaine.
- La forme par **registre (`Security Packages`)** se recharge au redémarrage, ce qui la rend durable ; la forme `memssp` est plus discrète (aucun artefact sur disque) mais s'efface au redémarrage.

## Notes d'exploitation

- Nécessite que la **protection LSA (RunAsPPL)** soit désactivée ou contournée, puisque le SSP se charge dans LSASS ; un LSASS signé/avec PPL appliqué bloque un package non signé.
- C'est une porte dérobée de collecte, pas un accès immédiat : la valeur s'accumule au fil des ouvertures de session, laissez-la donc en place et récoltez.
- Lire `kiwissp.log` nécessite un accès local à l'hôte, associez-le donc à une méthode de ré-entrée ([DSRM](dsrm.md), un [golden ticket](../kerberos/forged-tickets.md)).

## Outils

- **Mimikatz** (`misc::memssp`, et le SSP `mimilib.dll`) : SSP de journalisation en mémoire et sur disque.

## Références

- [MITRE ATT&CK T1547.005: Security Support Provider](https://attack.mitre.org/techniques/T1547/005/)
- [pentestlab: persistence, Security Support Provider](https://pentestlab.blog/2019/10/21/persistence-security-support-provider/)
- [bufu-sec wiki: custom SSPs](https://wiki.bufu-sec.com/active-directory/persistence/custom_ssps)
