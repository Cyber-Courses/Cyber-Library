---
order: 1
title: "Vulnérabilités du code applicatif"
description: "Failles qui proviennent du code source applicatif que l'équipe de développement contrôle : composition non sûre de requêtes et de commandes, autorisation cassée et erreurs de logique métier."
keywords:
  - code applicatif
  - logique métier
  - autorisation
  - injection
  - codage sécurisé
---

# Code

Ce sous-arbre couvre les vulnérabilités qui proviennent du **code applicatif** que l'équipe de développement contrôle : composition non sûre de requêtes et de commandes, autorisation et contrôle d'accès cassés, failles de logique et de workflow, et mauvais usage des frameworks. Celles-ci subsisteraient même si le serveur web et le runtime étaient remplacés par des configurations par défaut sûres, car la faute est dans le code source.

La mauvaise configuration du serveur et du reverse proxy relève de la couche plateforme, et les bugs d'interpréteur ou de moteur relèvent de la couche runtime ; cette section concerne « même serveur, même runtime, code applicatif vulnérable ».

## Sous-thèmes

- **[Contrôle d'accès](access-control/index.md)** : contrôle d'accès cassé dans le code web côté serveur : portée des endpoints, des objets et des propriétés, plus la confiance envers les en-têtes et les systèmes amont.
- **[Authentification](authentication/index.md)** : un guide offensif pour casser la façon dont le code côté serveur prouve l'identité (gestion des identifiants, fédération, MFA, sessions et jetons) avec la reconnaissance et la métho...
- **[Identification](identification/index.md)** : comment le comportement d'une application révèle quels comptes existent, facilite le credential stuffing et expose les identifiants utilisateur, la reconnaissance pré-authentification qui alimente la prise...
- **[Injection](injection/index.md)** : données non fiables liées à quelque chose que le serveur exécute, analyse, stocke, rend ou requête, organisées par la primitive dangereuse attaquée.
- **[Workflow](workflow/index.md)** : logique métier et intégrité d'état dans les flux web multi-étapes et multi-services : séquencement, concurrence et intégrité des paramètres.
