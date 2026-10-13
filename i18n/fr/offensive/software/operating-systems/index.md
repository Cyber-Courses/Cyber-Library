---
title: "Systèmes d'exploitation : techniques offensives contre la plateforme hôte"
description: "Techniques offensives visant le système d'exploitation lui-même, la plateforme qui héberge chaque application : élévation de privilèges locale, accès aux identifiants, persistance et exploitation du noyau, organisées par plateforme car Windows, Linux et macOS exposent des modèles de privilèges, des magasins d'identifiants et des services système fondamentalement différents."
keywords:
  - attaques du système d'exploitation
  - élévation de privilèges locale
  - accès aux identifiants
  - persistance
  - exploitation du noyau
---

# Systèmes d'exploitation

Là où [Applications](../applications/index.md) couvre le code qu'une organisation écrit, ce domaine couvre la plateforme sous-jacente : le noyau, les services système, les pilotes, ainsi que le modèle de privilèges et d'isolation dont tout ce qui est au-dessus dépend. C'est le travail qui commence une fois que vous avez déjà du code en cours d'exécution sur un hôte, généralement en tant qu'utilisateur non privilégié, et que vous voulez devenir `root` ou `SYSTEM`, lire les identifiants que la machine détient, survivre à un redémarrage et déjouer les frontières que le système d'exploitation impose.

Le domaine est organisé **par plateforme**, car la surface d'attaque est spécifique à chaque plateforme : le modèle de privilèges, les magasins d'identifiants, l'architecture des services et des pilotes, ainsi que les fonctionnalités d'isolation locale diffèrent fondamentalement entre les trois grands systèmes d'exploitation, tout comme les techniques et les outils qui les attaquent.

## Comment l'hôte est attaqué

Les mêmes objectifs reviennent sur chaque plateforme, atteints par des mécanismes propres à chacune :

- **Élévation de privilèges locale** : passer d'un utilisateur non privilégié au contrôle total de la machine, via une mauvaise configuration, des privilèges exploitables, des services et pilotes vulnérables, ou une faille du noyau.
- **Accès aux identifiants** : extraire mots de passe, hashes, clés et jetons depuis la mémoire, le registre ou le trousseau, et les magasins sur disque pour les réutiliser ailleurs.
- **Persistance** : survivre à la déconnexion et au redémarrage via les mécanismes de démarrage, de planification et de services de la plateforme.
- **Isolation et frontières** : déjouer les contrôles locaux de la plateforme (UAC, TCC, SIP, Gatekeeper, bacs à sable, namespaces).

## Plateformes

- **[Windows](windows/index.md)** : la plateforme de postes de travail et de serveurs d'entreprise dominante, dotée de la surface d'attaque locale la plus riche.
- **[Linux](linux/index.md)** : la plateforme de serveurs et d'infrastructure dominante, attaquée via son modèle de privilèges Unix et la configuration de ses services.
- **[macOS](macos/index.md)** : la plateforme de bureau d'Apple, avec ses propres contrôles de confidentialité, de signature et d'intégrité en couches.

## Portée et jonctions

Ce domaine couvre la surface d'attaque **locale** de l'hôte. Active Directory, qui est la manière dont les hôtes Windows se font confiance et s'authentifient entre eux au sein d'un domaine, est attaqué dans [Directory](../applications/server/directory/index.md) ; et les services accessibles par le réseau sur un hôte relèvent des domaines de [Server](../applications/server/index.md). Ceux-ci sont référencés ici plutôt que dupliqués.

## Références

- [MITRE ATT&CK: Privilege Escalation](https://attack.mitre.org/tactics/TA0004/)
- [MITRE ATT&CK: Credential Access](https://attack.mitre.org/tactics/TA0006/)
- [MITRE ATT&CK: Persistence](https://attack.mitre.org/tactics/TA0003/)
