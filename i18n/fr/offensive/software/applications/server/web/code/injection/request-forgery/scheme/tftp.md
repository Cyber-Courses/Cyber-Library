---
title: "SSRF via le schéma TFTP pour la récupération de configuration"
description: "Le schéma tftp:// fait qu'un client reposant sur curl émet une requête de lecture UDP vers un serveur TFTP interne, qui contient souvent la configuration et les fichiers de boot des équipements réseau."
keywords:
  - schéma tftp
  - tftp://
  - UDP 69
  - récupération de configuration
  - équipement réseau
  - mode octet
---

# TFTP

`tftp://` pilote une requête Trivial File Transfer Protocol sur le port UDP 69 depuis un client reposant sur curl. TFTP ne nécessite aucune authentification, de sorte que lorsqu'un serveur TFTP interne existe, ce schéma lit des fichiers directement à l'intérieur, ce qui sur les réseaux internes signifie fréquemment la configuration des switchs, routeurs et du boot PXE.

## Lire un fichier

L'URL nomme l'hôte, un port facultatif et le nom du fichier à récupérer :

```
tftp://10.0.0.5/running-config
tftp://127.0.0.1:69/startup-config
tftp://10.0.0.5/pxelinux.cfg/default
```

Le client envoie une requête de lecture pour le fichier nommé en mode octet et renvoie le contenu. Parce que TFTP est non authentifié et de type requête-réponse, une seule récupération `tftp://` récupère tout ce que le serveur veut bien servir, les fichiers de configuration d'équipement (qui intègrent souvent des identifiants et des chaînes SNMP) étant la cible de choix.

## Portée et limites

TFTP n'a ni listing ni authentification, de sorte que le succès dépend de nommer un fichier que le serveur expose ; les noms de fichiers de configuration courants ci-dessus sont les suppositions habituelles. Le schéma est en UDP, donc une absence de réponse est ambiguë (perdue ou absente), ce qui le rend moins fiable comme oracle de port que les schémas TCP, mais un fichier renvoyé est une divulgation sans ambiguïté. Utilisez-le lorsqu'une sonde de [Port](../authority/port.md) ou le contexte réseau suggère la présence de TFTP, généralement aux côtés d'une infrastructure réseau.

## Outils

- **curl** : requêtes de lecture manuelles `tftp://` pour les noms de fichiers de configuration courants sur le port UDP 69.
- Tests manuels avec Burp Repeater et des payloads élaborés.

## Références

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: supported protocols](https://curl.se/docs/url-syntax.html)
