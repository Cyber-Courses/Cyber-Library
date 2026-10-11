---
title: "Contournement d'authentification de l'API : failles d'authentification Zabbix propres à la version"
order: 5
description: "Certaines versions de Zabbix ont connu des failles de contournement d'authentification et de gestion de session dans le frontend et l'API qui permettent à un attaquant d'atteindre des fonctionnalités authentifiées, ou d'agir avec des droits élevés, sans identifiants valides. Elles incluent des bogues de traitement JSON-RPC et SAML/SSO ; faire correspondre la version à la faille fournit un accès qui s'enchaîne vers les fonctionnalités d'exécution de code de la plateforme."
keywords:
  - zabbix auth bypass
  - json-rpc
  - saml
  - session
  - version-specific
---

# Contournement d'authentification de l'API

Au-delà des identifiants faibles, Zabbix lui-même a livré des vulnérabilités de contournement d'authentification et de gestion de session, de sorte que contre une version affectée, un attaquant atteint des fonctionnalités authentifiées sans identifiants valides. Les catégories de failles incluent des bogues dans le traitement des requêtes JSON-RPC et du frontend qui exposaient des actions censées exiger une connexion, et des faiblesses dans la logique de validation SAML/SSO et de session qui permettaient qu'une requête soit traitée comme celle d'un utilisateur authentifié (parfois privilégié). L'impact, c'est l'accès à l'API/UI, qui s'enchaîne ensuite vers les fonctionnalités d'exécution de code propres à la plateforme (scripts, éléments) ou vers une exploitation directe. Ces failles sont propres à la version, de sorte que la méthode consiste à identifier le build et à faire correspondre l'avis.

```bash
# fingerprint first (the applicable bypass is version-specific)
curl -sk https://<target>/zabbix/api_jsonrpc.php -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
# the bypass itself is specific to the version/flaw (e.g. crafted JSON-RPC calls that
# reach protected methods, or SAML/session handling that yields an authenticated session);
# match the Zabbix build to the advisory for the exact request.
```

## Notes d'exploitation

- Le point d'entrée est propre à la version : `apiinfo.version` fournit le build sans authentification, ce qui correspond à la faille de contournement d'authentification ou de session applicable ; il n'existe pas de contournement universel unique.
- Les bogues de traitement SAML/SSO sont notables car ils peuvent fournir une session authentifiée, parfois administrateur ; là où le SSO est configuré, évaluez-le au regard de la version.
- Un contournement qui atteint l'API vaut une connexion pour ce qui suit : énumérez les hôtes/macros et pilotez l'[exécution de code](../code-execution/index.md) via des scripts ou des éléments.
- Lorsque aucun contournement ne s'applique, repliez-vous sur les identifiants ([par défaut](default-credentials.md)/[faibles](weak-passwords.md)) et le [vol de session](session-and-token-theft.md).

## Références

- [Avis de sécurité Zabbix](https://support.zabbix.com/browse/ZBX)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
