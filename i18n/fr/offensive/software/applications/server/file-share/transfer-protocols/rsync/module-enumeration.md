---
title: "Énumération des modules : lister les modules du démon rsync et l'accès"
order: 2
description: "Le démon rsync liste ses modules à tout client qui le demande, révélant les noms des partages et leurs commentaires. Sonder chaque module montre s'il nécessite une authentification et s'il est lisible ou inscriptible, produisant la carte qui oriente le pillage et la dépose de fichiers contre le démon."
keywords:
  - rsync modules
  - rsyncd
  - enumeration
  - listing
  - access
---

# Énumération des modules

Le démon rsync annonce ses modules à quiconque se connecte, sans qu'aucun identifiant ne soit requis pour le listage, l'énumération est donc immédiate. Chaque module est une exportation nommée correspondant à un répertoire serveur ; le listage montre les noms et les commentaires, et sonder chaque module révèle s'il exige une authentification (`auth users`) et si l'accès courant est en lecture ou en écriture. Cette carte, quels modules sont anonymes, lisibles et inscriptibles, oriente tout ce qui suit.

```bash
# list all modules and their comments
rsync rsync://<target>/
# equivalent double-colon syntax
rsync <target>::
# probe a specific module: does it need auth? is it listable?
rsync rsync://<target>/<module>/              # lists the module's top-level contents
rsync -av --list-only rsync://<target>/<module>/   # recursive-ish listing of entries
```

Lisez les résultats : un module qui liste son contenu sans demander de mot de passe est lisible de manière anonyme ; un module qui renvoie une erreur d'exigence d'authentification nécessite des identifiants ; l'inscriptibilité se confirme séparément en tentant un dépôt.

## Notes d'exploitation

- Le listage des modules est toujours disponible à sonder (c'est ainsi que les clients découvrent les partages), c'est donc une première étape fiable ne nécessitant aucun accès.
- Les noms et commentaires des modules révèlent leur usage (sauvegardes, contenu web, config, répertoires personnels), ce qui hiérarchise ceux à piller ou à cibler pour des écritures.
- Un module anonyme qui liste son contenu est immédiatement lisible, voir [Accès anonyme aux fichiers](anonymous-file-access.md) ; testez chacun pour l'écriture séparément, voir [Accès en écriture](write-access.md).
- Les modules nécessitant `auth users` requièrent des identifiants (l'authentification du démon rsync utilise un fichier de secrets, pas des comptes système) ; notez-les pour des attaques sur les identifiants mais concentrez-vous d'abord sur les anonymes.

## Outils

- [rsync](https://download.samba.org/pub/rsync/rsync.1)
- [nmap rsync-list-modules](https://nmap.org/nsedoc/scripts/rsync-list-modules.html)

## Références

- [rsyncd.conf](https://download.samba.org/pub/rsync/rsyncd.conf.5)
- [HackTricks : rsync](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
