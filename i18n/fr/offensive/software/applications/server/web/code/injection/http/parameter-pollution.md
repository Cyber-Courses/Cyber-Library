---
order: 6
title: "HTTP parameter pollution (HPP)"
description: "Fournir des paramètres HTTP dupliqués que différents composants résolvent de façon incohérente, pour contourner la validation et les WAF ou écraser des valeurs après un contrôle."
keywords:
  - HTTP parameter pollution
  - HPP
  - paramètres dupliqués
  - contournement de WAF
  - précédence de paramètre
---

# Parameter pollution

Le HTTP parameter pollution (HPP) fournit le même paramètre plus d'une fois (`?id=1&id=2`) et exploite le fait que les plateformes résolvent les doublons différemment. Lorsque deux composants de la stack choisissent des occurrences différentes, une valeur qui passe un contrôle est celle qui n'est pas utilisée en aval.

La résolution varie selon la plateforme : PHP/Apache et de nombreux frameworks prennent la **dernière** occurrence, d'autres prennent la **première**, ASP.NET/IIS (et l'ASP classique) les **concatènent** avec une virgule (`1,2`), et JSP/servlets les exposent sous forme de **tableau** où le code lit souvent l'index 0. Connaître la règle de la cible décide quelle copie armer.

Deux abus s'ensuivent. Un WAF ou un filtre d'entrée qui inspecte la première occurrence peut être contourné lorsque l'application utilise la dernière, de sorte qu'une première valeur bénigne cache une seconde malveillante (utile pour faire passer un payload d'injection devant un filtre) :

```
/search?q=safe&q=' OR '1'='1
```

Le contournement de logique écrase une valeur après qu'elle a été validée : fournissez la valeur autorisée là où le validateur lit et la valeur de l'attaquant là où la logique métier lit. La duplication query-string contre corps est un cas connexe, puisque de nombreux frameworks fusionnent les deux portées et l'une peut l'emporter sur l'autre (`?role=user` dans l'URL et `role=admin` dans le corps).

Le HPP est un bug de précédence, pas un payload en soi, donc il est généralement combiné avec une autre technique (injection SQL, contrôle d'accès) dont le payload chevauche l'occurrence que le composant vulnérable lit. Confirmez d'abord la règle de résolution de doublons de la cible en observant quelle valeur prend effet.

## Outils

- **Burp Suite** : dupliquer des paramètres à travers la query et le corps et observer quelle occurrence prend effet.
- **Param Miner** : extension Burp pour découvrir des paramètres cachés et dupliqués.
- **curl** : envoyer des paramètres répétés et des doublons inter-portées depuis la ligne de commande.

## Références

- OWASP Testing Guide: Testing for HTTP Parameter Pollution
- PortSwigger Web Security Academy: parameter pollution notes
