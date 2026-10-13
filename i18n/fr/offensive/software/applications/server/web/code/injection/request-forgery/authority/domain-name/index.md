---
title: "Astuces de nom de domaine dans la validation d'hôte SSRF"
description: "Lorsque la cible est écrite comme un nom d'hôte plutôt qu'une IP littérale, le timing de résolution et le traitement Unicode ouvrent des failles entre le nom qu'un validateur vérifie et l'adresse à laquelle le client se connecte."
keywords:
  - contournement de nom d'hôte SSRF
  - DNS rebinding
  - domaine internationalisé
  - punycode
  - allowlist d'hôtes
---

# Nom de domaine

Lorsque la cible SSRF est fournie comme un nom d'hôte plutôt qu'une IP littérale, une seconde couche apparaît entre la chaîne et le socket : la résolution de noms. Un validateur inspecte le nom d'hôte en tant que texte, mais le client HTTP remet ce nom à un résolveur et se connecte à l'adresse renvoyée, possiblement à un autre moment et possiblement après qu'un traitement Unicode l'a réécrit. Les deux pages ici exploitent cette couche.

- **[DNS rebinding](dns-rebinding.md)** attaque le *timing* de la résolution. Un nom que l'attaquant contrôle répond avec une adresse publique lorsque le validateur le résout, puis avec `127.0.0.1` ou une IP de métadonnées lorsque le client se connecte, de sorte qu'un seul nom d'hôte passe la vérification et atteint un service interne.
- **[Domaine internationalisé](internationalized-domain.md)** attaque le *texte* du nom d'hôte. Les labels Unicode, le punycode et la normalisation permettent à un nom de se lire comme un hôte autorisé pour une comparaison et de résoudre comme un hôte attaquant pour une autre.

Les deux élargissent la même faille que vise tout le sous-arbre [Authority](../index.md), entre l'hôte qu'un filtre voit et l'hôte que la requête atteint, mais ils le font au niveau du nommage plutôt qu'à travers des encodages d'IP bruts.

## Outils

- **Singularity of Origin** : framework d'attaque DNS rebinding pour servir une résolution contrôlée par l'attaquant.
- **Burp Collaborator** : détecter quand le serveur résout et atteint un nom d'hôte out-of-band.
- **curl** : confirmer quelle forme de nom d'hôte fournie le client résout et à laquelle il se connecte.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
