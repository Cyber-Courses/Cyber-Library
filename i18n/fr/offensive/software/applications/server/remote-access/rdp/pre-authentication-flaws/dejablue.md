---
title: "DejaBlue : RCE RDP en pré-authentification sur les builds Windows plus récents"
description: "DejaBlue est le nom collectif d'un ensemble de vulnérabilités RDP de pré-authentification et apparentées divulguées après BlueKeep, qui ont étendu le risque de RCE propageable comme un ver aux versions plus récentes de Windows. Comme BlueKeep, elles sont atteignables avant l'authentification là où l'authentification au niveau du réseau n'est pas imposée, donnant une exécution de code non authentifiée contre les builds concernés."
keywords:
  - dejablue
  - rdp pre-auth
  - newer windows
  - wormable
  - rdp rce
---

# DejaBlue

DejaBlue est l'étiquette collective d'un groupe de vulnérabilités RDP divulguées après BlueKeep, qui ont étendu le risque de RCE en pré-authentification et propageable comme un ver aux versions plus récentes de Windows (la gamme Windows 8/10 et Server 2012/2016/2019), que BlueKeep n'affectait pas. Elles résident dans la gestion du protocole RDP et, comme BlueKeep, sont atteignables avant l'authentification sur les hôtes où l'authentification au niveau du réseau n'est pas imposée, de sorte qu'un attaquant non authentifié contre un hôte concerné, non corrigé et NLA désactivée obtient l'exécution de code. L'ensemble a comblé la brèche qui laissait les Windows modernes exposés à la même classe d'attaque.

```bash
# identify exposure: affected newer build + NLA off
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <target>   # build + NLA status
# exploitation matches the specific DejaBlue variant and build; the preconditions
# (unpatched in-range build, NLA not required) are the same as BlueKeep.
```

## Notes d'exploitation

- DejaBlue compte parce qu'il couvre les builds Windows modernes que BlueKeep ne couvrait pas, donc « OS plus récent » n'est pas en soi une garantie de sécurité ; identifiez le build exact et le niveau de correctif.
- Les conditions préalables reflètent celles de BlueKeep : un build affecté et la NLA non requise ; imposer la NLA bloque l'atteinte en pré-authentification, et le correctif ferme le bug.
- Comme pour BlueKeep, ce sont des failles de corruption de mémoire dont l'exploitation fiable est sensible au build et peut déstabiliser l'hôte ; faites correspondre l'exploit à la cible précise.
- Là où la NLA est requise ou l'hôte corrigé, basculez plutôt vers les surfaces d'[authentification](../authentication/index.md) et d'[exposition](../exposure/index.md).

## Références

- [Microsoft advisory: DejaBlue RDP vulnerabilities](https://msrc.microsoft.com/update-guide/)
- [HackTricks: RDP vulnerabilities](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
