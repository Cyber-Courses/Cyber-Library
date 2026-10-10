---
title: "Butin et fichiers sensibles : exploiter les partages SMB lisibles"
order: 5
description: "Les partages SMB lisibles contiennent régulièrement les identifiants et les données qui font progresser une intrusion : scripts et fichiers de configuration avec des mots de passe en dur, sauvegardes et images disque, données de stratégie de groupe dans SYSVOL, clés et jetons. Parcourir systématiquement les partages et rechercher des motifs dans leur contenu transforme l'accès en lecture en identifiants et données sensibles."
keywords:
  - smb loot
  - fichiers sensibles
  - chasse aux identifiants
  - sysvol
  - spider
---

# Butin et fichiers sensibles

Un partage lisible n'est que rarement un simple ensemble de fichiers ; c'est l'endroit où une organisation laisse les identifiants et les données qui font avancer un attaquant. Les scripts et fichiers de configuration embarquent des mots de passe de service et des chaînes de connexion, les sauvegardes et images disque contiennent des systèmes entiers à casser hors ligne, le partage de domaine `SYSVOL` héberge la stratégie de groupe (et parfois des identifiants mis en cache), et les développeurs comme les administrateurs laissent des clés, des jetons et des listes de mots de passe dans les répertoires partagés. La méthode consiste à parcourir les partages lisibles et à rechercher des motifs dans leur contenu à grande échelle plutôt que de naviguer à la main.

```bash
# inventory and search readable shares for likely loot
nxc smb <target> -u user -p 'pass' -M spider_plus -o DOWNLOAD_FLAG=True
# pattern-match downloaded content for secrets
grep -rinE 'password|passwd|pwd|secret|connectionstring|api[_-]?key' ./loot | head
# high-value filetypes to pull selectively
#   *.config *.ini *.xml *.ps1 *.bat *.vbs *.kdbx *.ppk *.pem id_rsa *.vmdk *.bak
```

## Sous-thèmes

- **[Chasse aux identifiants](credential-hunting.md)** : trouver des mots de passe, des clés et des jetons dans le contenu des partages.
- **[Sauvegardes et archives](backups-and-archives.md)** : exploiter les sauvegardes, les images et les archives pour récupérer des systèmes entiers.

## Références

- [NetExec : parcourir les partages](https://www.netexec.wiki/smb-protocol)
- [MITRE ATT&CK : identifiants non sécurisés dans les fichiers](https://attack.mitre.org/techniques/T1552/001/)
