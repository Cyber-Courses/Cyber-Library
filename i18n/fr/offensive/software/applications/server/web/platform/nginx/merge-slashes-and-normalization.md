---
order: 4
title: "merge_slashes et normalisation nginx : correspondance de location qui contourne les règles"
description: "Exploitation de la gestion des slashes et des chemins par nginx : merge_slashes off, slashes encodés, et correspondance de location par préfixe en désaccord avec le système de fichiers ou un amont, contournant les règles d'accès basées sur location."
keywords:
  - merge_slashes
  - normalisation nginx
  - slash encodé
  - correspondance de location
  - contournement de contrôle d'accès
---

# merge_slashes et normalisation

nginx décide quel `location` traite une requête à partir de l'URI normalisé, puis peut le mapper vers un fichier ou le transmettre en amont. Sa gestion des slashes et de l'encodage peut être réglée de façons qui laissent un chemin fabriqué esquiver une règle basée sur `location` ou atteindre un fichier que la règle entendait protéger.

## merge_slashes off

Par défaut, nginx réduit les slashes répétés, mais `merge_slashes off;` (réglé pour préserver les URL de certaines applications) change la correspondance. Une règle censée garder `/admin` peut être esquivée avec des slashes doublés quand la correspondance et l'amont divergent :

```
GET //admin/ HTTP/1.1
GET /admin//../admin/ HTTP/1.1
```

Avec la fusion désactivée, `//admin` peut ne pas correspondre à un `location = /admin` ou à une règle de préfixe de la même façon que le backend le résout, de sorte qu'une règle d'accès limitée à la forme à slash simple est contournée pendant que la ressource se résout quand même.

## Pièges de la location par préfixe

- Un préfixe `location /api` (sans regex, sans slash final) correspond à `/api`, `/api2` et `/api-internal`, ce qui peut exposer plus que prévu.
- Une location `internal;` protégeant une cible `X-Accel-Redirect` peut être atteinte directement si une autre règle mappe vers le même chemin.
- Mélanger des locations regex et préfixe avec des `proxy_pass`/`root` différents conduit à ce qu'une requête soit mise en correspondance par un bloc mais servie/transmise par un autre.

## Slashes encodés et segments de points

nginx normalise l'URI de requête avant la correspondance de `location` : il décode les `%XX`, résout les segments de points `.`/`..` et (sauf `merge_slashes off`) réduit les slashes répétés. Ainsi, une simple traversée encodée ne **survit pas** côté nginx pour sa propre correspondance, et `%2e%2e%2f` est résolu comme `../` avant qu'une règle ne soit choisie.

L'astuce du slash encodé appartient donc à un [décalage de normalisation](../reverse-proxy-and-edge/normalization-mismatch.md) de proxy, pas à la correspondance de nginx sur ses propres chemins de fichiers : elle fonctionne quand nginx transmet à une **origine qui décode différemment**. Le détail nginx pertinent est la façon dont `proxy_pass` passe l'URI : sans partie URI dans `proxy_pass`, nginx transmet l'URI de requête normalisé ; avec une partie URI, il transmet le chemin mis en correspondance et réécrit. Un décalage entre ce que nginx a normalisé et ce que l'amont redécode est l'endroit où un `..%2f` encodé atterrit à l'origine :

```
GET /public/..%2f..%2fadmin HTTP/1.1     # exploitable at the origin behind nginx, per the proxy page
```

## Exploitation

- Contre un chemin protégé par `location` (401/403 sur `/admin`, `/internal`), les gains côté nginx sont les slashes doublés (quand `merge_slashes off`) et les variantes à slash final ; un `200` sur la ressource protégée confirme une faille de correspondance.
- Énumérez le débordement de la correspondance par préfixe en sondant des chemins voisins qui partagent un préfixe protégé.
- Les slashes encodés et les segments de points sont normalisés par nginx lui-même, donc gardez-les pour quand nginx fait proxy vers une origine : associez-les à la technique de [décalage de normalisation reverse-proxy](../reverse-proxy-and-edge/normalization-mismatch.md) contre le backend.

## Outils

- **curl --path-as-is**, Burp Intruder avec des payloads de slash/encodage.

## Références

- nginx : location, merge_slashes, internal
- PortSwigger Web Security Academy : Access control bypass
