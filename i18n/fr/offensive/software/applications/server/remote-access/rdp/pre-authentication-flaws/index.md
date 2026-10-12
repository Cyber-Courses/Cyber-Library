---
title: "Failles de pré-authentification : RCE de protocole RDP propageable comme un ver"
description: "La gestion du protocole RDP antérieure à la session a produit des vulnérabilités d'exécution de code à distance non authentifiée et propageables comme un ver, la classe BlueKeep et DejaBlue, atteignables avant la connexion sur les hôtes sans authentification au niveau du réseau. Identifier le build Windows et le statut de la NLA permet de repérer les hôtes dans la plage affectée, où l'exploitation donne SYSTEM sans identifiants."
keywords:
  - rdp pre-auth rce
  - bluekeep
  - dejablue
  - wormable
  - nla
---

# Failles de pré-authentification

Les vulnérabilités RDP les plus graves sont en pré-authentification : des bugs de corruption de mémoire dans la gestion du protocole qui s'exécute avant qu'un utilisateur se connecte, atteignables par un client non authentifié et donnant une exécution de code en tant que SYSTEM. Parce que RDP est si largement exposé, ces failles sont propageables comme un ver, capables de s'auto-propager à travers les réseaux, ce qui explique pourquoi elles ont entraîné des correctifs urgents, hors cycle, et des comparaisons avec les épisodes de ver SMB. Elles résident dans le code antérieur à la session, de sorte que l'authentification au niveau du réseau (qui force d'abord l'authentification) bloque leur atteinte ; un hôte NLA désactivée du bon build est exposé. Les deux classes nommées sont BlueKeep et DejaBlue.

```bash
# identify hosts in the affected build range with NLA off
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <target>   # build + NLA status
nmap -p3389 --script rdp-vuln-ms12-020 <target>                   # (older RDP DoS/vuln check)
```

## Sous-thèmes

- **[BlueKeep](bluekeep.md)** : la RCE en pré-authentification dans le chemin de code RDP antérieur à la NLA.
- **[DejaBlue](dejablue.md)** : la classe suivante de RCE en pré-authentification affectant les builds plus récents.

## Références

- [Microsoft advisory: RDP pre-auth RCE](https://msrc.microsoft.com/update-guide/)
- [HackTricks: RDP vulnerabilities](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
