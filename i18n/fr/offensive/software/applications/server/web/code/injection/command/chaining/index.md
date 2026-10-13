---
title: "Chaînage de commandes et séparateurs"
description: "Opérateurs de contrôle du shell qui permettent à une valeur injectée de terminer la commande prévue et d'en démarrer une autre : ; && || et &."
keywords:
  - chaînage de commandes
  - séparateur de commandes
  - exécution séquentielle
  - exécution conditionnelle
  - exécution en arrière-plan
---

# Chaînage

Les opérateurs de contrôle du shell permettent à une valeur injectée de terminer la commande prévue et d'en commencer une autre. Un point-virgule exécute les commandes **séquentiellement** ; `&&` et `||` les exécutent **conditionnellement** selon le succès ou l'échec de la commande précédente (un oracle booléen utile) ; et une esperluette finale `&` exécute un job en **arrière-plan**. Chaque fois que l'application invoque un shell, chacun de ces opérateurs transforme un seul paramètre injectable en exécution de commande arbitraire.

## Pages

- **[Exécution en arrière-plan (&)](background.md)** : Utiliser & pour détacher une commande injectée afin qu'elle s'exécute de façon asynchrone, gardant la réponse rapide pendant qu'un payload s'exécute, et la signification de & comme...
- **[Exécution conditionnelle (&& et ||)](conditional.md)** : Utiliser les opérateurs de shell AND (&&) et OR (||) pour exécuter des commandes en cas de succès ou d'échec et construire un oracle d'exfiltration booléen en injection de commande OS aveugle.
- **[Exécution séquentielle (;)](sequential.md)** : Utiliser ; pour ajouter des commandes qui s'exécutent inconditionnellement après qu'une valeur injectée atteint un shell, la primitive d'injection de commande OS la plus simple.

## Outils

- **[commix](https://github.com/commixproject/commix)** : détection et exploitation automatisées de l'injection de commande.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour l'injection de séparateurs.

## Références

- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
