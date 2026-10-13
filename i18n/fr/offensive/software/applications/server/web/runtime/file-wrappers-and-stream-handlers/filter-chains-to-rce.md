---
order: 2
title: "Chaînes de filtres PHP vers RCE : synthétiser un payload à partir de filtres de conversion"
description: "Transformer une primitive de lecture php://filter en exécution de code à distance en chaînant des filtres convert.iconv pour générer des octets arbitraires, sans avoir besoin d'aucun fichier sur le disque."
keywords:
  - chaîne de filtres php
  - convert.iconv
  - LFI vers RCE
  - oracle de chaîne de filtres
  - RCE sans fichier
---

# Chaînes de filtres vers RCE

Une primitive de lecture `php://filter` ressemble à de la simple divulgation, mais, chaînés, les **filtres de conversion peuvent générer des octets arbitraires à partir de rien**. En empilant de nombreuses étapes `convert.iconv.*`, un attaquant fait émettre au wrapper une chaîne PHP choisie en tête du flux, de sorte qu'un sink qui *exécute* la sortie du wrapper (comme un `include`) exécute le code de l'attaquant alors même qu'aucun fichier contrôlé par l'attaquant n'existe sur le disque. Cela fait passer de nombreux sinks LFI en lecture seule à une RCE complète.

## L'idée

`convert.iconv.<from>.<to>` transforme du texte entre encodages. Certaines paires d'encodages préfixent ou mutent des octets de façon fiable, et certaines échouent sur des valeurs d'octet précises. En les composant :

- cela permet d'**ajouter des octets contrôlés** aux données (en construisant une chaîne cible caractère par caractère), et
- cela fournit un **oracle** : une chaîne qui échoue (ou non) selon l'octet de tête, ce qui suffit à brute-forcer le contenu lorsqu'on ne dispose que d'une lecture.

Empilée suffisamment, la chaîne produit un payload `<?php ... ?>` valide comme octets de tête du flux. Lorsque le sink inclut ce flux, le payload s'exécute.

## L'utiliser

Les chaînes sont longues et mécaniques, donc générez-les avec un outil plutôt qu'à la main. **php_filter_chain_generator** construit une chaîne `php://filter/...` prête à l'emploi pour une commande choisie :

```bash
python3 php_filter_chain_generator.py --chain '<?php system($_GET["c"]); ?>'
# emits: php://filter/convert.iconv.UTF8.CSISO2022KR|...|convert.base64-decode/resource=php://temp
```

Placez le wrapper émis dans le paramètre vulnérable (par exemple `?page=<chain>`), puis pilotez la commande via le payload introduit (`&c=id`). Le `resource=` peut pointer vers un flux trivialement lisible ou vide, car tout le contenu provient des filtres.

## Notes d'exploitation

- Le sink doit *exécuter* le flux (`include`/`require`/`eval` du contenu de fichier). Un simple `file_get_contents` qui ne fait qu'afficher donne de la divulgation, pas de l'exécution, bien que la même technique de chaîne puisse toujours fabriquer du contenu pour d'autres sinks.
- Aucun `allow_url_include`, aucun fichier téléversé et aucun répertoire inscriptible ne sont requis, ce qui le rend puissant contre des LFI durcies où le log-poisoning ou les chemins de téléversement traditionnels sont fermés.
- Les chaînes sont volumineuses ; si un WAF plafonne la longueur des paramètres, ce vecteur peut être impraticable et les wrappers phar ou data sont le repli.

## Outils

- **php_filter_chain_generator** (synacktiv) et équivalents.

## Références

- Synacktiv: PHP filter chains research
- PHP manual: convert.iconv filters
