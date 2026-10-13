---
title: "Injection sur caractère large (GBK) dans MySQL"
description: "Contourner un échappement ignorant le jeu de caractères dans MySQL en utilisant des séquences multioctets GBK pour qu'un antislash ajouté soit consommé et que le guillemet survive."
keywords:
  - injection sur caractère large
  - injection GBK
  - contournement d'addslashes
  - jeu de caractères multioctet
  - contournement d'échappement
---

# Injection sur caractère large

L'injection sur caractère large défait l'échappement qui ajoute un antislash devant un guillemet (`'` devient `\'`) sans tenir compte du jeu de caractères de la connexion. Sur un jeu de caractères multioctet tel que GBK, un octet de tête forgé se combine avec l'antislash d'échappement pour former un seul caractère multioctet valide, ce qui laisse le guillemet suivant debout comme délimiteur de chaîne.

Fournir `%bf%27` est le cas classique. Un échappement naïf transforme le guillemet `%27` en `%5c%27` (`\'`), produisant la séquence d'octets `%bf%5c%27`. Sous GBK, `%bf%5c` est un seul caractère valide, de sorte que l'antislash est consommé et que le guillemet `%27` est libre de sortir de la chaîne :

```
input:   %bf%27 OR 1=1-- 
escaped: %bf%5c%27 OR 1=1--     (0xBF5C = one GBK character, quote survives)
```

Les conditions préalables sont précises : le jeu de caractères de la connexion est GBK (ou un autre encodage multioctet vulnérable), et l'échappement ignore le jeu de caractères, comme `addslashes()` ou `mysql_real_escape_string()` appelé avant que le jeu de caractères ne soit correctement défini. Définir correctement le jeu de caractères avec `mysqli_set_charset('gbk')` rend `mysql_real_escape_string` conscient du jeu de caractères et comble la faille, de sorte que cela affecte surtout les piles plus anciennes ou mal configurées. D'autres octets de tête tels que `%bf`, `%df` et `%a1` fonctionnent sur le même principe.

## Outils

- **sqlmap** : script tamper `unmagicquotes` pour le contournement d'échappement sur caractère large.
- **Burp Repeater** : fabriquez manuellement le payload multioctet `%bf%27`.

## Références

- MySQL Reference Manual : character sets, `mysql_real_escape_string` charset handling
- OWASP Testing Guide : Testing for SQL Injection
