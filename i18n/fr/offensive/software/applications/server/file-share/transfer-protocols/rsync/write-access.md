---
title: "Accès en écriture : déposer des fichiers via un module rsync inscriptible"
order: 1
description: "Un module rsync configuré en lecture-écriture (read only = false) permet à un attaquant de déposer des fichiers dans le répertoire serveur qu'il expose. Lorsque ce répertoire est consommé par l'hôte, une racine web, un répertoire personnel, un chemin de cron ou de scripts, l'écriture devient exécution de code ou dépose d'identifiants, limitée par le chroot, l'uid du démon et le fait qu'il préserve ou non les permissions."
keywords:
  - rsync write
  - read only false
  - upload
  - webroot
  - authorized_keys
---

# Accès en écriture

Un module rsync avec `read only = false` accepte les dépôts, de sorte qu'un attaquant écrit des fichiers dans le répertoire serveur qu'il expose. Comme pour tout partage inscriptible, l'impact dépend de ce qui consomme ce répertoire : un module exposant une racine web transforme un dépôt en web shell, un module exposant un répertoire personnel permet de déposer un `authorized_keys`, et un module exposant un chemin de cron ou de scripts exécute votre code selon la planification de l'hôte. La configuration du démon, le chroot (`use chroot`), l'`uid`/`gid` sous lequel il effectue les écritures, et le fait qu'il préserve ou non les permissions, déterminent jusqu'où l'écriture porte et quelle propriété prennent les fichiers déposés.

```bash
# confirm the module is writable
echo test > t; rsync -av t rsync://<target>/<module>/ && echo "writable"
# plant into a consumed location (examples depend on what the module maps)
rsync -av shell.php rsync://<target>/<webroot-module>/        # web shell if it is a webroot
rsync -av --chmod=600 authorized_keys rsync://<target>/<home-module>/.ssh/   # key plant
rsync -av cronjob rsync://<target>/<cron-module>/             # if it maps /etc/cron.d
```

## Notes d'exploitation

- L'inscriptibilité seule n'est pas l'exécution ; le gain nécessite que le répertoire exposé fasse l'objet d'un traitement, alors identifiez ce que le module expose (racine web, home, cron, déploiement) à partir de son nom, de son contenu et du comportement du serveur.
- L'`uid`/`gid` du démon déterminent la propriété du fichier déposé et donc quels consommateurs l'acceptent ; un démon s'exécutant en tant que root (ou écrivant vers un chemin consommé par root) rend efficaces les déposes d'`authorized_keys`/de cron.
- `use chroot` confine les écritures à la racine du module ; sans autre erreur de configuration, vous ne pouvez pas en sortir, alors ciblez ce qui, à l'intérieur du module, est tout de même exécuté par l'hôte.
- Préférez un consommateur déterministe (cron, une clé de connexion, une racine web servie) plutôt que d'espérer qu'un fichier soit exécuté ; la mécanique reflète l'[empoisonnement de partage SMB inscriptible](../../network-file-systems/smb/writable-share-poisoning/index.md).

## Références

- [rsyncd.conf : read only, use chroot, uid/gid](https://download.samba.org/pub/rsync/rsyncd.conf.5)
- [HackTricks : rsync write](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
