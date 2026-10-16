---
title: "Authentification et autorisation : prouver son identité, puis accorder l'accès"
description: "Les deux processus de sécurité associés qui établissent d'abord qui est un principal, puis décident ce que ce principal est autorisé à faire."
keywords:
  - authentification et autorisation
  - facteurs d'authentification
  - contrôle d'accès
  - vérification d'identité
  - contrôle d'accès défaillant
  - député confus
---

# Authentification et autorisation

**L'authentification et l'autorisation sont deux décisions distinctes qui se produisent l'une après l'autre, et les confondre est l'une des plus anciennes sources de bogues de sécurité.** L'authentification répond à la question "qui êtes-vous ?" en vérifiant qu'un principal (un utilisateur, un service ou un appareil) est bien celui qu'il prétend être. L'autorisation répond à la question "qu'avez-vous le droit de faire ?" en décidant si ce principal vérifié peut effectuer une action précise sur une ressource précise. La première établit l'identité ; la seconde conditionne l'accès.

## Deux questions, posées dans l'ordre

Pour tout ce qui est conditionné par l'identité, l'ordre est fixe : un système doit savoir qui est un principal avant de pouvoir raisonnablement décider ce que ce principal peut faire, c'est pourquoi l'authentification vient en premier et l'autorisation suit. Tout n'est pas conditionné ainsi, cependant. Une page publique peut être lue sans aucune identité, et une requête peut être refusée d'emblée sans authentifier qui que ce soit ; ce n'est que lorsque l'accès dépend de qui vous êtes que l'authentification doit venir en premier. Les deux sont souvent abrégées en "authN" et "authZ" pour les garder distinctes à l'oral comme dans le code.

Un test utile : l'authentification demande si vous êtes bien celui que vous prétendez être, et la vérification est la même quoi que vous fassiez ensuite ; l'autorisation demande si votre identité établie peut effectuer une action précise sur une ressource précise, de sorte que sa réponse varie selon la ressource, l'action et souvent l'identité elle-même. Décider qu'Alice peut exporter la paie alors que Bob ne le peut pas relève de l'autorisation, et non de l'authentification, même si le résultat change selon qui s'est connecté. De même, une connexion valide qui permet ensuite d'atteindre les dossiers de quelqu'un d'autre est une défaillance d'autorisation, et non d'authentification, même si les deux se sont produites dans la même requête.

## Les trois facteurs de preuve

L'authentification repose sur des preuves que le demandeur peut produire. Elles se répartissent en trois catégories classiques, et c'est la combinaison de ces catégories qui rend l'authentification multifacteur robuste.

| Facteur | Ce que c'est | Exemples |
|---|---|---|
| Quelque chose que vous savez | Un secret conservé en mémoire | Mot de passe, code PIN, phrase secrète |
| Quelque chose que vous possédez | Un jeton physique ou cryptographique | Clé de sécurité, application sur téléphone, carte à puce |
| Quelque chose que vous êtes | Un trait biométrique | Empreinte digitale, visage, iris |

Deux mots de passe ne constituent pas une authentification multifacteur, car ils relèvent de la même catégorie et tombent sous le même vol. Un mot de passe plus une clé matérielle résiste à un attaquant qui n'a fait qu'hameçonner le mot de passe.

## Décomposer une connexion unique

Prenons l'exemple d'un utilisateur qui ouvre un tableau de bord interne :

1. **Revendication.** Le navigateur présente un nom d'utilisateur et demande à se connecter.
2. **Authentifier.** Le serveur vérifie l'empreinte du mot de passe, puis demande un code à usage unique provenant du téléphone de l'utilisateur. Les deux facteurs sont validés, l'identité est donc établie.
3. **Émettre une preuve.** Le serveur renvoie un jeton de session signé qui encode l'identité vérifiée.
4. **Autoriser.** Lorsque l'utilisateur clique sur "exporter la paie", le serveur lit l'identité contenue dans le jeton, recherche les rôles de cet utilisateur et ne trouve aucune permission liée aux finances. La requête est refusée.

Les étapes 1 à 3 relèvent de l'authentification ; l'étape 4 relève de l'autorisation. Un système peut gérer parfaitement la connexion et malgré tout divulguer des données si l'étape 4 est faible.

## Le député confus et le contrôle d'accès défaillant

Les défaillances les plus fréquentes se situent dans l'autorisation. Le **contrôle d'accès défaillant** signifie que le serveur fait confiance au client pour rester dans son couloir : modifier un identifiant dans une URL permet d'atteindre les données d'un autre compte parce que le serveur a authentifié la session mais n'a jamais revérifié la propriété. Le **député confus** est plus subtil : un programme de confiance est manipulé pour utiliser ses propres privilèges plus élevés au profit d'un appelant moins privilégié, parce qu'il agit selon sa propre autorité au lieu de reprendre et de restreindre celle de l'appelant. Les deux diffèrent par leur cause, le contrôle d'accès défaillant ne revérifie pas une session tandis que le député confus ne transmet pas l'autorité limitée de l'appelant, mais tous deux laissent passer une action que les permissions propres au demandeur auraient refusée.

## Où cela rejoint les autres principes

L'autorisation est le point où le [moindre privilège](least-privilege.md) est réellement appliqué, requête par requête. Le [zéro confiance](zero-trust.md) refuse de considérer une connexion réussie comme une permission durable et revérifie chaque requête à la place. Et parce que l'authentification lie les actions à une identité nommée, elle constitue le fondement de la [responsabilité](accountability.md) : les journaux n'ont de sens que lorsque vous pouvez faire confiance à l'information sur qui a effectué chaque action.

## Références

- [NIST SP 800-63B, Digital Identity Guidelines: Authentication](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [OWASP Top 10: A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [NIST Glossary: Authentication](https://csrc.nist.gov/glossary/term/authentication)
- [MITRE CWE-862: Missing Authorization](https://cwe.mitre.org/data/definitions/862.html)
