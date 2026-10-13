---
title: "Matrix : attaquer les homeservers via les API client-serveur et de fédération"
order: 3
description: "Les homeservers Matrix (Synapse, Dendrite, Conduit) exposent une API client-serveur HTTP/JSON sur 8008 ou 443 et une API de fédération sur 8448, la délégation .well-known pointant les clients et les serveurs vers l'hôte réel. La surface comprend le sondage non authentifié de version et d'inscription, l'inscription ouverte et l'inscription admin par secret partagé, la SSRF du dépôt de médias, et l'abus de requêtes de fédération."
keywords:
  - Matrix
  - homeserver
  - Synapse
  - federation
  - client-server API
---

# Matrix

Matrix est un protocole de messagerie fédéré parlé en HTTP avec des corps JSON. Un homeserver (Synapse est l'implémentation de référence ; Dendrite et Conduit sont des alternatives) sert l'API client-serveur sur TCP 8008 derrière un reverse proxy sur 443, et l'API de fédération serveur à serveur sur 8448. Les clients et les serveurs fédérés trouvent l'hôte réel via les fichiers de délégation `.well-known`. Comme tout est en HTTP, toute la surface est atteignable avec `curl` : vous pouvez identifier le serveur, sonder si l'inscription est ouverte, tester l'API admin de Synapse, rechercher dans les annuaires d'utilisateurs et de salons, et interroger des profils via la fédération, le tout sans identifiants.

## Triage

```bash
# fingerprint and delegation, no auth needed
curl -s https://target.lan/_matrix/client/versions          # supported spec versions + "unstable_features"
curl -s https://target.lan/.well-known/matrix/server         # federation delegation target:port
curl -s https://target.lan/.well-known/matrix/client         # client API base_url
curl -s https://target.lan:8448/_matrix/federation/v1/version
# the federation version body names the software, e.g. {"server":{"name":"Synapse","version":"1.xx.x"}}
```

Le corps de `/_matrix/federation/v1/version` nomme Synapse, Dendrite ou Conduit et sa version, ce qui décide des failles serveur qui s'appliquent. Les fichiers `.well-known` révèlent l'hôte et le port réels du backend lorsque le service est placé derrière un proxy.

## Questions de triage

- L'inscription est-elle ouverte (`/_matrix/client/v3/register`), et l'accès invité est-il activé ? Cela pilote l'[authentification](authentication.md).
- L'API admin de Synapse est-elle atteignable, et le déploiement utilise-t-il une inscription par secret partagé dont vous pouvez abuser pour créer un administrateur ? Également l'[authentification](authentication.md).
- Pouvez-vous rechercher dans les annuaires d'utilisateurs et de salons et interroger des profils via la fédération sans compte ? Cela pilote l'[énumération](enumeration.md).
- De quel homeserver et de quelle version s'agit-il, et est-ce une version de Synapse présentant la SSRF de prévisualisation d'URL du dépôt de médias ? Cela pilote l'[exploitation serveur](server-exploitation.md).

## Sous-thèmes

- **[Énumération](enumeration.md)** : délégation `.well-known`, sondage de l'inscription et de l'API admin, recherche dans l'annuaire des utilisateurs et des salons publics, et requêtes de profil de fédération.
- **[Authentification](authentication.md)** : inscription ouverte, accès invité, flux SSO/jeton, et abus de l'inscription par secret partagé de Synapse pour créer un compte administrateur.
- **[Exploitation serveur](server-exploitation.md)** : SSRF de prévisualisation d'URL du dépôt de médias de Synapse vers des services internes et les métadonnées cloud, falsification de requêtes de fédération et épuisement des ressources, et bogues d'analyse de Dendrite/Conduit.

## Références

- [Matrix specification](https://spec.matrix.org/latest/)
- [Matrix client-server API](https://spec.matrix.org/latest/client-server-api/)
- [Synapse homeserver](https://github.com/element-hq/synapse)
