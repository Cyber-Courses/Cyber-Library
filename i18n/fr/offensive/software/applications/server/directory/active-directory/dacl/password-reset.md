---
title: "Réinitialisation de mot de passe : prendre le contrôle d'un compte avec ForceChangePassword"
order: 2
description: "Utilisation du droit de contrôle d'accès User-Force-Change-Password (ou GenericAll) sur un compte Active Directory pour définir un nouveau mot de passe sans connaître l'ancien, depuis Linux avec net rpc, bloodyAD et Impacket, et le compromis par rapport aux prises de contrôle plus discrètes."
keywords:
  - ForceChangePassword
  - réinitialisation de mot de passe
  - bloodyAD
  - net rpc password
  - prise de contrôle de compte
---

# Réinitialisation de mot de passe

Le droit étendu **`User-Force-Change-Password`** (et `GenericAll`, qui l'inclut) vous permet de définir un **nouveau** mot de passe sur un compte cible sans fournir l'ancien, la même opération qu'utilise un helpdesk. C'est le moyen le plus direct de transformer une écriture sur un utilisateur en contrôle de celui-ci : réinitialisez le mot de passe, puis connectez-vous en tant que lui. Le coût est que c'est destructif, c'est donc un choix délibéré plutôt que l'option par défaut.

## Réinitialiser depuis Linux

```bash
# bloodyAD
bloodyAD --host <dc> -d example.local -u user -p pass set password 'victim' 'Newpass123!'

# Samba net (SAMR), works with password or -k for Kerberos
net rpc password 'victim' 'Newpass123!' -U 'example.local/user%pass' -S <dc>

# Impacket (reset via SAMR without knowing the current password)
changepasswd.py example.local/victim@<dc> -reset -altuser example.local/user -altpass 'pass' -newpass 'Newpass123!'
```

Depuis Windows, PowerView fait la même chose :

```powershell
Set-DomainUserPassword -Identity victim -AccountPassword (ConvertTo-SecureString 'Newpass123!' -AsPlainText -Force)
```

## Quand l'utiliser, et quand ne pas le faire

Une réinitialisation est bruyante : l'utilisateur légitime ne peut plus se connecter, ce qui est remarqué rapidement et génère des événements de gestion de compte. Donc :

- **Préférez d'abord une arête non destructive.** Le même `GenericWrite`/`GenericAll` permet généralement aussi les [shadow credentials](../authentication/kerberos/shadow-credentials.md) (s'authentifier en tant que l'utilisateur sans toucher au mot de passe) ou le [Kerberoasting ciblé](targeted-kerberoasting.md). Utilisez-les quand le compte est en usage.
- **La réinitialisation convient bien pour** les comptes obsolètes ou de service que personne n'utilise pour se connecter, les comptes d'ordinateur (bien que réinitialiser un mot de passe d'ordinateur puisse casser l'appartenance au domaine de l'hôte), ou quand PKINIT est indisponible de sorte que les shadow credentials ne sont pas une option.
- **Rétablissez quand c'est possible** : si vous avez d'abord capturé l'ancienne empreinte NT, vous pouvez remettre le mot de passe par empreinte ensuite pour réduire la perturbation.

## Notes d'exploitation

- `ForceChangePassword` est une *réinitialisation* (définir un nouveau, pas besoin de l'ancien), distincte d'un *changement* (nécessite le mot de passe actuel) ; l'arête `ForceChangePassword` de BloodHound est exactement ce droit.
- Sur un compte **ordinateur**, préférez une arête key-credential ou RBCD : réinitialiser `MACHINE$` désynchronise le secret de l'hôte et est à la fois bruyant et perturbateur.
- Après la réinitialisation vous détenez le texte en clair, qui alimente donc directement le déplacement authentifié et, si le compte est privilégié, d'autres arêtes de DACL.

## Outils

- **bloodyAD** (`set password`) : réinitialisation en une seule commande depuis Linux.
- **Samba `net rpc password` / Impacket `changepasswd.py -reset`** : réinitialisation basée sur SAMR.
- **PowerView** (`Set-DomainUserPassword`) : réinitialisation en local.

## Références

- The Hacker Recipes: ForceChangePassword
- SpecterOps: BloodHound ForceChangePassword edge
