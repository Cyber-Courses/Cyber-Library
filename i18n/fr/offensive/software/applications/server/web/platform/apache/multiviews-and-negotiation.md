---
order: 3
title: "MultiViews et négociation de contenu Apache : énumération de fichiers et divulgation de source"
description: "Abus d'Apache mod_negotiation MultiViews pour énumérer des fichiers par nom de base, découvrir des variantes non référencées et divulguer le code source ou des sauvegardes que le serveur propose comme représentations négociées."
keywords:
  - MultiViews
  - mod_negotiation
  - négociation de contenu
  - énumération de fichiers
  - apache
---

# MultiViews et négociation de contenu

`mod_negotiation` avec `Options +MultiViews` permet à Apache de servir la variante correspondant le mieux quand une requête n'a pas de fichier exact. Demander `/page` fait chercher à Apache `page.html`, `page.php`, `page.en`, `page.bak`, et ainsi de suite, puis en choisir un. Pour un attaquant, cela transforme un nom de base en oracle d'énumération de fichiers et de divulgation de source.

## Énumération des variantes

Comme Apache signale les représentations disponibles, une requête sur un nom de base (ou un `Accept` incorrect) peut révéler toutes les variantes existantes :

```
GET /index HTTP/1.1
Accept: application/x-does-not-exist
```

Quand aucune variante acceptable ne correspond, `mod_negotiation` peut renvoyer un **406 Not Acceptable** listant les représentations disponibles par nom de fichier, divulguant des variantes telles que `index.php`, `index.php.bak`, `index.old` et des copies étiquetées par langue qui ne sont liées nulle part. Cela énumère des fichiers sans wordlist.

## Divulgation et contenu non référencé

- Une variante `.bak`/`.old`/`.txt` négociée est servie avec un type de contenu statique, divulguant la source (recoupe [sauvegardes et fichiers temporaires](../general/backup-and-temporary-files.md), mais ici c'est Apache qui offre le nom).
- Les requêtes sur un nom de base atteignent des variantes que l'application ne lie jamais, exposant des pages de dev/test et des copies plus anciennes.
- `MultiViews` combiné à un mappage de handler peut aussi changer quelle variante s'exécute ou se divulgue.

## Exploitation

- Demandez des noms de base de pages connues sans extension et avec des en-têtes `Accept` hostiles ; lisez la liste de représentations du 406 pour obtenir les noms de fichiers des variantes.
- Récupérez directement chaque variante divulguée (les copies `.bak`/`.old` sont les plus précieuses pour le code source et les secrets).
- Réinjectez les scripts découverts dans les vérifications de [mappage des handlers et des types](handler-and-type-mapping.md).

## Outils

- **curl**/Burp (manipuler `Accept`) ; outillage de content-discovery pour les noms de base.

## Références

- Apache httpd : mod_negotiation, Options MultiViews
- OWASP WSTG : Review old/unreferenced files
