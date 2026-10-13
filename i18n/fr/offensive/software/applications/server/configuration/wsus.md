---
title: "WSUS : pousser une mise à jour malveillante vers SYSTEM"
description: "Abuser de Windows Server Update Services pour exécuter du code en tant que SYSTEM sur les clients gérés : approuver une mise à jour malveillante depuis un serveur WSUS compromis, et en injecter une en coupure lorsque WSUS est servi en HTTP en clair."
keywords:
  - WSUS
  - SharpWSUS
  - WSUSpendu
  - mise à jour malveillante
  - SYSTEM
---

# WSUS

Windows Server Update Services distribue des mises à jour que les clients installent en tant que **SYSTEM**, ce qui est précisément la primitive recherchée par un attaquant : approuvez la bonne « mise à jour » et vous exécutez du code en tant que SYSTEM sur chaque machine visée. Il existe deux voies, l'une depuis un serveur WSUS compromis et l'autre depuis le réseau lorsque WSUS est servi en HTTP en clair.

## Approuver une mise à jour malveillante

Avec les droits d'administration sur le serveur WSUS, créez une mise à jour qui exécute un **binaire signé par Microsoft** avec des arguments choisis par l'attaquant, et approuvez-la pour un groupe d'ordinateurs cible. La contrainte de signature est satisfaite par des binaires living-off-the-land déjà approuvés (par exemple `PsExec.exe`), pilotés via des arguments en ligne de commande pour ajouter un administrateur ou exécuter une charge utile :

```text
# SharpWSUS: inspect, create, approve, then clean up
SharpWSUS.exe inspect
SharpWSUS.exe create /payload:"C:\PsExec64.exe" /args:"-accepteula -s cmd /c \"net user ...\"" /title:"Update"
SharpWSUS.exe approve /updateid:<guid> /computername:<target> /groupname:"Group"
```

`WSUSpendu` est l'approche PowerShell d'origine, qui injecte l'approbation directement dans la base de données WSUS, utile lorsque le flux de travail de la console est bloqué.

## Injection en coupure sur HTTP

Lorsque WSUS est configuré en **HTTP** plutôt qu'en HTTPS, un attaquant positionné en homme du milieu peut injecter une mise à jour malveillante vers les clients, puisque les métadonnées ne sont pas authentifiées au niveau du transport. Positionné entre le client et le serveur, livrez une « mise à jour » à binaire signé à chaque client qui se signale :

```text
# PyWSUS: serve a malicious update to clients whose WSUS URL is http:// (works on modern clients)
# WSUSpect is legacy (tested on Windows 7/8, not Windows 10/11), so prefer PyWSUS for modern targets
# combine with an MITM primitive (ARP, rogue DHCP/WPAD, mitm6) to reach the client's update traffic
```

## Notes d'exploitation

- La charge utile doit être **signée par Microsoft**, donc ces attaques associent un LOLBIN approuvé (PsExec, BgInfo) à ses arguments en ligne de commande plutôt que de déposer un exécutable non signé.
- Un serveur WSUS met souvent à jour **de nombreux** clients, et parfois d'autres serveurs et des DC, de sorte qu'une seule approbation équivaut à un SYSTEM à l'échelle de tout le parc, un peu comme le [déploiement d'applications](sccm/application-deployment.md) SCCM.
- La variante **HTTP** ne nécessite aucune compromission de WSUS, seulement un MITM réseau sur le trafic de mise à jour du client, alors vérifiez lors de la reconnaissance si WSUS fonctionne en HTTP.
- Nettoyez l'approbation et la mise à jour par la suite (SharpWSUS `delete`), car une mise à jour orpheline est à la fois bruyante et perturbatrice pour l'environnement.

## Outils

- **SharpWSUS** (LRQA/Nettitude) : inspecter, créer, approuver et supprimer des mises à jour malveillantes depuis un serveur compromis.
- **WSUSpendu** : injection PowerShell d'une approbation dans la base de données WSUS.
- **PyWSUS** : injection en coupure contre des clients WSUS en HTTP, y compris des Windows modernes.
- **WSUSpect** : le proxy en coupure d'origine, obsolète uniquement (Windows 7/8).

## Références

- [LRQA/Nettitude : introducing SharpWSUS](https://www.lrqa.com/en/cyber-labs/introducing-sharpwsus/)
- [GoSecure : PyWSUS, WSUS man-in-the-middle](https://github.com/GoSecure/pywsus)
- [h4ms1k : red teaming WSUS](https://h4ms1k.github.io/Red_Team_WSUS/)
