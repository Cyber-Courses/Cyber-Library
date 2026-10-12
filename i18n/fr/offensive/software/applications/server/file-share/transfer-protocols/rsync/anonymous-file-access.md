---
title: "Accès anonyme aux fichiers : lire le contenu des modules rsync sans identifiants"
order: 3
description: "Les modules du démon rsync dépourvus de directive auth users sont lisibles par n'importe qui, de sorte qu'un attaquant télécharge l'intégralité de leur contenu de manière anonyme. Comme les modules rsync sauvegardent ou publient fréquemment des répertoires serveur, ce contenu renferme des configurations, du code source, des clés et des sauvegardes entières, faisant d'un module anonyme un point d'appui direct pour le vol de données."
keywords:
  - rsync anonymous
  - download
  - backup
  - loot
  - rsync module
---

# Accès anonyme aux fichiers

Un module rsync configuré sans `auth users` accepte n'importe quel client, de sorte que son contenu est entièrement lisible de manière anonyme. Les modules rsync sont couramment utilisés pour sauvegarder ou publier des répertoires serveur, ce qu'ils exposent est donc considérable : arborescences de configuration, code source web et applicatif, clés SSH et TLS, et jeux de sauvegardes complets. Télécharger un module anonyme constitue donc un point d'appui direct pour le vol de données, équivalent à la lecture d'un partage, et rsync rend trivial le miroir de l'arborescence entière.

```bash
# mirror an entire anonymous module for offline loot
rsync -av rsync://<target>/<module>/ ./loot/
# or the double-colon form
rsync -av <target>::<module>/ ./loot/
# then mine the contents for secrets
grep -rinE 'password|secret|BEGIN (RSA|OPENSSH) PRIVATE KEY|api[_-]?key' ./loot | head
find ./loot -iname 'id_rsa' -o -iname '*.pem' -o -iname '*.conf' -o -iname '*.bak'
```

## Notes d'exploitation

- `rsync -av` met en miroir le module de façon récursive avec les attributs, extrayant efficacement toute l'arborescence pour une analyse hors ligne ; triez avec les mêmes motifs d'identifiants et de fichiers sensibles utilisés pour tout butin de partage.
- Les modules de sauvegarde sont les plus riches : un module qui contient des sauvegardes de serveur ou de base de données peut livrer des systèmes entiers (et, à partir d'une sauvegarde de DC, les hashs du domaine), comme pour le pillage de partages de sauvegarde.
- L'accès en lecture est peu bruyant et ne nécessite aucun identifiant ; c'est l'action première évidente contre tout module anonyme trouvé lors de l'[énumération](module-enumeration.md).
- Les clés et configurations récupérées permettent de pivoter plus loin (SSH, services) ; réinjectez-les dans l'environnement.

## Références

- [rsync manual](https://download.samba.org/pub/rsync/rsync.1)
- [HackTricks : rsync anonyme](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
