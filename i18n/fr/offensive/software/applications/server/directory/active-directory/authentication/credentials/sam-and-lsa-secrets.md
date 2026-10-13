---
order: 2
title: "SAM et secrets LSA : hashs locaux et secrets de machine et de service mis en cache"
description: "Dumper la base SAM locale pour les hashs des comptes locaux et les secrets LSA pour les identifiants de domaine mis en cache, les mots de passe de comptes de service et le secret du compte machine depuis les ruches de registre d'un hôte."
keywords:
  - SAM
  - secrets LSA
  - secretsdump
  - compte machine
  - identifiants mis en cache
---

# SAM et secrets LSA

Au-delà de la mémoire vive, un hôte Windows conserve du matériel d'identifiants dans le registre : la **SAM** contient les hashs des mots de passe des comptes locaux, et les **secrets LSA** contiennent les connexions de domaine mises en cache, les mots de passe des comptes de service et le propre secret du compte machine de l'hôte. Ils sont lisibles avec un accès administrateur local (SYSTEM) et, contrairement à LSASS, ne nécessitent pas que les principaux ciblés soient actuellement connectés.

## Dumper les ruches

Les secrets résident dans les ruches de registre `SAM`, `SYSTEM` et `SECURITY`. La méthode distante standard les récupère via SMB :

```bash
# Impacket: dump SAM + LSA + (if DC) NTDS over the network
secretsdump.py 'EXAMPLE/admin:password@<host>'
secretsdump.py -hashes :<nthash> 'EXAMPLE/admin@<host>'

# NetExec
nxc smb <host> -u admin -p pass --sam --lsa
```

Localement, sauvegardez les ruches et analysez-les hors ligne :

```
reg save HKLM\SAM sam.save & reg save HKLM\SYSTEM system.save & reg save HKLM\SECURITY security.save
secretsdump.py -sam sam.save -system system.save -security security.save LOCAL
```

## Ce que fournit chaque magasin

- **SAM** : hashs NTLM des comptes **locaux** (y compris l'administrateur local). Un mot de passe d'administrateur local réutilisé sur plusieurs machines permet le déplacement latéral par [pass-the-hash](../ntlm/pass-the-hash.md) ; là où LAPS le randomise, chaque hôte diffère.
- Les **secrets LSA** (ruche `SECURITY`) contiennent plusieurs éléments de grande valeur :
  - **`$MACHINE.ACC`** : le mot de passe/hash du compte machine, qui authentifie en tant qu'ordinateur (utile pour le RBCD, le S4U et les silver tickets contre les services de cet hôte).
  - **Connexions de domaine mises en cache** (`CACHEDDUMP`, le format MSCache2/DCC2) : les derniers utilisateurs de domaine à s'être connectés de façon interactive, crackables hors ligne mais pas utilisables directement pour le pass-the-hash.
  - **Secrets de comptes de service** : les mots de passe en clair des services configurés pour s'exécuter sous un compte de domaine, stockés afin que le service puisse démarrer.
  - **Sauvegarde de la clé maître DPAPI** et mots de passe d'ouverture de session automatique (`DefaultPassword`).

## Notes d'exploitation

- Le secret du compte machine est souvent négligé mais puissant : un compte d'ordinateur peut être le pivot pour la délégation contrainte basée sur les ressources et les silver tickets.
- Les hashs DCC2 (domaine mis en cache) utilisent un format lent et salé, ils craquent donc bien plus lentement que NTLM ; priorisez d'autres éléments sauf si le mot de passe est faible.
- Le mot de passe en clair d'un compte de service issu des secrets LSA est immédiatement utilisable et appartient fréquemment à un compte privilégié.

## Outils

- **secretsdump.py** (Impacket) : extraction SAM/LSA/NTDS distante et hors ligne.
- **NetExec (nxc) --sam --lsa** : dumps SAM et LSA rapides sur plusieurs hôtes.
- **Mimikatz** (`lsadump::sam`, `lsadump::secrets`) : dumps sur l'hôte.

## Références

- [The Hacker Recipes: SAM & LSA secrets](https://www.thehacker.recipes/ad/movement/credentials/dumping/sam-and-lsa-secrets)
- [Impacket secretsdump (fortra)](https://github.com/fortra/impacket)
- [NetExec: SAM and LSA dumping](https://github.com/Pennyw0rth/NetExec)
- [hashcat: example hashes (DCC2 is mode 2100)](https://hashcat.net/wiki/doku.php?id=example_hashes)
