---
order: 4
title: "Schéma : les gestionnaires de protocole au-delà de http en SSRF"
description: "Une bibliothèque d'URL honore tous les schémas qu'elle enregistre. Au-delà de http et https, les gestionnaires de file, gopher, dict, ldap, ftp, tftp, jar et netdoc atteignent chacun une classe différente de cibles."
keywords:
  - schéma d'URL SSRF
  - gopher
  - schéma file
  - dict
  - ldap
  - smuggling de protocole
---

# Schéma

Le schéma est le préfixe de protocole qui sélectionne le gestionnaire chargé de résoudre une URL. Les défenses contre la SSRF se concentrent sur `http` et `https`, mais une bibliothèque d'URL dessert tous les gestionnaires que son runtime enregistre, et chacun atteint une cible différente. Fournir un schéma inattendu fait souvent la différence entre une récupération HTTP aveugle et une lecture directe de fichier ou une écriture d'octets bruts vers un service interne.

Les pages se regroupent selon ce que le gestionnaire apporte à un attaquant :

- **Données locales** : [File](file.md) lit le système de fichiers via `file://`.
- **TCP brut vers les services internes** : [Gopher](gopher.md) écrit des octets arbitraires vers un port TCP, ce qui permet de faire passer des payloads de protocole élaborés (Redis, SMTP, HTTP) et constitue le schéma le plus puissant lorsqu'il est disponible.
- **Portée spécifique à un service** : [DICT](dict.md), [LDAP](ldap.md), [SFTP](sftp.md) et [TFTP](tftp.md) dialoguent avec leurs services nommés et font aussi office de sondes d'interaction et de port.
- **HTTP lui-même** : [HTTP et HTTPS](http-and-https.md) est le schéma de base, atteignant les services web internes et l'endpoint de métadonnées, et s'associe à [Port](../authority/port.md) pour le balayage.
- **Gestionnaires spécifiques au runtime** : [JAR](jar.md) et [Netdoc](netdoc.md) sont des gestionnaires d'URL Java qui étendent la portée sur les stacks JVM.

## Sonder quels schémas se résolvent

Avant de construire un payload spécifique à un schéma, confirmez que le gestionnaire existe. Pointez chaque schéma vers un listener contrôlé par l'attaquant ou vers une cible manifestement invalide et observez l'erreur ou l'interaction : une tentative de connexion, une erreur spécifique au protocole ou un timeout distinguent chacun un gestionnaire enregistré d'un gestionnaire que la bibliothèque rejette d'emblée. L'ensemble des gestionnaires dépend du langage, de la bibliothèque d'URL et de sa configuration, aussi l'énumération précède-t-elle l'exploitation.

## Outils

- **SSRFmap** : exploitation automatisée de SSRF sur les cibles internes avec des modules par schéma.
- **Gopherus** : construit des payloads gopher pour Redis, FastCGI et SMTP via SSRF.
- **Burp Collaborator** : détection d'interactions hors-bande pour les SSRF aveugles.
- **curl** : sondage manuel des schémas qu'un gestionnaire résout.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
