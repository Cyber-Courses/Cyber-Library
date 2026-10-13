---
order: 1
title: "Attaquer l'authentification dans le code des applications web : sessions, jetons, OAuth, SAML et MFA"
description: "Un guide offensif pour casser la façon dont le code côté serveur prouve l'identité (gestion des identifiants, fédération, MFA, sessions et jetons) avec la méthodologie de reconnaissance et d'attaque qui mène à la prise de contrôle de compte."
keywords:
  - attaques d'authentification
  - prise de contrôle de compte
  - gestion des sessions
  - JWT
  - OAuth
  - contournement de MFA
---

# Authentification

L'**authentification** est la façon dont le code côté serveur établit *qui* est l'appelant avant toute décision d'autorisation. Pour un attaquant, c'est la cible à plus forte valeur dans l'application : cassez-la et vous obtenez généralement d'emblée une **prise de contrôle de compte ou un accès administratif complet**, sans aucun chaînage supplémentaire nécessaire. Cette section couvre les faiblesses au niveau applicatif de cette logique (identifiants, connexions fédérées, contrôles multifacteurs, sessions et jetons) telles qu'implémentées dans le code de la cible et ses dépendances.

L'authentification répond à « qui êtes-vous ? » ; l'autorisation (contrôle d'accès) répond à « que pouvez-vous faire ? ». Elles se cassent différemment et s'attaquent différemment, et ce sous-arbre porte strictement sur la première. Lorsque vous pouvez *devenir* un autre utilisateur, vous avez rarement besoin de vaincre le contrôle d'accès.

## Là où ça casse (vos ouvertures)

Le code d'authentification est fragile parce qu'il assemble cryptographie, état et protocoles tiers. Les ouvertures récurrentes :

- **Des décisions de sécurité prises sur une entrée contrôlée par l'attaquant.** Les en-têtes de jeton (`alg`, `kid`), les paramètres de redirection et l'en-tête `Host` sont tous modifiables par le client, pourtant on leur fait fréquemment confiance *avant* que l'identité ne soit prouvée. C'est le filon le plus riche, traité dans [Confusion d'algorithme JWT](jwt-algorithm-confusion.md).
- **Des contrôles sautés dans des protocoles complexes.** OAuth, OIDC et SAML comportent de nombreux paramètres obligatoires mais d'apparence optionnelle. Les parties de confiance qui codent le chemin heureux abandonnent régulièrement la validation de `state`, `nonce`, de l'audience, de la signature ou de `redirect_uri`, chacun étant un point d'appui.
- **Des primitives faites maison.** Jetons de réinitialisation prévisibles, identifiants de session artisanaux et cookies « remember me » personnalisés réintroduisent des problèmes résolus que vous pouvez exploiter.
- **Absence de throttling et de cycle de vie.** Les endpoints de connexion, d'OTP et de réinitialisation sans limites de débit invitent au brute force ; les sessions qui ne tournent ni n'expirent jamais permettent la fixation et le rejeu.

## Méthodologie d'attaque

Travaillez une surface d'authentification dans cet ordre :

1. **Cartographiez chaque point d'entrée.** Connexion par mot de passe, SSO/connexion sociale, jetons d'API, réinitialisation de mot de passe, invitation de compte, « remember me », step-up/MFA, et tout endpoint hérité ou mobile. Chacun est une frontière de confiance distincte et une attaque distincte.
2. **Trouvez l'identifiant de confiance.** Déterminez exactement ce que le serveur vérifie à chaque requête (un cookie de session, un JWT, une assertion SAML) et où il est vérifié. Le vérificateur est votre cible première ; tout le reste n'est que reconnaissance vers lui.
3. **Attaquez le vérificateur.** Altérez le jeton ou l'assertion : échangez les algorithmes et les en-têtes de sélection de clé, retirez ou annulez les signatures, rejouez entre utilisateurs et sessions, et testez si l'émetteur, l'audience, l'expiration et la liaison sont réellement imposés.
4. **Attaquez les flux autour de lui.** Pour les connexions fédérées, visez la gestion de `redirect_uri`/`state`/`nonce` et les contrôles d'audience ; pour les réinitialisations et invitations, visez l'entropie du jeton, sa durée de vie, son usage unique et l'empoisonnement de l'en-tête Host ; pour la MFA, visez le throttling, l'ordonnancement des étapes et les chemins de repli.
5. **Escaladez et persistez.** Transformez un point d'appui en impact : forgez une identité d'administrateur, pivotez entre tenants, ou capturez des jetons à longue durée de vie. Confirmez si le changement de privilège et la déconnexion invalident réellement les sessions antérieures.

Les tests de prise de contrôle de compte manipulent de vraies identités ; limitez-les donc aux systèmes dans le périmètre et évitez de verrouiller ou de détruire des comptes authentiques pendant les travaux de brute force et de réinitialisation.

## Vecteurs d'attaque

- **[Confusion d'algorithme JWT](jwt-algorithm-confusion.md)** : forgez des JSON Web Tokens en abusant de l'en-tête `alg` contrôlé par l'attaquant (`none`, confusion de clé RS256 vers HS256) ou de la sélection de clé `jwk`/`jku`/`kid`, afin qu'un jeton aux claims choisis passe la vérification.
- **[Mauvaise configuration de redirection OAuth](oauth-redirect-misconfiguration.md)** : volez des codes d'autorisation ou des jetons via une validation faible de `redirect_uri` et l'absence de `state`/`nonce`, y compris le login CSRF contre la partie de confiance.
- **[Jetons de réinitialisation de mot de passe et d'invitation](password-reset-and-invite-tokens.md)** : prenez le contrôle de comptes via des liens de réinitialisation et d'invitation prévisibles, à longue durée de vie, réutilisables ou empoisonnés par l'en-tête Host.
- **[Fixation de session](session-fixation.md)** : plantez un identifiant de session connu qui n'est pas régénéré à la connexion, puis chevauchez la session authentifiée de la victime.
- **[OTP faible et throttling de MFA](weak-otp-and-mfa-throttling.md)** : brute-forcez ou contournez les seconds facteurs lorsque les codes à usage unique ont une faible entropie, aucune limitation de débit, ou des contrôles d'ordonnancement défectueux.
