---
title: "Print Spooler : coercition et exécution de code par chargement de pilote"
description: "Le Windows Print Spooler comme surface d'attaque : la primitive de coercition PrinterBug qui force une machine à s'authentifier auprès d'un attaquant via MS-RPRN, et les failles de chargement de pilote Point and Print (PrintNightmare, SpoolFool) qui exécutent du code en tant que SYSTEM."
keywords:
  - print spooler
  - PrinterBug
  - PrintNightmare
  - MS-RPRN
  - Point and Print
---

# Print Spooler

Le **Windows Print Spooler** s'exécute par défaut sur les postes de travail et les serveurs, y compris les contrôleurs de domaine, et expose des interfaces RPC distantes (MS-RPRN, MS-PAR) à tout utilisateur authentifié. C'est depuis des années l'un des services Windows les plus régulièrement abusés, de deux manières distinctes : on peut l'amener à **s'authentifier auprès d'un attaquant** (coercition), et on peut l'amener à **charger la DLL de pilote d'un attaquant en tant que SYSTEM** (exécution de code).

## Un long historique d'abus

La valeur offensive du spooler s'est accumulée au fil de plusieurs vagues, et chacune reste utile là où le patching ou le durcissement ont pris du retard :

- **2018, PrinterBug / SpoolSample** : Lee Christensen a montré que `RpcRemoteFindFirstPrinterChangeNotificationEx` force une cible à s'authentifier en retour via SMB, une primitive de [coercition](coercion.md) fiable encore utilisée pour alimenter les attaques de relais et de délégation.
- **2021, PrintNightmare** : une faille dans `RpcAddPrinterDriverEx` et Point and Print a permis à un utilisateur non privilégié de charger une DLL de pilote et d'exécuter en tant que SYSTEM, à la fois à distance et localement, traitée sous [Point and Print](point-and-print.md).
- **2022, SpoolFool** et des bugs d'élévation de privilèges du spooler ultérieurs ont poursuivi le schéma du SYSTEM local via le même service.

## Pages

- **[Coercition](coercion.md)** : forcer une machine à s'authentifier avec le PrinterBug, pour alimenter le relais et la délégation.
- **[Point and Print](point-and-print.md)** : charger un pilote pour exécuter en tant que SYSTEM, à distance et localement.

## Références

- [HackTricks: print spooler service abuse](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/printers-spooler-service-abuse.html)
- [itm4n: a practical guide to PrintNightmare in 2024](https://itm4n.github.io/printnightmare-exploitation/)
- [SpecterOps: SpoolSample / PrinterBug origin](https://github.com/leechristensen/SpoolSample)
