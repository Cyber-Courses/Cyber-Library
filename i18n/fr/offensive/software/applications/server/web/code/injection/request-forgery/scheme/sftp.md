---
title: "SSRF via le schéma SFTP contre les endpoints internes"
description: "Le schéma sftp:// fait qu'un client reposant sur curl se connecte à un service de transfert de fichiers SSH interne, ce qui sonde le port 22 et, lorsque des identifiants sont fournis ou réutilisés, atteint le transfert de fichiers interne."
keywords:
  - schéma sftp
  - sftp://
  - port 22
  - transfert de fichiers interne
  - SSH
  - réutilisation d'identifiants
---

# SFTP

`sftp://` pilote une connexion de transfert de fichiers SSH depuis un client reposant sur curl. En tant que schéma SSRF, sa principale valeur est d'atteindre et de sonder un service SSH interne, le transfert de fichiers n'étant disponible que lorsque la connexion peut s'authentifier.

## Se connecter à un service interne

L'URL porte l'hôte, un port facultatif et un chemin :

```
sftp://127.0.0.1:22/
sftp://10.0.0.5/etc/hostname
```

Le client ouvre une connexion vers le port 22 (ou le port indiqué) et entame le handshake SSH. Parce que la connexion réussit, échoue ou reste suspendue de manière distincte, `sftp://` est un oracle net pour savoir si un hôte interne exécute SSH, complétant un balayage de [Port](../authority/port.md) par une sonde consciente du protocole.

## L'authentification est la limite

Contrairement à [File](file.md) ou [Gopher](gopher.md), SFTP exige une authentification SSH réussie avant qu'un fichier ne soit lu, de sorte qu'un `sftp://` nu vers un hôte interne confirme le service mais ne renvoie pas de données par lui-même. Il devient une voie de lecture de fichier lorsque le sink laisse des identifiants voyager dans l'URL ou l'environnement, ou lorsque le client est configuré pour réutiliser une clé ou un agent, auquel cas fournir un identifiant connu ou réutilisé atteint le transfert interne :

```
sftp://user:pass@10.0.0.5/home/user/.ssh/id_rsa
```

Lorsqu'aucun identifiant n'est disponible, traitez `sftp://` comme un oracle de présence de service et d'accessibilité plutôt que comme une primitive de divulgation.

## Outils

- **curl** : sondage manuel `sftp://` pour tester si un hôte interne parle SSH sur le port 22.
- Tests manuels avec Burp Repeater et des payloads élaborés.

## Références

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: supported protocols](https://curl.se/docs/url-syntax.html)
