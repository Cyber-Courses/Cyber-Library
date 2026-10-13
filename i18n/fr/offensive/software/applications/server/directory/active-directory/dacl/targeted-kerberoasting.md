---
order: 4
title: "Targeted Kerberoasting : écrire un SPN pour roaster un utilisateur"
description: "Utiliser WriteSPN (ou GenericWrite) sur un utilisateur Active Directory pour définir un servicePrincipalName temporaire, demander et cracker son ticket de service, puis retirer le SPN, transformant une écriture sur un compte en son mot de passe hors ligne."
keywords:
  - targeted kerberoasting
  - WriteSPN
  - servicePrincipalName
  - targetedKerberoast
  - GenericWrite
---

# Targeted Kerberoasting

Le [Kerberoasting](../authentication/kerberos/roasting.md) ordinaire vise les comptes qui ont déjà un Service Principal Name. Le Kerberoasting **ciblé** fabrique cette condition : avec le droit d'écrire le **`servicePrincipalName`** d'un utilisateur (l'arête `WriteSPN`, ou tout `GenericWrite`/`GenericAll`), vous définissez un SPN arbitraire sur la victime, demandez un ticket de service pour celui-ci, le crackez hors ligne, et retirez le SPN. Cela convertit une écriture sur un utilisateur en le mot de passe de cet utilisateur sans jamais le réinitialiser.

## La séquence

```bash
# Automated: set an SPN on every writable user, roast, then clean up
targetedKerberoast.py -v -d example.local -u user -p pass

# Manual with bloodyAD
bloodyAD --host <dc> -d example.local -u user -p pass set object 'victim' servicePrincipalName -v 'fake/svc'
GetUserSPNs.py -request-user victim example.local/user:pass -dc-ip <dc> -outputfile roast.hash
bloodyAD --host <dc> -d example.local -u user -p pass set object 'victim' servicePrincipalName  # clear
hashcat -m 13100 roast.hash wordlist.txt -r rules/best64.rule
```

`targetedKerberoast.py` effectue toute la boucle (trouver les utilisateurs inscriptibles, ajouter un SPN temporaire, roaster, restaurer) et est l'outil habituel.

## Pourquoi le choisir

- Il est **non destructif** : contrairement à une [réinitialisation de mot de passe](password-reset.md) il ne change jamais le mot de passe de la victime, donc le compte continue de fonctionner et l'action est discrète.
- Il ne paie que si le mot de passe du compte est **crackable**, donc il convient aux comptes humains/de service avec des mots de passe faibles, pas aux comptes machine (dont les mots de passe aléatoires ne se crackeront pas).
- C'est le mouvement naturel quand une arête `GenericWrite` existe mais que PKINIT est indisponible pour les [shadow credentials](../authentication/kerberos/shadow-credentials.md) ; sinon les shadow credentials donnent un accès direct quelle que soit la robustesse du mot de passe.

## Notes d'exploitation

- Demandez des tickets **RC4** (etype 23, `-m 13100`) là où c'est autorisé ; ils se crackent bien plus vite que l'AES.
- **Retirez** toujours le SPN que vous avez ajouté ; laisser des SPN égarés est à la fois un désordre et un indicateur.
- Écrire un SPN nécessite que la valeur soit unique dans la forêt et syntaxiquement un SPN valide, mais elle n'a pas besoin de résoudre vers quoi que ce soit, donc `fake/svc` fonctionne.

## Outils

- **targetedKerberoast.py** : écriture-roast-nettoyage de bout en bout depuis Linux.
- **bloodyAD** / **PowerView** (`Set-DomainObject -Set @{serviceprincipalname=...}`) : définir/effacer le SPN manuellement.
- **Impacket `GetUserSPNs.py`**, **hashcat -m 13100** : demander et cracker.

## Références

- The Hacker Recipes: targeted Kerberoasting
- SpecterOps: BloodHound WriteSPN edge
