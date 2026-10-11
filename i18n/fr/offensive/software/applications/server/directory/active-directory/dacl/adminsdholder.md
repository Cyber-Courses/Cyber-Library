---
title: "AdminSDHolder : persistance via les DACL grâce à SDProp"
order: 6
description: "Création d'une porte dérobée dans Active Directory en écrivant une ACE dans le conteneur AdminSDHolder, que SDProp réapplique à chaque groupe et compte protégé selon sa minuterie, redonnant à l'attaquant des droits sur Domain Admins et les autres principaux protégés même après leur retrait."
keywords:
  - AdminSDHolder
  - SDProp
  - adminCount
  - groupes protégés
  - persistance
---

# AdminSDHolder

`AdminSDHolder` est un conteneur (`CN=AdminSDHolder,CN=System,DC=...`) dont la DACL est un **modèle**. Un processus en arrière-plan, le **Security Descriptor Propagator (SDProp)**, copie cette DACL environ toutes les **60 minutes** sur les groupes **protégés** du domaine et leurs membres actuels, résolus récursivement (Domain Admins, Administrators, Enterprise Admins et le reste de l'ensemble protégé), écrasant leurs DACL individuelles et y apposant `adminCount=1`. Cette conception, censée maintenir les objets privilégiés verrouillés de façon cohérente, est une primitive de persistance : écrivez une seule ACE dans le modèle AdminSDHolder et SDProp vous accorde des droits sur **tous** ces principaux, et vous les redonne à chaque cycle même si un défenseur les retire d'un objet individuel.

**Lignée.** AdminSDHolder et SDProp ont été livrés comme une protection, maintenant les objets privilégiés verrouillés de façon cohérente. Ils ont été détournés en persistance furtive à partir de 2015 environ, et le signe révélateur n'a pas changé depuis : un `adminCount=1` orphelin portant une ACE inattendue relie la conception originale à son détournement.

## Poser la porte dérobée

Avec le contrôle de l'objet AdminSDHolder (propriétaire, `WriteDacl` ou `GenericAll` dessus, généralement atteignable seulement une fois que vous êtes déjà très privilégié), ajoutez une ACE accordant le contrôle total à un principal que vous contrôlez :

```bash
# Impacket dacledit.py: write a FullControl ACE into the AdminSDHolder template
dacledit.py -action write -rights FullControl -principal 'user' \
  -target-dn 'CN=AdminSDHolder,CN=System,DC=example,DC=local' example.local/admin:pass

# bloodyAD
bloodyAD --host <dc> -d example.local -u admin -p pass add genericAll \
  'CN=AdminSDHolder,CN=System,DC=example,DC=local' 'user'
```

```powershell
# PowerView
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=example,DC=local' `
  -PrincipalIdentity user -Rights All
```

En l'espace d'un cycle SDProp, `user` détient `GenericAll` sur Domain Admins, Administrators et tout autre groupe et compte protégé, de sorte qu'il peut réinitialiser leurs mots de passe, ajouter des membres ou faire du DCSync à volonté. Pour éviter d'attendre, SDProp peut être déclenché sur le DC (l'opération `RunProtectAdminGroupsTask`).

## Pourquoi c'est durable

- L'octroi réside sur le **modèle**, pas sur le groupe, donc retirer vos droits de Domain Admins est annulé au cycle suivant ; le défenseur doit trouver et nettoyer la DACL d'AdminSDHolder elle-même.
- Il couvre l'ensemble de l'**ensemble protégé** du domaine d'un coup, donc une seule ACE accorde l'accès sur chaque groupe protégé et ses membres actuels. AdminSDHolder est maintenu **par domaine**, donc la porte dérobée est à l'échelle du domaine (répétez-la dans chaque domaine pour une portée à l'échelle de la forêt).
- Elle survit aux changements d'appartenance et aux réinitialisations de mot de passe des comptes protégés, parce qu'elle redonne la *permission*, pas un identifiant.

## Notes d'exploitation

- Il s'agit de **persistance post-compromission** : vous devez déjà disposer de privilèges élevés pour écrire AdminSDHolder, il s'agit donc de conserver l'accès, pas de l'obtenir.
- Les objets `adminCount=1` orphelins sont un signe révélateur connexe : un compte retiré des groupes protégés conserve le drapeau, mais SDProp **cesse** de rafraîchir sa DACL, donc il ne reçoit plus le modèle (et n'est donc pas l'endroit où atterrit la porte dérobée) ; le drapeau obsolète subsiste simplement après la rétrogradation.
- Écrire la DACL d'AdminSDHolder génère des événements de changement du service d'annuaire sur le DC, ce n'est donc pas furtif au moment de l'écriture ; sa valeur réside dans l'accès durable et discrètement réappliqué par la suite.

## Outils

- **Impacket `dacledit.py`** / **bloodyAD** : écrire l'ACE du modèle depuis Linux.
- **PowerView** (`Add-DomainObjectAcl`) : en local, et `Get-DomainObjectAcl` pour relire le modèle.

## Références

- The Hacker Recipes: AdminSDHolder
- SpecterOps: An ACE Up the Sleeve (AdminSDHolder backdoors)
