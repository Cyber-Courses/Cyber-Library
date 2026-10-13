---
title: "Mattermost : attaquer le serveur REST /api/v4 et sa couche de plugins"
description: "Attaquer un serveur Mattermost auto-hébergé : un processus Go sur 8065 exposant l'API REST /api/v4 au-dessus de Postgres ou MySQL. Identifiez la version depuis /api/v4/system/ping et l'en-tête X-Version-Id, puis progressez à travers l'énumération des utilisateurs et des équipes, la couche d'authentification par connexion et jeton d'accès personnel, et l'exploitation serveur via IDOR d'API, injection SQL par paramètre de filtre, et upload de plugin Go vers l'exécution de code."
keywords:
  - mattermost
  - mattermost api
  - mattermost plugin rce
  - api v4
  - x-version-id
---

# Mattermost

Mattermost est un serveur Go qui sert un client React et effectue tout son travail via une seule API REST enracinée à `/api/v4`, écoutant sur `8065` par défaut (souvent placée derrière un reverse proxy vers 443), adossée à PostgreSQL ou MySQL. Parce que le binaire Go est un processus unique qui sert à la fois l'API et charge les plugins côté serveur, le chemin de grande valeur est : obtenir un jeton via la couche d'authentification, trouver un compte (ou un bogue) qui atteint le statut system-admin, puis téléverser un plugin Go qui s'exécute en tant que compte de service `mattermost`. Tout ce que fait le client correspond à un appel `/api/v4` documenté que vous pouvez rejouer avec `curl` et un en-tête `Authorization: Bearer`.

## Identifier d'abord

Confirmez le produit et cernez la version exacte, car les paramètres de filtre d'injection SQL et le comportement de signature de plugin dépendent de la version.

```bash
curl -sk http://<target>:8065/api/v4/system/ping
#   {"status":"OK"}                      confirms Mattermost
curl -skI http://<target>:8065/api/v4/users | grep -i x-version-id
#   X-Version-Id: 9.5.0.9.5.0.<hash>.true    major.minor.patch + build hash + "enterprise" flag
curl -sk 'http://<target>:8065/api/v4/config/client?format=old' | python3 -m json.tool | grep -iE 'version|buildnumber|enablesignup|enableopenserver'
```

Le `config/client?format=old` non authentifié est l'empreinte la plus riche à elle seule : il renvoie la configuration client complète, y compris `Version`, si l'inscription ouverte et l'enregistrement de serveur ouvert sont activés (`EnableOpenServer`, `EnableUserCreation`), les fournisseurs SSO câblés, et le `SiteURL` configuré. Le `.true`/`.false` final dans `X-Version-Id` vous indique s'il s'agit de la version Enterprise, ce qui change quels points de terminaison existent. Enregistrez tout cela avant de vous authentifier.

## Pages

- **[Énumération](enumeration.md)** : moissonner les utilisateurs, équipes et salons via `/api/v4/users`, `/api/v4/users/search` et `/api/v4/teams`, confirmer l'existence d'un compte via `/api/v4/users/email/{email}` et les différences d'erreurs de connexion, et lire `X-Version-Id` pour cadrer l'exploitation ultérieure.
- **[Authentification](authentication.md)** : le flux `POST /api/v4/users/login` qui renvoie une session dans l'en-tête `Token` et le cookie `MMAUTHTOKEN`, le password spraying, les jetons d'accès personnels, les failles SSO et MFA, et le point de terminaison de réinitialisation de mot de passe.
- **[Exploitation serveur](server-exploitation.md)** : failles d'autorisation d'API et IDOR entre équipes, injection SQL dans les paramètres de filtre d'API, path traversal dans les points de terminaison de fichiers et d'import, et l'upload de plugin system-admin (`/api/v4/plugins`) qui exécute un plugin Go sur le serveur.

## Références

- [Mattermost API reference (`/api/v4`)](https://api.mattermost.com/)
- [Mattermost plugin developer documentation](https://developers.mattermost.com/integrate/plugins/)
- [mattermost/mattermost (server source)](https://github.com/mattermost/mattermost)
- [Mattermost security updates](https://mattermost.com/security-updates/)
