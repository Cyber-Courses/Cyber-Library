---
order: 3
title: "Énumération : identification d'un service SSH"
description: "L'énumération SSH collecte ce qui oriente chaque attaque ultérieure : la bannière de version qui identifie l'implémentation et ses vulnérabilités, les algorithmes proposés qui révèlent l'exposition à la crypto faible, l'empreinte de clé d'hôte et les noms d'utilisateur valides. Rien ne nécessite d'authentification, et les résultats sélectionnent quelle voie d'authentification, de crypto ou d'exploit s'applique."
keywords:
  - énumération ssh
  - bannière
  - algorithmes
  - énumération d'utilisateurs
  - clé d'hôte
---

# Énumération

Avant d'attaquer SSH, énumérez-le : la bannière de version nomme l'implémentation et la build (reliées aux vulnérabilités connues), les algorithmes d'échange de clés, de chiffrement et de MAC proposés révèlent l'exposition à la crypto faible, l'empreinte de clé d'hôte identifie le serveur (et permet les attaques de confiance), et les noms d'utilisateur valides focalisent les attaques d'identifiants. Tout cela est non authentifié, et chaque résultat pointe vers une suite précise : une version vulnérable à exploiter, des algorithmes faibles à cibler, ou une liste d'utilisateurs à sprayer.

```bash
nmap -p22 -sV --script ssh2-enum-algos,ssh-hostkey,ssh-auth-methods <target>
nc <target> 22                                 # raw banner
```

## Sous-thèmes

- **[Récupération de bannière](banner-grabbing.md)** : divulgation de version et d'implémentation.
- **[Énumération d'algorithmes](algorithm-enumeration.md)** : crypto proposée et détection d'options faibles.
- **[Énumération d'utilisateurs](user-enumeration.md)** : découverte de noms d'utilisateur valides.

## Références

- [HackTricks: SSH enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [nmap SSH scripts](https://nmap.org/nsedoc/)
