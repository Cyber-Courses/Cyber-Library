---
title: "Énumération"
order: 1
description: "Récolter un serveur Rocket.Chat via l'API REST et DDP : lire settings.public sans authentification, paginer /api/v1/users.list et /api/v1/channels.list, extraire l'annuaire via /api/v1/spotlight, et appeler des méthodes DDP Meteor via le websocket. Exemple concret de curl avec les en-têtes X-Auth-Token et X-User-Id et interprétation du JSON, la version étant lue depuis /api/info."
keywords:
  - rocket.chat enumeration
  - users.list
  - spotlight
  - settings.public
  - meteor ddp
---

# Énumération

L'énumération de Rocket.Chat se divise entre ce qui est lisible sans identifiant (la version et les paramètres du serveur) et ce qui est lisible avec n'importe quel jeton authentifié (l'annuaire des utilisateurs et des canaux). Les endpoints de listage REST appliquent les permissions de manière incohérente selon les builds, et `spotlight` en particulier est un endpoint de recherche accessible par les utilisateurs ordinaires qui renvoie les utilisateurs et les canaux correspondant à un terme, de sorte que même un jeton à faibles privilèges cartographie l'annuaire. La sortie ici alimente le contournement de connexion et la chaîne de jeton admin dans l'[exploitation du serveur](server-exploitation.md).

## Sans authentification : version et paramètres

`/api/info` donne la version ; `/api/v1/settings.public` renvoie les paramètres publics, qui vous indiquent si l'inscription est ouverte, quels fournisseurs OAuth sont câblés, et s'il existe des personnalisations d'iframe/de connexion :

```bash
curl -sk http://<target>:3000/api/info
#   {"version":"6.5.0","success":true}
curl -sk http://<target>:3000/api/v1/settings.public | python3 -c 'import sys,json
for s in json.load(sys.stdin)["settings"]:
  if s["_id"] in ("Accounts_RegistrationForm","Accounts_RegistrationForm_SecretURL","LDAP_Enable","Accounts_OAuth_Custom"): print(s["_id"],"=",s.get("value"))'
```

```text
Accounts_RegistrationForm = Public      <- open registration: mint your own account
LDAP_Enable = false
```

`Accounts_RegistrationForm = Public` signifie que vous pouvez vous inscrire via `POST /api/v1/users.register` et obtenir un jeton sans invitation. Si c'est `Secret URL`, l'inscription nécessite un jeton que vous n'avez pas ; repliez-vous sur le contournement de connexion dans l'[authentification](authentication.md).

## Obtenir un jeton

Avec l'inscription ouverte :

```http
POST /api/v1/users.register HTTP/1.1
Host: target:3000
Content-Type: application/json

{"username":"a1","email":"a@evil.test","pass":"Sprayed-Passw0rd!","name":"a1"}
```

Ensuite, `POST /api/v1/login` renvoie la paire d'identifiants utilisée sur chaque appel ci-dessous :

```text
{"status":"success","data":{"authToken":"xox...","userId":"aBcD..."}}
```

Rocket.Chat authentifie les appels REST avec deux en-têtes ensemble : `X-Auth-Token: <authToken>` et `X-User-Id: <userId>`.

## Paginer les utilisateurs et les canaux

```bash
AUTH='-H X-Auth-Token:<authToken> -H X-User-Id:<userId>'
curl -sk $AUTH "http://<target>:3000/api/v1/users.list?count=100&offset=0" \
  | python3 -c 'import sys,json;[print(u["username"],u.get("emails"),u.get("roles")) for u in json.load(sys.stdin)["users"]]'
```

```text
admin     [{'address': 'root@corp.test'}]   ['admin']
j.doe     [{'address': 'j.doe@corp.test'}]  ['user']
```

Lisez `roles` : une entrée contenant `admin` marque les comptes à cibler avec le contournement de connexion ou à qui voler un jeton, car un jeton admin est la précondition du RCE par script d'intégration. `channels.list` cartographie les canaux publics ; `channels.list.joined` et `im.list` montrent ce que votre propre compte peut déjà lire :

```bash
curl -sk $AUTH "http://<target>:3000/api/v1/channels.list?count=100" | python3 -c 'import sys,json;[print(c["name"],c.get("t")) for c in json.load(sys.stdin)["channels"]]'
```

## Spotlight et annuaire

`spotlight` est la recherche client en direct et est accessible par n'importe quel utilisateur, renvoyant à la fois les utilisateurs et les salons pour un terme partiel, de sorte qu'il trouve les motifs de nommage privilégiés sans la permission de listage complet :

```http
GET /api/v1/spotlight?query=admin HTTP/1.1
Host: target:3000
X-Auth-Token: <authToken>
X-User-Id: <userId>
```

```text
{"users":[{"username":"admin","name":"Root"}],"rooms":[{"name":"ops-secrets","t":"p"}],"success":true}
```

Un salon avec `"t":"p"` est un groupe privé ; le voir nommé dans spotlight sans en être membre est l'amorce d'énumération pour le lire une fois que vous détenez un jeton admin.

## Méthodes DDP Meteor

Le même annuaire est accessible via le websocket en appelant des méthodes Meteor, ce qui est utile là où un endpoint REST est verrouillé par permission mais pas la méthode DDP. Connectez-vous à `/websocket` (ou `/sockjs/.../websocket`), envoyez la trame DDP `connect`, puis appelez une méthode telle que `getUsersOfRoom` ou `spotlight` :

```json
{"msg":"method","method":"spotlight","params":["admin"],"id":"1"}
```

La trame `result` transporte les mêmes données utilisateur/salon que l'appel REST. DDP expose aussi `listEmojiCustom`, `getRoomRoles`, et des flux d'abonnement qui divulguent l'appartenance.

## Suite

La liste des utilisateurs (noms d'utilisateur, e-mails, rôles `admin`) cadre le contournement de connexion NoSQL dans l'[authentification](authentication.md) ; les noms de salons privés issus de spotlight sont les cibles à lire une fois que le contournement produit un jeton admin dans l'[exploitation du serveur](server-exploitation.md).

## Outils

- [RocketChat/Rocket.Chat (source du serveur, pour confirmer les permissions des endpoints par build)](https://github.com/RocketChat/Rocket.Chat)
- Un client WebSocket (`wscat`, `websocat`) pour les appels de méthodes DDP.

## Références

- [Rocket.Chat API: users.list](https://developer.rocket.chat/apidocs/get-users-list)
- [Rocket.Chat API: spotlight](https://developer.rocket.chat/apidocs/spotlight)
- [Meteor DDP protocol specification](https://github.com/meteor/meteor/blob/devel/packages/ddp/DDP.md)
