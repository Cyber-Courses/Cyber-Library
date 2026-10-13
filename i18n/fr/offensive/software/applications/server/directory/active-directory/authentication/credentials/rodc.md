---
order: 16
title: "RODC : identifiants mis en cache et golden ticket à portée restreinte"
description: "Compromettre un contrôleur de domaine en lecture seule pour extraire les identifiants que sa Password Replication Policy l'autorise à mettre en cache, et forger un golden ticket à portée restreinte avec la propre clé krbtgt du RODC qu'un DC inscriptible acceptera pour les principaux autorisés."
keywords:
  - RODC
  - Password Replication Policy
  - msDS-RevealedUsers
  - RODC krbtgt
  - golden ticket
---

# RODC

Un **contrôleur de domaine en lecture seule** (Read-Only Domain Controller) détient une copie filtrée et en lecture seule de l'annuaire et ne met en cache que les identifiants que sa **Password Replication Policy (PRP)** permet. Les RODC sont souvent déployés dans des agences et sont physiquement ou administrativement **moins protégés** que les DC inscriptibles, ce qui en fait une voie peu résistante vers les identifiants qu'ils détiennent, et vers une primitive de forge de tickets limitée mais réelle.

## Ce qu'un RODC met en cache

- `msDS-RevealOnDemandGroup` est la liste d'**autorisation** (comptes dont le RODC peut mettre en cache les mots de passe) ; `msDS-NeverRevealGroup` est la liste de **refus**.
- Par défaut, seuls le **compte d'ordinateur du RODC** et le **krbtgt du RODC** (`krbtgt_<number>`) sont mis en cache ; tout le reste l'est à la demande lorsqu'un principal autorisé s'authentifie.
- `msDS-RevealedList` sur l'objet RODC enregistre quels comptes ont effectivement été mis en cache, ce qui constitue une liste de cibles toute prête.

## Extraire les identifiants mis en cache

Avec un accès administrateur sur le RODC, dumpez les secrets qu'il a mis en cache localement pour chaque principal autorisé :

```bash
# an RODC does not replicate outward, so DRSUAPI (-just-dc) fails; read its LOCAL NTDS.dit
secretsdump.py -just-dc -use-vss 'EXAMPLE/rodc-admin@<rodc>'   # VSS snapshot of the local DB
# or copy ntds.dit + the SYSTEM hive off the box and extract offline:
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```

## Le golden ticket du RODC

Le RODC dispose de sa **propre clé krbtgt**. Forgez un TGT avec elle, et un DC **inscriptible** honorera ce ticket, mais uniquement pour les principaux que le RODC est autorisé à révéler (présents dans `msDS-RevealOnDemandGroup`, et non dans `msDS-NeverRevealGroup`) et uniquement avec le bon **numéro de version de clé** :

```bash
# Rubeus forges the RODC branch (sets the RODC number and matching kvno); plain ticketer.py
# hardcodes kvno 2 and cannot set the RODC number, so a writable DC rejects its ticket
Rubeus.exe golden /rc4:<rodc-krbtgt-hash> /rodcNumber:<N> /user:allowed_user /id:<rid> \
  /domain:example.local /sid:<sid> /groups:<allowed>
```

## Notes d'exploitation

- Le gain évolue avec la **PRP** : une politique qui autorise la mise en cache d'un compte privilégié (ou un groupe `Allowed RODC Password Replication Group` trop large) transforme la compromission du RODC en compromission de ce compte, et parfois en Domain Admin.
- Le golden ticket du RODC est **à portée restreinte**, ce n'est pas un golden ticket complet : il ne fonctionne que pour l'ensemble autorisé, énumérez donc `msDS-RevealOnDemandGroup` avant de forger.
- Les RODC se situent à la périphérie du Tier Zero et sont fréquemment gérés par des administrateurs non Tier-0, de sorte que la délégation `managedBy`/d'administration locale sur l'objet RODC est elle-même une voie d'accès.
- Les hashs mis en cache alimentent le [pass-the-hash](../ntlm/pass-the-hash.md) et le krbtgt du RODC récupéré alimente le [ticket forgé](../kerberos/forged-tickets.md) à portée restreinte.

## Outils

- **Impacket `secretsdump.py`** (`-use-vss` ou hors ligne `-ntds ... LOCAL`) : dumper les secrets mis en cache localement par le RODC.
- **Rubeus** (`golden /rodcNumber`) : forger le TGT à portée RODC avec le bon numéro de RODC et le bon kvno.
- **mimikatz** : extraction de secrets locaux sur la machine.

## Références

- [adsecurity.org: attacking Read-Only Domain Controllers](https://adsecurity.org/?p=3592)
- [SpecterOps: at the edge of Tier Zero, the curious case of the RODC](https://posts.specterops.io/at-the-edge-of-tier-zero-the-curious-case-of-the-rodc-ef5f1799ca06)
- [The Hacker Recipes: RODC](https://www.thehacker.recipes/ad/movement/builtins/rodc)
