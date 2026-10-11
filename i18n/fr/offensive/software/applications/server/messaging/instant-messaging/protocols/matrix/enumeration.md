---
title: "Énumération : cartographier un homeserver via les API HTTP de Matrix"
description: "Les API client-serveur et de fédération de Matrix répondent aux sondes non authentifiées avec curl : les fichiers .well-known révèlent le vrai backend, le point de terminaison register indique si l'inscription est ouverte, le chemin admin de Synapse révèle sa présence, et les points de terminaison de l'annuaire des utilisateurs et des salons publics listent les comptes et les salons. La requête de profil de fédération renvoie l'existence des utilisateurs et les noms d'affichage du point de vue d'un autre serveur."
keywords:
  - Matrix enumeration
  - well-known delegation
  - user directory
  - publicRooms
  - federation query
---

# Énumération

Tout dans Matrix est une requête HTTP, donc un homeserver s'énumère entièrement avec `curl`. Trois surfaces non authentifiées comptent : la délégation (`.well-known`) vous indique l'hôte et le port réels du backend derrière tout proxy ; les points de terminaison register et admin vous indiquent comment on peut entrer dans le serveur ; et les points de terminaison de l'annuaire et des requêtes de fédération listent les utilisateurs et les salons. La recherche d'annuaire et la liste des salons publics répondent souvent sans jeton, et la requête de profil de fédération vous permet de confirmer l'existence d'un utilisateur et de lire les noms d'affichage depuis l'extérieur du serveur entièrement.

## Prérequis

Une portée réseau vers 443/8008 (API client) et idéalement 8448 (fédération). Lisez d'abord `/_matrix/client/versions` pour confirmer qu'il s'agit d'un homeserver Matrix et pour voir quelles versions de spécification et fonctionnalités instables sont activées. Certains points de terminaison (la recherche d'annuaire en particulier) peuvent exiger un jeton sur les déploiements durcis ; s'ils renvoient `M_MISSING_TOKEN` ou `M_UNKNOWN_TOKEN`, provisionnez un compte via l'[authentification](authentication.md) et réessayez avec `Authorization: Bearer <token>`.

## Délégation et points d'entrée

```bash
# where the real server lives (proxy fronting)
curl -s https://target.lan/.well-known/matrix/server
# {"m.server":"matrix-internal.target.lan:8448"}  => the true federation backend

# is registration open? ask the register endpoint what flows it offers
curl -s -X POST https://target.lan/_matrix/client/v3/register \
  -H 'Content-Type: application/json' -d '{}'
# {"flows":[{"stages":["m.login.dummy"]}], "session":"..."}  => OPEN registration (dummy stage only)
# {"errcode":"M_FORBIDDEN","error":"Registration has been disabled"}  => closed

# is this Synapse with its admin API exposed?
curl -s -o /dev/null -w "%{http_code}\n" https://target.lan/_synapse/admin/v1/server_version
# 200 => Synapse admin API reachable; 404 => not Synapse or admin path not routed
```

Interprétation : une réponse `flows` ne contenant que `m.login.dummy` signifie que n'importe qui peut s'inscrire sans e-mail ni jeton (passez directement à l'[authentification](authentication.md)) ; une `session` avec une étape `m.login.recaptcha`/`m.login.email.identity` signifie que l'inscription est ouverte mais encadrée. Un `200` sur `/_synapse/admin/v1/server_version` confirme Synapse et que la surface admin est routée, ce qui est la cible de l'abus par secret partagé.

## Annuaires d'utilisateurs et de salons

```bash
# search the user directory (often token-gated; use a Bearer token if required)
curl -s -X POST https://target.lan/_matrix/client/v3/user_directory/search \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{"search_term":"a","limit":100}'
# returns {"results":[{"user_id":"@jdoe:target.lan","display_name":"John Doe"}], ...}

# list published rooms (frequently answers unauthenticated)
curl -s https://target.lan/_matrix/client/v3/publicRooms
# each chunk entry has room_id, name, topic, num_joined_members, and whether it is world-readable
```

Rechercher dans l'annuaire des lettres courantes énumère les comptes et les noms d'affichage, construisant l'inventaire des utilisateurs ; le bloc des salons publics nomme les salons, leurs sujets (qui fuitent des détails internes) et le nombre de membres, et signale les salons world-readable dont vous pouvez lire l'historique en les rejoignant.

## Requête de profil de fédération

```bash
# confirm a user exists and read their profile from another server's point of view
curl -s "https://target.lan:8448/_matrix/federation/v1/query/profile?user_id=@jdoe:target.lan&field=displayname"
# {"displayname":"John Doe"}  => the account exists
# {"errcode":"M_NOT_FOUND"}   => no such user
```

La requête de profil de fédération n'a besoin d'aucun compte local et confirme si un `@user:domain` donné existe sur le homeserver, de sorte qu'une liste de noms via ce point de terminaison énumère les comptes valides même lorsque l'annuaire côté client est verrouillé.

## Suite

- Un flux `register` ouvert est un identifiant immédiat ; passez à l'[authentification](authentication.md) pour créer un compte et, sur Synapse, un administrateur.
- Les identifiants `@user:domain` énumérés alimentent l'usurpation, le phishing par message direct, et l'API de gestion des utilisateurs admin une fois que vous détenez le statut admin.
- Les salons publics world-readable donnent un historique lisible sans les rejoindre ; moissonnez les sujets et les messages épinglés à la recherche de secrets.

## Références

- [Matrix client-server API (registration and directories)](https://spec.matrix.org/latest/client-server-api/)
- [Matrix server-server (federation) API](https://spec.matrix.org/latest/server-server-api/)
- [Synapse admin API documentation](https://element-hq.github.io/synapse/latest/usage/administration/admin_api/index.html)
