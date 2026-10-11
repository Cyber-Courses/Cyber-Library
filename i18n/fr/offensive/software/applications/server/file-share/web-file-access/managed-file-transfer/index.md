---
title: "Transfert de fichiers géré : attaquer les appliances MFT"
description: "Les produits de transfert de fichiers géré fournissent des échanges de fichiers inter-entreprises gouvernés via des portails web et des API exposés sur Internet. Leur exposition et les données sensibles qu'ils intermédient en ont fait des cibles de premier choix : les contournements d'authentification et les failles d'injection vers RCE dans les appliances MFT ont motivé certaines des plus grandes campagnes d'exploitation de masse et de vol de données, notamment contre MOVEit, GoAnywhere, CrushFTP et Serv-U."
keywords:
  - managed file transfer
  - mft
  - moveit
  - goanywhere
  - crushftp
---

# Transfert de fichiers géré

Les produits de transfert de fichiers géré (MFT) existent pour déplacer des fichiers entre organisations sous gouvernance : audit, chiffrement, rétention et contrôle d'accès, exposés via un portail web et des API qui sont, par conception, tournés vers Internet. Cette combinaison, exposition externe plus concentration de données sensibles provenant de nombreux partenaires, a fait des appliances MFT l'une des classes de cibles les plus lourdes de conséquences ces dernières années. Les vulnérabilités se regroupent en deux types : les contournements d'authentification qui atteignent les fonctions d'administration ou de fichiers sans identifiants, et les failles d'injection (injection SQL, désérialisation, injection de commande) qui escaladent vers une exécution de code à distance. Plusieurs ont été exploitées en masse pour du vol de données.

```bash
# fingerprint the MFT product and version
curl -skI https://<target>/                   # Server/app headers
curl -sk https://<target>/ | grep -iE 'moveit|goanywhere|crushftp|serv-u|filezilla'
```

## Sous-thèmes

- **[Contournement d'authentification](authentication-bypass.md)** : atteindre les fonctions MFT sans identifiants.
- **[Injection vers RCE](injection-to-rce.md)** : injection SQL, désérialisation et injection de commande.
- **[Exploits MFT connus](known-mft-exploits/index.md)** : les campagnes spécifiques aux appliances.

## Références

- [Alertes CISA sur l'exploitation du MFT](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [OWASP : injection et authentification](https://owasp.org/www-project-top-ten/)
