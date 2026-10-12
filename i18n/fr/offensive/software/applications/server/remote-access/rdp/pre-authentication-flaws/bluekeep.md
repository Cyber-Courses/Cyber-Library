---
title: "BlueKeep : exécution de code à distance RDP non authentifiée"
description: "BlueKeep est un use-after-free en pré-authentification dans la gestion des canaux virtuels par RDP, atteignable avant la connexion sur les anciens Windows sans authentification au niveau du réseau. Un attaquant non authentifié déclenche la faille via une requête de canal forgée pour exécuter du code en tant que SYSTEM, et le bug est propageable comme un ver, ce qui a motivé le correctif d'urgence des Windows en fin de vie."
keywords:
  - bluekeep
  - use-after-free
  - virtual channel
  - mst120
  - wormable
---

# BlueKeep

BlueKeep est un use-after-free en pré-authentification dans la gestion des canaux virtuels par le serveur RDP, plus précisément la liaison interne d'un canal statique qui permettait à un attaquant de libérer puis de réutiliser un objet noyau avant l'authentification. Il affecte les anciens Windows (l'ère Windows 7 / Server 2008 R2 et antérieure, y compris les Windows XP/2003 en fin de vie) et n'est atteignable que lorsque l'authentification au niveau du réseau n'est pas imposée, car il réside dans le code antérieur à la session. Un attaquant non authentifié le déclenche par une configuration de canal forgée et un grooming du tas pour obtenir l'exécution de code en tant que SYSTEM. Sa propagabilité comme un ver a poussé Microsoft à corriger même les Windows hors support.

```bash
# identify exposure: affected build + NLA off
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <target>
# scanners/exploit modules exist (use the one matching the exact target build)
#   the exploit binds the vulnerable channel, grooms the kernel pool, and triggers
#   the UAF to execute a payload as SYSTEM; kernel-pool layout makes it build-sensitive.
```

## Notes d'exploitation

- Deux conditions préalables : un build Windows affecté (Windows 7/Server 2008 R2 et antérieur) et la NLA non requise ; `rdp-ntlm-info` donne le build et `rdp-enum-encryption` le statut de la NLA.
- La faille est dans la gestion de canal en pré-authentification, donc la NLA (CredSSP d'abord) bloque son atteinte ; imposer la NLA était la mesure provisoire d'atténuation avant le correctif.
- Une exploitation fiable nécessite de groomer le pool noyau pour la cible spécifique, elle est donc sensible au build et à la configuration et peut faire planter l'hôte si la disposition est erronée ; c'est le risque standard de la classe des UAF noyau.
- Le succès donne SYSTEM sans identifiants, et la propagabilité du bug comme un ver signifie qu'un seul hôte exploité peut servir à se propager ; la classe suivante sur les builds plus récents est [DejaBlue](dejablue.md).

## Références

- [Microsoft advisory: BlueKeep](https://msrc.microsoft.com/update-guide/)
- [NCC/ZDI BlueKeep analyses](https://www.zerodayinitiative.com/blog)
