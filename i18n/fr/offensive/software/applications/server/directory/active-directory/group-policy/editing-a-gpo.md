---
title: "Éditer une GPO : exécution de code à travers sa portée"
order: 2
description: "Transformer un accès en écriture sur un objet de stratégie de groupe Active Directory en exécution de code sur chaque ordinateur ou utilisateur auquel il s'applique, en injectant une tâche planifiée immédiate, un script de démarrage ou d'ouverture de session, ou une appartenance au groupe administrateur local, avec pyGPOAbuse et SharpGPOAbuse."
keywords:
  - abus de GPO
  - tâche planifiée immédiate
  - pyGPOAbuse
  - SharpGPOAbuse
  - SYSVOL
---

# Éditer une GPO

Une GPO que vous pouvez **écrire au niveau de l'objet d'annuaire** (une arête `GenericWrite`/`GenericAll`/`WriteProperty`) est une exécution de code sur tout ce à quoi la GPO est liée. Vous ajoutez un élément de stratégie, les machines clientes le récupèrent à leur prochain rafraîchissement, et il s'exécute. L'élément à plus fort impact est une **tâche planifiée immédiate**, qui s'exécute une fois dès que la stratégie s'applique, en tant que **SYSTEM** sur les ordinateurs dans la portée.

Injecter un **nouvel** élément demande plus que les fichiers SYSVOL : un client ne traite une zone de stratégie que si son extension côté client est enregistrée dans l'attribut **`gPCMachineExtensionNames`** (ou `gPCUserExtensionNames`) de l'objet GPO, et seulement lorsque le **`versionNumber`** de l'objet s'incrémente. Ce sont des attributs LDAP sur l'objet GPO, donc une toute nouvelle tâche planifiée nécessite l'écriture sur l'objet d'annuaire, pas seulement sur le dossier SYSVOL. Un accès en écriture au **seul** dossier SYSVOL est plus faible : il vous permet d'altérer du contenu que la GPO référence **déjà** (un script ou une tâche existante), pas d'ajouter une nouvelle zone de stratégie. Les outils ci-dessous effectuent à la fois l'écriture SYSVOL et les mises à jour de l'annuaire, ils supposent donc l'écriture sur l'objet.

## Ce que vous pouvez pousser

- **Tâche planifiée immédiate** : la plus rapide et la plus fiable ; exécute votre commande en tant que SYSTEM (GPO d'ordinateur) ou en tant qu'utilisateur (GPO d'utilisateur) au prochain rafraîchissement.
- **Script de démarrage / d'ouverture de session** : s'exécute au démarrage (SYSTEM) ou à l'ouverture de session (utilisateur), persistant à travers les rafraîchissements jusqu'au retrait.
- **Appartenance aux administrateurs locaux** : ajouter un compte contrôlé au groupe Administrateurs local de chaque machine dans la portée (un changement de type Restricted Groups / LGPO).
- **Services, autoruns du registre** : exécution et persistance alternatives via une configuration délivrée par stratégie.

## Le faire depuis Linux et Windows

```bash
# pyGPOAbuse (Linux): add an immediate scheduled task to a writable GPO.
# Default action makes a local admin; -command runs an arbitrary payload as SYSTEM.
pygpoabuse.py example.local/user -hashes :<nthash> -gpo-id '<GPO-GUID>' \
  -command 'net user hacker P@ss123! /add && net localgroup administrators hacker /add' \
  -taskname 'Update' -description 'Update task'
```

```powershell
# SharpGPOAbuse (Windows): local admin, or an arbitrary computer task
SharpGPOAbuse.exe --AddLocalAdmin --UserAccount svc_backup --GPOName "Vulnerable GPO"
SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author DOMAIN\user `
  --Command "cmd.exe" --Arguments "/c <payload>" --GPOName "Vulnerable GPO"

# PowerView: immediate task via the GPO display name
New-GPOImmediateTask -TaskName Update -Command cmd -CommandArguments "/c <payload>" -GPODisplayName "Vulnerable GPO" -Force
```

Ces outils écrivent le fichier de stratégie (par exemple `ScheduledTasks.xml`) dans le dossier SYSVOL de la GPO et augmentent la version de la GPO (`GPT.ini`) ainsi que la liste des extensions côté client (`gPCMachineExtensionNames`) pour que les clients la traitent.

## Notes d'exploitation

- L'application se fait selon le **cycle de rafraîchissement de la stratégie de groupe** (~90 minutes plus un décalage aléatoire sur les postes de travail, 5 minutes sur les DC), ou immédiatement avec `gpupdate /force` si vous pouvez l'exécuter ; une tâche immédiate se déclenche alors à la prochaine application.
- **La portée est primordiale** : une GPO liée à l'OU Domain Controllers exécute votre tâche en tant que SYSTEM sur les DC, ce qui constitue une compromission du domaine. Confirmez les liaisons de la GPO (voir [énumération](gpo-and-ou-enumeration.md)) avant de déclencher.
- **Nettoyez** : retirez la tâche/le script et rétablissez le numéro de version ensuite ; une tâche immédiate résiduelle se redéclenche et constitue un artefact évident dans SYSVOL.
- Une tâche immédiate à portée ordinateur s'exécute en tant que SYSTEM et ne nécessite aucune ouverture de session utilisateur, elle est donc préférée aux scripts d'ouverture de session pour la fiabilité.

## Outils

- **pyGPOAbuse** : tâche planifiée immédiate / admin local depuis Linux.
- **SharpGPOAbuse** : `--AddLocalAdmin`, `--AddComputerTask`, `--AddUserTask`, `--AddUserRights` sous Windows.
- **PowerView `New-GPOImmediateTask`** : tâche immédiate depuis un shell.

## Références

- The Hacker Recipes: Group policies
- SpecterOps: GPO abuse and the GenericWrite-over-GPO edge
