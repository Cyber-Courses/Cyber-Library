---
order: 14
title: "Usurpation de token : de SeImpersonate à SYSTEM"
description: "Transformer le privilège SeImpersonatePrivilege que détiennent les comptes de service en SYSTEM en forçant un processus privilégié à s'authentifier auprès d'un endpoint contrôlé et en volant son token, la famille potato de techniques qui relie un point d'appui en compte de service au contrôle local complet."
keywords:
  - SeImpersonatePrivilege
  - usurpation de token
  - PrintSpoofer
  - RoguePotato
  - GodPotato
---

# Usurpation de token

Lorsque vous atterrissez sur un hôte en tant que **compte de service** (IIS, MSSQL, et la plupart des identités de service), vous détenez généralement **`SeImpersonatePrivilege`** : le droit d'usurper l'identité d'un client qui s'authentifie auprès de vous. C'est tout l'enjeu. Forcez un processus **SYSTEM** à s'authentifier auprès d'un endpoint que vous contrôlez, capturez et usurpez son token, et vous vous exécutez en tant que SYSTEM. Cette famille « potato » est le pont standard d'un point d'appui restreint en compte de service vers le contrôle local complet, et de là vers [LSASS](lsass-dumping.md) et le reste des secrets de l'hôte.

## Confirmer le privilège

```cmd
whoami /priv
:: look for SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege = Enabled
```

## Les variantes

Elles ne diffèrent que par **la façon dont elles forcent un processus SYSTEM à s'authentifier** :

```cmd
:: PrintSpoofer: spooler RpcRemoteFindFirstPrinterChangeNotification over a named pipe (modern default)
PrintSpoofer.exe -i -c cmd.exe

:: GodPotato: a broad DCOM/RPC method that works across modern Windows versions
GodPotato.exe -cmd "cmd /c whoami"

:: RoguePotato: DCOM OXID resolver abuse for post-1809 systems
:: JuicyPotato: the original DCOM method, for legacy targets (pre-1809)
```

## Une lignée de potatoes

La famille a suivi les tentatives de Microsoft de fermer chaque canal de coercition :

- **2016, Hot/RottenPotato** : l'idée originale de capture de token par NBNS + DCOM.
- **2018, JuicyPotato** : a militarisé largement le chemin DCOM, jusqu'à ce que le changement d'OXID de la 1809 le casse.
- **2019-2020, RoguePotato et PrintSpoofer** : ont restauré l'abus de DCOM post-1809 et ajouté le chemin du named pipe du spouleur.
- **À partir de 2022, GodPotato et les successeurs DCOM/RPC** : des méthodes larges qui continuent de fonctionner sur les Windows actuels.

## Notes d'exploitation

- Le privilège est détenu par **défaut** par les comptes de service, de sorte qu'un web shell ou un `xp_cmdshell` SQL sous l'identité de service est presque toujours à un pas de SYSTEM.
- **PrintSpoofer** est le premier essai habituel sur les hôtes modernes ; repliez-vous sur **GodPotato**/RoguePotato pour le chemin DCOM et JuicyPotato sur le legacy.
- SYSTEM sur un hôte joint au domaine signifie son **compte machine** et tous les identifiants mis en cache, cela pivote donc directement vers le déplacement dans le domaine via [pass-the-hash](../ntlm/pass-the-hash.md) ou [coercition](../ntlm/coercion.md).
- Ces techniques abusent d'API de token locales à l'hôte, pas d'AD, elles fonctionnent donc indépendamment du durcissement du domaine une fois que vous détenez un contexte de compte de service.

## Outils

- **PrintSpoofer** (itm4n) : coercition via le named pipe du spouleur.
- **GodPotato** / **RoguePotato** (antonioCoco) : coercition DCOM moderne et post-1809.
- **JuicyPotato** : la technique DCOM héritée.

## Références

- [itm4n: PrintSpoofer, abusing impersonation privileges](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)
- [GodPotato (antonioCoco)](https://github.com/BeichenDream/GodPotato)
- [HackTricks: abusing tokens and SeImpersonate](https://hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens.html)
