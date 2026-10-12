---
title: "Réseau"
order: 7
description: "Périmètre offensif des services exposés sur le réseau et des interfaces RPC que présentent les hôtes Windows et d'infrastructure, où un seul service atteignable peut contraindre une authentification ou exécuter du code, avec le Windows Print Spooler comme exemple de longue date."
keywords:
  - network services
  - RPC
  - print spooler
  - coercion
  - Windows services
---

# Réseau

Les hôtes Windows et d'infrastructure exposent une large gamme de services atteignables sur le réseau et d'interfaces RPC, et beaucoup d'entre eux ont été conçus pour un LAN de confiance plutôt que pour un LAN hostile. Un seul service atteignable peut suffire à **contraindre une machine à s'authentifier** auprès d'un attaquant, ou à **charger du code en tant que SYSTEM**, ce qui fait de ces interfaces un pilier de la compromission interne.

## Services

- **[Print Spooler](print-spooler/index.md)** : le service d'impression de Windows (MS-RPRN / MS-PAR), source à la fois de la primitive de coercition PrinterBug et de l'exécution de code par chargement de pilote Point and Print.

## Références

- [Microsoft: MS-RPRN, Print System Remote Protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rprn/d42db7d5-f141-4466-8f47-0a4be14e2fc1)
- [HackTricks: print spooler service abuse](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/printers-spooler-service-abuse.html)
