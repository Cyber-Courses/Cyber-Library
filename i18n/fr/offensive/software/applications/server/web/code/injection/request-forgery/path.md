---
order: 2
title: "Traversée de chemin et confusion de parseur dans les URL SSRF"
description: "Lorsque l'hôte est fixe mais que le chemin est contrôlé par l'utilisateur, la traversée atteint d'autres endpoints sur l'hôte interne, et les différentiels de parseur authority/path déplacent la cible effective au-delà d'une allowlist de préfixe."
keywords:
  - traversée de chemin SSRF
  - confusion de parseur d'URL
  - contournement d'allowlist de préfixe
  - authority par backslash
  - hôte par arobase
  - troncature par fragment
---

# Path

Lorsqu'un sink de fetch fixe l'hôte et ne laisse l'attaquant influencer que le chemin, deux voies s'ouvrent : la traversée qui échappe à un chemin de base contraint pour atteindre d'autres endpoints sur le même hôte interne, et les différentiels de parseur où des caractères en position de chemin changent quelle authority la requête cible réellement.

## Échapper à un chemin de base fixe

Un sink qui construit `http://internal-api/v1/users/<input>` contraint l'attaquant à un sous-arbre, mais `../` en sort pour atteindre n'importe quel endpoint sur le même hôte :

```
../../admin/config
..%2f..%2fadmin%2fconfig        # encoded, to pass a literal ../ filter
..%252f..%252fadmin             # double-encoded, where one decode happens before the check
```

Cela ne change pas l'hôte, c'est donc utile lorsque l'hôte interne est déjà la cible et que l'objectif est d'atteindre un chemin sensible (une route admin, un endpoint actuator ou debug, une API interne non authentifiée) que le chemin de base prévu cachait.

## Contournement d'allowlist de préfixe

Un contrôle courant vérifie que l'URL assemblée commence par un préfixe autorisé, par exemple `https://storage.example.com/`. La traversée et la normalisation déjouent une correspondance de préfixe naïve lorsque le client normalise le chemin après la vérification :

```
https://storage.example.com/../../internal/secret
https://storage.example.com/..%2f..%2finternal%2fsecret
```

La chaîne passe `startsWith`, et le client réduit les segments `../` pour atteindre un chemin différent de ce que le préfixe impliquait.

## Différentiels de parseur authority et path

Les parseurs d'URL sont en désaccord sur l'endroit où l'authority se termine et où le chemin commence, et ce désaccord laisse une valeur qu'un composant lit comme un chemin rediriger l'hôte auquel un autre composant se connecte. Les caractères qui pilotent cela sont `@`, `#`, `?` et le backslash :

```
http://allowed.example@127.0.0.1/          # userinfo trick: host is 127.0.0.1, not allowed.example
http://127.0.0.1#@allowed.example/         # fragment: real host is 127.0.0.1 before the #
http://127.0.0.1\@allowed.example/         # backslash: some parsers treat \ as /
http://allowed.example\@127.0.0.1/         # the reverse, depending on which parser validates
```

L'attaque est un différentiel : le validateur parse l'URL d'une façon (voyant `allowed.example` comme l'hôte) tandis que le client HTTP la parse d'une autre (se connectant à `127.0.0.1`). Les parseurs de style WHATWG traitent le backslash comme un séparateur de chemin et résolvent le userinfo différemment des anciens parseurs RFC 3986, donc un sink qui valide avec une bibliothèque et récupère avec une autre est la combinaison vulnérable. Le payload qui fonctionne dépend de la paire exacte, donc parcourez les formes `@`, `#` et `\` et observez quel hôte la requête atteint.

## Outils

- **Burp Repeater** : parcourir les payloads de traversée encodée et d'authority `@`/`#`/`\` contre le sink.
- **curl** : sonder les formes brutes de chemin et d'authority pour voir quel hôte la requête atteint.
- **SSRFmap** : automatiser la livraison de payloads d'allowlist de préfixe et de traversée.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
