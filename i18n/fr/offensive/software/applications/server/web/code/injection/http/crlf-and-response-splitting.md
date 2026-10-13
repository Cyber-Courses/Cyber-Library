---
order: 4
title: "Injection CRLF et response splitting HTTP"
description: "Injecter un retour chariot/saut de ligne dans un en-tête de réponse reflété pour ajouter des en-têtes, poser des cookies, ou scinder la réponse en un second corps contrôlé par l'attaquant."
keywords:
  - injection CRLF
  - response splitting
  - injection d'en-tête
  - injection Set-Cookie
  - "%0d%0a"
---

# CRLF et response splitting

Les en-têtes HTTP sont séparés par CR/LF (`\r\n`, `%0d%0a`), et une ligne vide (`\r\n\r\n`) sépare les en-têtes du corps. Lorsque de l'entrée non fiable est placée dans un en-tête de réponse sans retirer ces octets, un attaquant peut injecter de nouvelles lignes d'en-tête, et avec un double CRLF, une seconde réponse entière.

L'entrée est généralement une valeur que le serveur reflète dans un en-tête, comme une cible de redirection copiée dans `Location`, ou un paramètre renvoyé dans `Set-Cookie` ou un en-tête personnalisé. Injecter un seul CRLF ajoute un en-tête :

```
/redirect?url=/home%0d%0aSet-Cookie:%20sid=attacker
-> Location: /home
   Set-Cookie: sid=attacker
```

Injecter un double CRLF termine la section des en-têtes et écrit des octets contrôlés par l'attaquant dans le corps de la réponse courante, ce qui donne du cross-site scripting lorsque ce corps est rendu :

```
/redirect?url=%0d%0a%0d%0a<script>alert(document.domain)</script>
```

C'est de l'injection dans le corps de réponse : le double CRLF ne fait que démarrer le corps de l'unique réponse, il n'émet pas à lui seul une seconde ligne de statut. Le véritable response splitting, une seconde réponse cadrée séparément qui peut être mise en cache comme sa propre entrée, requiert en plus le contrôle du cadrage de la première réponse (par exemple un `Content-Length` reflété qui termine la première réponse de sorte que les octets injectés sont analysés comme une nouvelle), ce que les serveurs et bibliothèques HTTP modernes empêchent largement. Les conséquences fiables ici sont donc l'injection d'en-tête (pose de cookies, en-têtes de sécurité ou CORS) et le XSS via le corps injecté ; le cache poisoning par réponse scindée complète nécessite le cadrage supplémentaire et une stack permissive.

La réserve majeure est que la plupart des serveurs et bibliothèques HTTP modernes rejettent ou retirent CR et LF dans les API de pose d'en-têtes (`HttpServletResponse` de Java, Node, et les serveurs d'application actuels), donc le response splitting survit surtout sur des stacks plus anciennes, des routines d'écriture d'en-têtes personnalisées, ou des composants qui construisent du texte de réponse brut. Là où le framework bloque le CRLF brut, la même primitive CRLF peut encore compter dans d'autres protocoles que parle l'application (par exemple SMTP ou les lignes de log), couverts dans leurs propres sujets.

## Outils

- **Burp Suite / Burp Repeater** : forger des requêtes avec des payloads CRLF et inspecter les en-têtes de réponse reflétés.
- **curl** : envoyer des payloads d'en-tête bruts et CRLF (`%0d%0a`) directement depuis la ligne de commande.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- PortSwigger Web Security Academy: HTTP response header injection
- OWASP Testing Guide: Testing for HTTP Splitting/Smuggling
