---
order: 5
title: "Telnet : attaquer le protocole de terminal non chiffré"
description: "Telnet sur le port TCP 23 fournit un accès terminal distant sans chiffrement, de sorte que les identifiants, les commandes et la sortie traversent le réseau en clair. Sa surface est une authentification faible (valeurs par défaut, absence d'auth, force brute), une bannière verbeuse qui divulgue l'OS, une RCE par corruption mémoire dans d'anciennes implémentations telnetd, et l'interception de la session en clair pour les identifiants et le hijacking."
keywords:
  - telnet
  - port 23
  - clair
  - telnetd
  - terminal
---

# Telnet

Telnet est un protocole de terminal distant sur le port TCP 23 sans aucun chiffrement : le login, le mot de passe, chaque commande et toute la sortie circulent en clair. Il persiste sur les systèmes hérités, l'équipement réseau et les appareils IoT. La surface d'attaque est en conséquence large : authentification faible (valeurs par défaut constructeur, absence d'authentification, et force brute contre le login en clair), une bannière souvent verbeuse qui divulgue l'OS et l'appareil, une exécution de code à distance par corruption mémoire dans d'anciennes implémentations `telnetd`, et, parce que le canal est en clair, une interception directe des identifiants, des commandes et même du hijacking de session en direct pour un attaquant positionné.

```bash
nmap -p23 -sV --script telnet-ntlm-info,telnet-encryption <target>
nc <target> 23                                 # banner and login prompt
```

## Sous-thèmes

- **[Énumération](enumeration/index.md)** : bannière et détection d'OS.
- **[Authentification](authentication/index.md)** : valeurs par défaut, absence d'auth et force brute.
- **[Corruption mémoire](memory-corruption/index.md)** : RCE dans les implémentations telnetd.
- **[Interception de trafic](traffic-interception/index.md)** : capture en clair et hijacking de session.

## Références

- [RFC 854 (Telnet protocol)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet (23)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
