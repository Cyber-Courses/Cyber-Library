---
title: "Authentification et autorisation : prouver l'identité, puis accorder l'accès"
description: "Les deux fonctions de sécurité complémentaires qui établissent d'abord qui est un principal, puis décident ce que ce principal est autorisé à faire."
keywords:
  - authentification
  - autorisation
  - contrôle d'accès
  - vérification d'identité
  - authentification multifacteur
  - principes de sécurité
---

# Authentification et autorisation

**L'authentification et l'autorisation** sont deux étapes distinctes qui fonctionnent ensemble pour contrôler l'accès. L'authentification répond à la question "qui êtes-vous ?" en vérifiant qu'un principal (un utilisateur, un service ou un appareil) est bien celui qu'il prétend être. L'autorisation répond à une question différente, "que pouvez-vous faire ?", en décidant quelles actions et quelles ressources ce principal vérifié est autorisé à atteindre. L'ordre compte : un système établit d'abord l'identité, puis applique les règles qui régissent ce que cette identité peut toucher. Confondre les deux, ou en omettre une, est une source fréquente de défaillances de sécurité graves.

## Deux questions, deux défaillances différentes

Un bâtiment en donne une image utile : l'authentification est la porte d'entrée qui tient à l'écart les inconnus et les imposteurs, et l'autorisation est l'ensemble des couloirs verrouillés qui empêchent un visiteur vérifié d'entrer dans des pièces qui ne le regardent pas. Comme les deux étapes répondent à des questions différentes, elles échouent aussi de manière différente, et les confondre conduit à corriger le mauvais problème.

| | Authentification | Autorisation |
|---|---|---|
| Question | Qui émet cette requête ? | Ce principal a-t-il le droit de faire cela, sur cette ressource ? |
| Preuves typiques | Mots de passe, jetons, passkeys, biométrie | Rôles, attributs, propriété, règles de politique |
| À quoi ressemble une défaillance | Un attaquant se connecte avec un mot de passe deviné, hameçonné ou réutilisé | Un utilisateur ordinaire lit les dossiers d'un administrateur |
| Ce qu'elle rend possible | La responsabilité et des journaux d'audit qui ont du sens | Le moindre privilège en pratique |

Les identifiants volés et les contrôles d'accès défaillants figurent parmi les faiblesses les plus fréquentes et les plus dommageables des systèmes modernes. L'OWASP classe le contrôle d'accès défaillant (Broken Access Control) au premier rang de son Top 10 pour les applications web, dans les éditions 2021 comme 2025.

## Prouver l'identité : les facteurs et la façon de les contourner

Les preuves d'authentification sont traditionnellement regroupées en trois facteurs. Chacun a son propre point faible, d'où l'intérêt de les combiner.

- **Quelque chose que vous connaissez :** les mots de passe et les codes PIN. Ils peuvent être devinés, hameçonnés, ou divulgués par un service puis rejoués sur un autre lorsque les gens les réutilisent.
- **Quelque chose que vous possédez :** les jetons matériels, les applications d'authentification ou les passkeys liées à un appareil. Ils résistent aux tentatives de devinette à distance, mais l'appareil peut être perdu, ou l'utilisateur amené par la ruse à approuver une connexion.
- **Quelque chose que vous êtes :** les empreintes digitales, le visage et les autres données biométriques. Ils sont pratiques, mais ne peuvent pas être changés en cas de compromission ; ils servent donc généralement à déverrouiller une clé locale plutôt qu'à circuler sur le réseau.

**L'authentification multifacteur** combine deux de ces facteurs ou plus, afin qu'un seul secret volé ne suffise pas à usurper l'identité de quelqu'un. **L'identité fédérée et l'authentification unique** confient ce travail à un fournisseur d'identité de confiance : il authentifie l'utilisateur une seule fois, et les applications qui s'y fient acceptent son assertion signée au lieu de conserver chacune leur propre base de mots de passe.

## Décider de l'accès : des rôles aux politiques

Une fois l'identité établie, les règles d'autorisation doivent être écrites sous une forme que des personnes peuvent maintenir et relire. Trois familles de modèles sont courantes :

1. **Contrôle d'accès basé sur les rôles (RBAC) :** les permissions sont rattachées à des rôles tels que "agent de facturation" ou "auditeur", et les utilisateurs reçoivent des rôles. Simple à comprendre, mais les rôles ont tendance à se multiplier avec le temps.
2. **Contrôle d'accès basé sur les attributs (ABAC) :** les décisions prennent en compte des attributs de l'utilisateur, de la ressource et du contexte (service, classification des données, heure de la journée).
3. **Contrôle d'accès basé sur les politiques :** les règles sont exprimées sous forme de politiques gérées de manière centralisée, évaluées par un moteur de décision et appliquées de façon cohérente à l'ensemble des services.

Quel que soit le modèle, l'application des règles se fait côté serveur et s'exécute à chaque requête. L'interface utilisateur peut masquer un bouton par commodité, mais ce n'est jamais là que l'accès est réellement décidé. L'autorisation est aussi la couche où le [moindre privilège](least-privilege.md) devient concret : chaque identité ne reçoit que ce dont sa tâche a besoin.

## Suivre une requête : "afficher la facture 1042"

Prenons une utilisatrice connectée, Alice, dont le navigateur demande la facture 1042 à une application de comptabilité. Suivre la requête étape par étape montre où chaque contrôle doit se trouver et ce qui se passe lorsque l'un d'eux manque.

1. **Le jeton de session d'Alice arrive.** Le serveur le valide et conclut "c'est Alice". C'est l'authentification, et elle n'est jamais plus solide que la connexion qui l'a précédée : si Alice a réutilisé un mot de passe qui a fuité ailleurs, un attaquant peut simplement se connecter à sa place et recevoir un jeton parfaitement valide.
2. **Le serveur recherche la facture 1042.** Savoir qui est Alice ne dit encore rien sur le fait que cette facture lui appartienne.
3. **Le contrôle de propriété.** Le serveur doit confirmer que la facture 1042 appartient au compte d'Alice, ou que son rôle l'autorise à la consulter. Omettre cette étape constitue un contrôle défaillant au niveau des objets : en modifiant le numéro dans l'URL, Alice pourrait lire les factures de tous les clients (une référence directe non sécurisée à un objet, ou IDOR).
4. **Le contrôle de l'action.** Consulter et supprimer sont des permissions différentes. Un système qui se contente de masquer le bouton "supprimer" tout en laissant le point d'accès de suppression appelable n'a filtré l'accès que côté client.
5. **La décision est journalisée au nom d'Alice.** C'est ce qui rend la [responsabilité](accountability.md) possible : une entrée de journal n'a de sens que si elle est liée à un principal authentifié de manière fiable.

Le scénario met aussi en évidence une défaillance plus discrète : **les permissions obsolètes**. Si Alice est passée de la facturation au marketing le trimestre dernier et que son ancien rôle n'a jamais été retiré, chacun des contrôles ci-dessus réussit encore, en appliquant correctement des règles qui ne devraient plus exister.

## Vérifier de nouveau à chaque requête

Les architectures modernes éloignent encore davantage ces deux étapes d'une connexion unique. Le [zero trust](zero-trust.md) traite chaque requête comme non fiable et revérifie l'identité et les permissions au lieu de se fier à l'emplacement réseau, et la [défense en profondeur](defense-in-depth.md) considère les contrôles d'identité et d'accès comme une couche parmi d'autres, afin qu'une seule défaillance n'expose pas tout le système. Notez que le mot "autorisation" a aussi un sens juridique et éthique, celui de la permission de tester un système ou même d'y accéder, traité dans [autorisation et consentement](../ethics-and-philosophy/authorization-and-consent.md).

## Références

- [NIST SP 800-63-4 Digital Identity Guidelines](https://pages.nist.gov/800-63-4/)
- [NIST Glossary: Authentication](https://csrc.nist.gov/glossary/term/authentication)
- [OWASP Top 10:2025, A01 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [MITRE CWE-862: Missing Authorization](https://cwe.mitre.org/data/definitions/862.html)
