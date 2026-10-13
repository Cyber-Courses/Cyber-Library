---
order: 10
title: "Skeleton Key : un mot de passe maître injecté dans LSASS"
description: "Patcher le processus LSASS d'un contrôleur de domaine pour que chaque compte s'authentifie avec un mot de passe maître choisi par l'attaquant en plus du sien, offrant un accès à l'échelle du domaine jusqu'au redémarrage du DC."
keywords:
  - skeleton key
  - patch LSASS
  - misc skeleton
  - downgrade RC4
  - persistance de domaine
---

# Skeleton Key

Skeleton Key patche le chemin d'authentification **en mémoire** sur un contrôleur de domaine de sorte que, aux côtés du véritable mot de passe de chaque compte, un unique **mot de passe maître** de votre choix est également accepté. Une fois appliqué, vous pouvez vous connecter en tant que **n'importe quel** utilisateur du domaine avec ce seul mot de passe tandis que les connexions légitimes continuent de fonctionner, de sorte que rien ne se casse de façon visible. C'est un accès rapide à l'échelle du domaine, mais il ne vit que dans la mémoire de LSASS, il ne **survit** donc **pas** à un redémarrage.

## L'appliquer

Avec un accès Domain Admin et la capacité d'exécuter du code en tant que SYSTEM sur un DC (ou de patcher LSASS à distance), injectez la clé :

```text
# Mimikatz on the DC (or via remote code execution)
privilege::debug
misc::skeleton
```

Après cela, authentifiez-vous en tant que n'importe quel utilisateur avec le mot de passe maître (Mimikatz utilise `mimikatz` comme maître) :

```bash
# From Linux, log in as any account with the skeleton password
nxc smb <dc> -u Administrator -p mimikatz
```

## Pourquoi cela fonctionne, et ses limites

- Cela patche le chemin **RC4** dans LSASS, cela nécessite donc que Kerberos RC4 soit disponible ; les environnements forcés en **AES uniquement** peuvent le déjouer ou le compliquer, et un re-patch pour l'AES est nécessaire.
- Le patch est **en mémoire uniquement** : un redémarrage du DC l'efface, Skeleton Key est donc une porte dérobée *de la durée d'une session*, réappliquée au besoin ou associée à une technique qui survit au redémarrage ([DSRM](dsrm.md), [SSP personnalisé](custom-ssp.md), [golden ticket](../kerberos/forged-tickets.md)).
- L'appliquer de façon fiable nécessite **LSA Protection (RunAsPPL) désactivée** ou un contournement de PPL, et cela touche un processus surveillé, c'est donc puissant mais pas subtil.

## Notes d'exploitation

- L'intérêt est la **commodité** : un seul mot de passe vous connecte en tant que n'importe qui, y compris des comptes dont vous n'avez jamais dumpé les hashs.
- Comme les vrais mots de passe fonctionnent toujours, les utilisateurs et administrateurs ne remarquent rien, ce qui procure du temps de présence jusqu'au prochain redémarrage.
- Traitez-le comme tactique : utilisez-le pour une fenêtre d'accès large, et posez séparément une porte dérobée durable.

## Outils

- **Mimikatz** (`misc::skeleton`) : l'implémentation de référence.
- **Impacket / NetExec** : s'authentifier en tant que n'importe quel compte avec le mot de passe maître une fois appliqué.

## Références

- [MITRE ATT&CK T1556.001: Modify Authentication Process, Domain Controller Authentication](https://attack.mitre.org/techniques/T1556/001/)
- [pentestlab: Skeleton Key](https://pentestlab.blog/2018/04/10/skeleton-key/)
- [The Hacker Recipes: Skeleton Key](https://www.thehacker.recipes/ad/persistence/skeleton-key)
