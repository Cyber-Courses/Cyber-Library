---
title: "Commentaires versionnés et en ligne MySQL pour le contournement de filtre"
description: "Utiliser les commentaires /*! versionnés */ et /**/ en ligne de MySQL pour découper des mots-clés bloqués et exécuter conditionnellement du SQL malgré les filtres par signature."
keywords:
  - commentaire versionné
  - contournement par commentaire en ligne
  - découpage de mot-clé
  - évasion de filtre MySQL
---

# Commentaires versionnés

MySQL traite `/*! ... */` comme un commentaire versionné : le contenu est ignoré par les autres bases de données mais exécuté par MySQL, et avec un numéro de version intégré comme `/*!50000 ... */` il n'est exécuté que sur MySQL à partir de cette version (5.0.0 ici). C'est un grand classique de l'évasion de filtre car le mot-clé se cache dans ce qui ressemble à un commentaire.

Découper un mot-clé bloqué entre des commentaires versionnés défait une signature littérale `UNION SELECT` tout en s'exécutant :

```sql
' /*!50000UNION*/ /*!50000SELECT*/ 1,2,3-- 
```

Les commentaires en ligne simples `/**/` remplacent les espaces, de sorte qu'un filtre cherchant le jeton `UNION SELECT` séparé par un espace manque la forme accolée :

```sql
'/**/UNION/**/SELECT/**/1,2,3-- 
```

Les deux se combinent librement, et les commentaires peuvent aussi être insérés en milieu de mot-clé uniquement dans la forme versionnée (un simple `UN/**/ION` n'est pas valide, mais `/*!UNION*/` l'est). Comme la clause de version abandonne silencieusement le payload sur les serveurs en dessous du numéro, fixez-le bas (ou omettez-le) sauf si vous cherchez délibérément à prendre l'empreinte de la version en activant et désactivant l'exécution.

## Outils

- **sqlmap** : scripts tamper `versionedkeywords` et `versionedmorekeywords` pour l'évasion par commentaire.
- **Burp Repeater** : fabriquez à la main des payloads à commentaires versionnés et en ligne.

## Références

- MySQL Reference Manual : comment syntax, versioned comments
- OWASP Testing Guide : Testing for SQL Injection
