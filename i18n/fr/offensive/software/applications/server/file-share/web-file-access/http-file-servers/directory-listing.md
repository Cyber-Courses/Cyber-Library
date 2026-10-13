---
title: "Listage de répertoire : cartographier l'arborescence de fichiers servie"
order: 1
description: "Lorsqu'un serveur de fichiers HTTP n'a pas de fichier d'index ou que l'autoindex est activé, il rend un listage de répertoire qui expose chaque fichier et sous-répertoire. Un attaquant utilise ce listage, ou le force, pour cartographier l'arborescence servie, découvrir des fichiers qui n'étaient pas censés être liés (sauvegardes, configurations, code source), et trouver les points de téléversement ou de traversée à attaquer ensuite."
keywords:
  - directory listing
  - autoindex
  - index of
  - file discovery
  - enumeration
---

# Listage de répertoire

Un listage de répertoire est l'index automatique qu'un serveur web rend pour un répertoire dépourvu de fichier d'index (ou lorsque l'autoindex est explicitement activé). Il expose chaque fichier et sous-répertoire par son nom, ce qui est un cadeau pour la reconnaissance : les fichiers jamais liés depuis l'application, les sauvegardes, les copies `.bak` et `.old`, la configuration, les archives de code source et les journaux deviennent tous visibles et téléchargeables. Cartographier l'arborescence listée permet de trouver les fichiers sensibles à récupérer et les points de téléversement ou dynamiques à attaquer.

```bash
# detect and walk listings
curl -s http://<target>/            # "Index of /" or a file-server UI => listing
# recursively mirror a listed tree
wget -r -np -R 'index.html*' http://<target>/
# force/confirm listing on specific directories (dropping the index file name)
for d in backup old uploads config .git includes; do
  curl -s -o /dev/null -w "%{http_code} $d/\n" http://<target>/$d/; done
```

## Notes d'exploitation

- Priorisez par nom : `backup`, `old`, les suffixes `.bak`/`.old`, `config`, `.git`, `uploads` et les fichiers d'archive (`.zip`, `.tar.gz`, `.sql`) sont les trouvailles de grande valeur qu'un listage révèle.
- Un listage qui expose `.git/` permet une récupération complète du code source (extraction du dépôt), et les fichiers `.sql`/de sauvegarde exposés donnent des données et des identifiants hors ligne.
- Même là où l'application ne lie aucun listage, des répertoires individuels peuvent tout de même avoir l'autoindex ; sondez directement les noms de répertoires probables.
- Le listage est la carte des autres attaques : il localise le répertoire de téléversement accessible en écriture ([Téléversement de fichier vers RCE](file-upload-to-rce.md)) et montre la structure vers laquelle diriger la traversée ([Traversée de chemin](path-traversal.md)).

## Références

- [OWASP : indexation de répertoire](https://owasp.org/www-project-web-security-testing-guide/)
- [Comportement de l'autoindex Apache/nginx](https://httpd.apache.org/docs/current/mod/mod_autoindex.html)
