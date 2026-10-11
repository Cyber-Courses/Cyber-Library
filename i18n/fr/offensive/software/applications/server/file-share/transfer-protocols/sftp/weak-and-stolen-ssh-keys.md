---
title: "Clés SSH faibles et volées : accès basé sur les clés à SFTP et SSH"
order: 3
description: "L'authentification par clé publique SSH est la méthode d'accès SFTP courante, et ses clés fuient : clés privées laissées dans les partages, les sauvegardes et les dépôts, clés sans phrase secrète, et clés faibles ou connues comme vulnérables. Une clé privée récupérée authentifie en tant que son propriétaire sans mot de passe, et une clé contrôlée par l'attaquant ajoutée à authorized_keys constitue un accès durable."
keywords:
  - ssh keys
  - private key
  - authorized_keys
  - passphrase
  - key reuse
---

# Clés SSH faibles et volées

L'authentification par clé publique est la manière dont la plupart des accès SFTP et SSH sont réellement accordés, et les clés privées sont mal gardées. Elles apparaissent dans les partages de fichiers lisibles, les sauvegardes, les images de conteneurs et les dépôts de code source ; elles sont fréquemment stockées sans phrase secrète ; et certaines sont faibles ou ont été générées par des outils connus comme vulnérables. Une clé privée récupérée authentifie en tant que son propriétaire sans aucun mot de passe, et inversement, écrire une clé publique contrôlée par l'attaquant dans l'`authorized_keys` d'une cible accorde un accès durable. Les clés sont aussi largement réutilisées entre hôtes, si bien qu'une seule clé qui fuit en ouvre souvent beaucoup.

```bash
# use a recovered private key
chmod 600 id_rsa; ssh -i id_rsa user@<target>; sftp -i id_rsa user@<target>
# if the key has a passphrase, crack it offline
ssh2john id_rsa > hash && john --wordlist=rockyou.txt hash
# plant a key for durable access where you can write authorized_keys (e.g. via NFS/SMB
# write to a home dir, or an SFTP account whose home you can modify)
echo 'ssh-ed25519 AAAA... attacker' >> /mnt/victim-home/.ssh/authorized_keys
```

## Notes d'exploitation

- Cherchez les clés privées partout où les fichiers sont lisibles : partages (`id_rsa`, `*.pem`, `*.ppk`), sauvegardes, historiques `.git` et images ; une clé sans phrase secrète est un accès immédiat, une clé avec phrase secrète est cassable hors ligne avec `ssh2john`.
- La réutilisation des clés est le multiplicateur de force : la même clé autorise souvent son propriétaire sur de nombreux hôtes, de sorte qu'une seule découverte peut ouvrir un parc ; cartographiez où apparaît la moitié publique.
- Écrire `authorized_keys` est à la fois une primitive d'accès et de persistance ; combinez avec une primitive d'écriture de fichier sur le home de la cible (une écriture NFS avec usurpation d'UID, un home SMB accessible en écriture, ou une écriture SFTP existante) pour y déposer une clé.
- Les clés issues de générateurs connus comme vulnérables (entropie faible) sont prévisibles et méritent d'être vérifiées par rapport aux jeux de clés connues comme compromises.

## Outils

- [john the ripper (ssh2john)](https://github.com/openwall/john)

## Références

- [authorized_keys OpenSSH](https://man.openbsd.org/sshd.8#AUTHORIZED_KEYS_FILE_FORMAT)
- [HackTricks : clés SSH](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
