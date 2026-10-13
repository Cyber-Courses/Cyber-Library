---
title: "Pollution de paramètres HTTP pour contourner une allowlist SSRF"
description: "Fournir le paramètre cible deux fois exploite un désaccord entre le composant qui valide l'URL et celui qui la récupère sur l'occurrence dupliquée qui l'emporte."
keywords:
  - pollution de paramètres HTTP
  - paramètre dupliqué
  - last parameter wins
  - contournement d'allowlist SSRF
  - url=allowed&url=internal
---

# Pollution de paramètres

Lorsque la cible du fetch provient d'un paramètre de requête, fournir ce paramètre deux fois peut séparer le validateur et le récupérateur. Si le code qui vérifie l'allowlist lit une occurrence et le code qui émet la requête lit l'autre, une seule requête transporte à la fois une valeur autorisée (pour la vérification) et une valeur interne (pour le fetch).

## Last-wins versus first-wins

Les plateformes sont en désaccord sur l'occurrence dupliquée d'un paramètre répété qu'une API donnée renvoie. Fournir à la fois une valeur autorisée et une valeur interne exploite ce désaccord :

```
?url=https://allowed.example&url=http://169.254.169.254/latest/meta-data/
```

Si le validateur lit le premier `url` (autorisé) et la couche HTTP lit le dernier `url` (interne), la requête passe la vérification et récupère l'endpoint de métadonnées. L'ordre inverse fonctionne contre un récupérateur first-wins associé à un validateur last-wins :

```
?url=http://127.0.0.1/&url=https://allowed.example
```

La condition vulnérable est simplement que les deux étapes résolvent les doublons différemment. Fractionnements courants : une étape utilise un helper de framework qui renvoie la première valeur tandis qu'une autre découpe la query string brute et prend la dernière, ou un proxy et une application sont en désaccord.

## Délimiteurs encodés et imbriqués

Lorsqu'un seul paramètre est parsé en sous-valeurs, un délimiteur encodé fait passer en contrebande une seconde valeur au-delà de la vérification. Un `;` ou un `&` encodé à l'intérieur de la valeur peut introduire un second paramètre après validation :

```
?url=https://allowed.example%26url=http://127.0.0.1/
?url=https://allowed.example%3Burl=http://127.0.0.1/
```

Le validateur voit une valeur opaque ; un décodage ultérieur la découpe en deux, et l'interne est sélectionnée.

## Confirmer le fractionnement

Envoyez le paramètre dupliqué avec une valeur pointant vers un listener contrôlé par l'attaquant et l'autre vers un hôte autorisé, et observez laquelle reçoit la connexion. Cela identifie si le récupérateur prend la première ou la dernière, ce qui fixe l'ordre pour la vraie cible interne. La valeur interne sélectionnée utilise encore les encodages d'[Authority](../authority/index.md) lorsque sa forme littérale est bloquée, et s'associe naturellement avec une [redirection](bypassing-using-a-redirect.md) lorsqu'une seule URL interne propre est nécessaire.

## Outils

- **Burp Repeater** : envoyer le paramètre deux fois et réordonner les doublons pour trouver quelle occurrence le récupérateur utilise.
- **curl** : émettre des paramètres bruts dupliqués et à délimiteur encodé pour sonder le comportement first-wins versus last-wins.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [OWASP: Testing for HTTP Parameter Pollution](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution)
