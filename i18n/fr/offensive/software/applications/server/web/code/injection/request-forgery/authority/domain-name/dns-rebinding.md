---
title: "DNS rebinding pour déjouer les allowlists d'hôtes SSRF"
description: "Un nom d'hôte qui résout vers une IP publique au moment de la validation et vers une IP interne au moment de la connexion laisse un seul nom autorisé atteindre la loopback ou les métadonnées via une faille time-of-check to time-of-use dans la résolution."
keywords:
  - DNS rebinding
  - TOCTOU résolution
  - TTL court
  - IP de métadonnées
  - contournement d'allowlist d'hôtes
  - résoudre deux fois
---

# DNS rebinding

Les défenses SSRF basées sur l'hôte résolvent le nom d'hôte fourni, vérifient l'IP obtenue contre une politique, et autorisent la requête si elle paraît externe. Le DNS rebinding casse cela en faisant résoudre le nom vers des adresses *différentes* au moment de la vérification et au moment de la connexion. Le validateur voit une IP publique et approuve ; quelques instants plus tard le client résout de nouveau le même nom et se connecte à une IP interne.

## La faille time-of-check

La faiblesse réside dans deux résolutions distinctes d'un même nom d'hôte : une par le validateur, une par le client HTTP lorsqu'il ouvre le socket. Si le serveur faisant autorité que l'attaquant contrôle renvoie une réponse différente pour la seconde recherche, la décision de politique ne correspond plus à la connexion.

L'attaquant sert `attacker.example` depuis un serveur de noms qu'il opère et répond avec un TTL très court afin que le résolveur ne mette pas en cache :

```
attacker.example.  1  IN  A  203.0.113.10     # public, returned at validation
```

Une fois que l'application a validé et mis en cache sa décision d'autorisation, la requête suivante pour le même nom est répondue avec une adresse interne :

```
attacker.example.  1  IN  A  127.0.0.1        # loopback, returned at connect
```

Le TTL court (souvent `0` ou `1`) force le client à re-résoudre entre la vérification et la connexion, donc la seconde réponse est celle que le socket utilise. La cible est typiquement `127.0.0.1`, un hôte RFC1918, ou l'adresse de métadonnées `169.254.169.254`.

## Variantes à réponses multiples et à bascule rapide

Certaines piles ne résolvent qu'une seule fois, donc une pure bascule vérifier-puis-connecter ne s'applique pas. Deux variantes les couvrent :

- **Enregistrements A multiples** : renvoyer à la fois une adresse publique et une adresse interne dans une seule réponse et laisser le client les essayer à tour de rôle. Un validateur qui n'inspecte que le premier enregistrement passe, tandis que le client peut se connecter au second.
- **Réponse rapidement alternée** : alterner les réponses sur une minuterie de sorte que, quelle que soit la résolution que le client effectue après la validation, elle atterrisse sur l'adresse interne. Un outillage qui automatise cette bascule (servant du public, puis du privé, en fonction du nombre de requêtes ou du temps) rend la course fiable.

## Fetches pilotés par le navigateur

Lorsque le fetch est effectué par un [navigateur headless](../../fetch/headless-browser.md), le rebinding franchit aussi la propre frontière réseau du navigateur : une page chargée depuis l'adresse publique de l'attaquant se re-lie ensuite à une IP interne tout en gardant la même origine, de sorte que le script sur la page lit des réponses internes. Les vérifications Private Network Access qui s'appuient sur l'adresse apparente du document sont satisfaites par la première réponse, publique.

## Pourquoi cela bat spécifiquement les allowlists

Une allowlist de *noms d'hôtes* autorisés est la cible idéale : le nom reste constant et autorisé, seule son adresse change. Un contrôle qui résout le nom une seule fois, épingle cette IP exacte et se connecte à l'adresse épinglée ferme la faille, ce qui explique pourquoi le rebinding fonctionne contre les vérifications basées sur le nom mais pas contre les conceptions connecter-à-l'IP-validée. Lorsque l'application ne résout qu'une fois et épingle, passez plutôt à une approche de [domaine internationalisé](internationalized-domain.md) ou d'[adresse IP](../ip-address.md) brute.

## Outils

- [Singularity of Origin](https://github.com/nccgroup/singularity) (framework d'attaque DNS rebinding)
- [rebind / rbndr](https://github.com/taviso/rbndr) (serveur de noms de rebinding à bascule rapide)

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
