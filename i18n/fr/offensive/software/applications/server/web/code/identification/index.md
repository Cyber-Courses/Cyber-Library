---
order: 2
title: "Identification : découverte de comptes, énumération et exposition d'identifiants dans les applications web"
description: "Comment le comportement d'une application révèle quels comptes existent, facilite le credential stuffing et expose les identifiants utilisateur, la reconnaissance pré-authentification qui alimente la prise de contrôle de compte et les attaques de contrôle d'accès."
keywords:
  - énumération de comptes
  - énumération d'utilisateurs
  - credential stuffing
  - exposition d'identifiants
  - reconnaissance IDOR
---

# Identification

L'**identification** est la couche de reconnaissance sous l'authentification : avant d'essayer de *devenir* un utilisateur, vous déterminez *quels* utilisateurs existent, *quels* identifiants valent la peine d'être essayés, et *quels* identifiants l'application vous fournit gratuitement. Rien de tout cela ne défait un identifiant à lui seul, mais c'est ce qui transforme une devinette à l'aveugle en une attaque ciblée. Une liste de noms d'utilisateur confirmés plus une limite de débit faible constituent une campagne de credential stuffing ; une séquence d'IDs utilisateur divulguée est l'entrée de chaque test de contrôle d'accès qui suit.

Ce domaine porte sur les *signaux émis par l'application*, pas sur les contrôles d'autorisation qu'elle impose. Il se situe entre l'[Authentification](../authentication/index.md) (prouver l'identité) et le Contrôle d'accès (ce qu'une identité peut faire), et il alimente les deux.

## Là où ça casse

Trois faiblesses récurrentes donnent à un attaquant la reconnaissance dont il a besoin :

- **Des oracles qui distinguent le vrai du faux.** Une connexion qui dit « utilisateur inconnu » pour un identifiant et « mauvais mot de passe » pour un autre est un oracle d'énumération. Le même oracle se cache dans l'inscription, la réinitialisation de mot de passe, les invites MFA et même le timing de réponse.
- **Un throttling qui ne throttle pas vraiment.** Les limites uniquement par IP tombent face à la rotation d'en-têtes et aux proxies ; les limites par compte tombent face au password spraying ; les endpoints d'API, GraphQL et mobiles sont souvent exemptés des limites que la connexion web impose.
- **Des identifiants distribués librement.** Les IDs séquentiels dans les URLs, les champs internes dans les réponses d'API et les exports, et les endpoints de profil public ou d'autocomplétion divulguent les valeurs mêmes dont un attaquant a besoin pour énumérer les comptes et cibler les objets.

## Méthodologie d'attaque

1. **Collectez les identifiants candidats.** Récupérez les noms d'utilisateur et les e-mails depuis les profils publics, les endpoints d'autocomplétion et de recherche, les exports, les réponses d'API et les corpus de fuites.
2. **Trouvez un oracle d'existence.** Comparez les réponses pour un identifiant connu valide et un connu invalide à travers la connexion, l'inscription, la réinitialisation et le SSO, en gardant la forme de la requête identique, et mesurez le timing quand les corps correspondent.
3. **Confirmez la liste de comptes.** Automatisez l'oracle contre votre liste de candidats pour produire un ensemble de comptes confirmés.
4. **Mesurez le throttle.** Déterminez sur quoi la limite de débit se base (IP, compte, session, rien) et si les chemins API/GraphQL/mobiles la partagent.
5. **Alimentez l'étape suivante.** Transmettez les comptes confirmés au credential stuffing et au password spraying, et transmettez les identifiants divulgués au test de contrôle d'accès.

L'énumération et le stuffing touchent de vrais comptes ; restez donc dans le périmètre et évitez de verrouiller des utilisateurs authentiques.

## Pages

- **[Énumération de comptes](account-enumeration.md)** : oracles d'existence dans les messages d'erreur, les codes de statut, la forme de réponse, les redirections et le timing, à travers la connexion, l'inscription, la réinitialisation et le SSO.
- **[Limites de débit et automatisation](rate-limits-and-automation.md)** : lacunes de throttling, clé par IP contre par compte, batching GraphQL/JSON, password spraying, et l'outillage qui les pilote.
- **[Exposition d'identifiants dans les URLs et les exports](identifier-exposure-in-urls-and-exports.md)** : IDs séquentiels et prévisibles, API et exports trop bavards, et endpoints publics qui divulguent les identifiants utilisateur.
