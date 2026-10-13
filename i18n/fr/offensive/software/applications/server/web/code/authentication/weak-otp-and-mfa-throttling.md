---
title: "Attaquer l'OTP et la MFA : brute force des codes, contournement de limite de débit et failles logiques de 2FA"
description: "Comment un pentester défait l'authentification à second facteur : brute force de codes faibles, contournement des limites de débit, saut de l'étape MFA et abus de codes non liés à l'utilisateur."
keywords:
  - contournement de MFA
  - brute force d'OTP
  - 2FA
  - limitation de débit
  - mot de passe à usage unique
  - authentification
---

# OTP faible et throttling de MFA

Des faiblesses au niveau applicatif dans l'authentification à second facteur qui permettent à un attaquant détenant (ou falsifiant) le premier facteur d'atteindre une session pleinement authentifiée. Les failles se répartissent en trois familles : le **brute force** de codes à usage unique faibles, le **contournement** des limites de débit censées l'empêcher, et les **failles logiques** où l'étape MFA peut être sautée ou le code n'est pas lié au bon utilisateur.

## Vue d'ensemble

Le flux canonique est en deux étapes : l'étape un valide `username:password` ; l'étape deux valide un second facteur (un TOTP d'une application d'authentification, un OTP par SMS/e-mail, une approbation push, ou un code de secours). L'invariant critique pour la sécurité est que **l'étape deux doit être liée de manière étatique à l'utilisateur exact qui a passé l'étape un, et la session post-MFA doit être inutilisable tant que l'étape deux n'a pas réussi.**

PortSwigger formule la faille de conception centrale : lorsque l'utilisateur saisit un mot de passe puis se voit demander un code sur une page distincte, il est de fait « connecté » avant que le code ne soit vérifié. Un serveur qui émet une session identifiant l'utilisateur en attente *avant* de vérifier le second facteur est la cause racine de la plupart des ruptures ci-dessous. L'espace de clés l'aggrave : un code à 4 chiffres a 10 000 valeurs, un code à 6 chiffres environ 1 000 000, donc sans throttling côté serveur le second facteur dégénère en un défi résoluble par brute force.

## Exploitation

### Brute force de l'OTP

Avec un code numérique court et aucun plafond de tentatives imposé, énumérez l'espace de clés contre l'endpoint de vérification. OWASP avertit que les applications acceptent souvent des codes d'une fenêtre de part et d'autre du TOTP courant (précédent/courant/suivant), multipliant vos chances ; si plusieurs codes sont simultanément valides, des tentatives soutenues atteignent une forte probabilité de succès en quelques heures.

Une requête de vérification représentative à piloter avec Burp Intruder :

```
POST /login2 HTTP/1.1
Cookie: session=<post-password session>
Content-Type: application/x-www-form-urlencoded

mfa-code=0000
```

Placez la position du payload sur `mfa-code`, générez `0000-9999`, et utilisez une **macro** de gestion de session pour rejouer les requêtes de connexion précédentes afin que chaque tentative porte une session valide fraîche (les applications qui vous déconnectent après deux codes erronés sont défaites ainsi, car la protection est cosmétique). Le succès est typiquement un HTTP **302** vers la page du compte.

### Contourner la limite de débit

Lorsqu'un limiteur existe, attaquez sa clé plutôt que le code :

- **Faites tourner l'identité du client** si le limiteur est basé sur l'IP : usurpez `X-Forwarded-For` (aussi `X-Real-IP`, `X-Client-IP`, `Forwarded`) à chaque requête ; testez des en-têtes non standard (`X-Debug`) qui peuvent désactiver entièrement la MFA.
- **Réinitialisez le compteur** : redemandez un code frais, ou changez le paramètre `username`/compte pour que le compteur se base sur une identité différente.
- **Altérez le paramètre de throttle** : lorsqu'un champ de requête porte l'état de la limite, envoyez-le vide ou null.
- **Concurrence / race** : tirez de nombreuses requêtes de vérification en parallèle (attaque single-packet de Turbo Intruder) pour qu'elles soient vérifiées avant que le compteur n'incrémente.

### Sauter l'étape MFA (failles logiques)

Parce que la session peut déjà être « connectée », naviguez de force directement vers un endpoint post-MFA (`GET /my-account`) et voyez s'il se charge sans la seconde étape. Altérez la logique client pilotée par la réponse (`"verified":false` en `true`), guettez les indices de code de statut (302 contre 200), ou routez vers un flux sans MFA (OWASP cite le changement d'une policy Azure AD B2C `B2C_1_SignInWithMFA` en `B2C_1_SignIn`).

### Code non lié à l'utilisateur

L'exemple phare de PortSwigger : l'étape un définit `Set-Cookie: account=carlos`, et la requête de vérification fait confiance à ce cookie (ou à un paramètre de corps `verify`) pour décider *de qui* le code est vérifié. Changez-le en une victime et vous générez et validez un code contre un compte comme lequel vous ne vous êtes jamais authentifié :

```
POST /login-steps/second HTTP/1.1
Cookie: account=victim-user

verify=victim-user&mfa-code=1234
```

Failles logiques connexes : réutilisation de code (pas d'usage unique), absence d'expiration (codes périmés acceptés), et codes valides entre utilisateurs ou sessions.

### Jetons autour de la MFA

- **OTP divulgué** : inspectez le corps/les en-têtes de la réponse d'émission ; les codes sont parfois renvoyés pour le débogage.
- **Codes de secours / de récupération** : souvent à plus longue durée de vie et faiblement limités ; brute-forcez-les ou rejouez-les et vérifiez l'usage unique.
- **Cookies « se souvenir de l'appareil » / rester connecté** : s'ils sont forgeables, ils contournent la MFA. Le cookie du lab de PortSwigger est `base64(username + ':' + md5(password))`, un MD5 non salé du mot de passe qui peut être brute-forcé *hors ligne*.
- **SMS / SIM-swap** : l'OTP par SMS est vulnérable à l'interception SS7 et au SIM-swap ; testez aussi le changement d'un paramètre de numéro de téléphone pour que le code soit livré à un numéro contrôlé par l'attaquant.

## Outils

- **Burp Suite** : Intruder (brute force de code), Turbo Intruder (race/concurrence), macros de gestion de session, et l'extension Param Miner pour la découverte d'en-têtes.

## Références

- PortSwigger : [Vulnerabilities in multi-factor authentication](https://portswigger.net/web-security/authentication/multi-factor) et [other authentication mechanisms](https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie)
- OWASP WSTG : [Testing Multi-Factor Authentication (WSTG-ATHN-11)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/11-Testing_Multi-Factor_Authentication) et [Weak Lock Out Mechanism (WSTG-ATHN-03)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism)
- OWASP : [Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
