---
title: "Identifiants par défaut : connexions SSH d'usine et constructeur"
description: "Les équipements réseau, les objets connectés et les logiciels préinstallés sont souvent livrés avec des identifiants SSH fixes jamais changés : root:root, admin:admin, des paires propres au constructeur et des comptes de support codés en dur. Essayer les valeurs par défaut documentées de l'appareil contre SSH est souvent l'accès authentifié le plus rapide, et ces comptes sont fréquemment administratifs."
keywords:
  - identifiants par défaut
  - ssh
  - valeur par défaut constructeur
  - iot
  - mot de passe d'usine
---

# Identifiants par défaut

Une grande partie des systèmes exposés en SSH ne sont pas des serveurs généralistes mais des équipements et des appareils, routeurs, commutateurs, caméras, unités NAS et produits Linux embarqués, livrés avec des identifiants SSH fixes. Les constructeurs les documentent, les utilisateurs les changent rarement, et ils sont fréquemment administratifs ou root. Donc avant la force brute, le geste au plus fort rendement contre un équipement est d'essayer ses valeurs par défaut connues : paires génériques (`root:root`, `admin:admin`, `root:toor`), comptes propres au constructeur, et connexions de support ou de maintenance codées en dur que certains produits intègrent.

```bash
# identify the device/vendor from the banner, then try its documented defaults
nc <target> 22                                 # banner may name the product/firmware
ssh root@<target>        # try root:root, root:<vendor>, admin:admin, etc.
# spray a curated default-credential list across the service
nxc ssh <target> -u defaults_users.txt -p defaults_pass.txt --no-bruteforce   # pair 1:1
hydra -C vendor_defaults.txt ssh://<target>    # combined user:pass list
```

## Notes d'exploitation

- Identifiez d'abord l'appareil (bannière, interface web associée, chaînes du firmware) ; les identifiants par défaut sont propres au produit, donc l'identification transforme une devinette en simple recherche.
- Utilisez un appariement 1:1 utilisateur:mot de passe (`-C` dans hydra, listes combinées) pour les valeurs par défaut documentées plutôt qu'un produit croisé complet, ce qui est plus rapide et plus discret.
- Ces comptes sont souvent root/administratifs sur les équipements : un succès est donc généralement un contrôle complet de l'appareil, pas un shell à faibles privilèges.
- Les comptes de support codés en dur ou non documentés (une porte dérobée par conception) sont une classe connue sur certains appareils ; l'analyse de firmware ou les avis publics les révèlent.

## Références

- [HackTricks: SSH default credentials](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [SecLists default credential lists](https://github.com/danielmiessler/SecLists)
