---
title: "Aucune authentification : serveurs Telnet ne requérant aucun mot de passe"
description: "Certains services Telnet donnent directement un shell ou un menu de configuration sans login, ou acceptent n'importe quel nom d'utilisateur sans mot de passe. Fréquent sur les appareils embarqués, les interfaces de debug et les équipements mal configurés, un Telnet non authentifié accorde un accès terminal immédiat, souvent en tant que root ou dans un contexte administratif, à quiconque se connecte."
keywords:
  - aucune authentification
  - telnet ouvert
  - interface de debug
  - non authentifié
  - shell
---

# Aucune authentification

Un nombre surprenant de services Telnet ne requièrent aucune authentification : les appareils embarqués et les interfaces de debug donnent directement un shell, certains serveurs acceptent n'importe quel nom d'utilisateur sans mot de passe, et des équipements mal configurés exposent un menu de configuration sans login. Dans ce cas, se connecter est l'attaque, et le contexte est fréquemment privilégié (un shell root BusyBox, un menu d'administration de l'appareil, une interface de diagnostic). Les interfaces Telnet de debug/backdoor constructeur qui contournent l'authentification sont une classe récurrente sur les appareils grand public et industriels.

```bash
nc <target> 23        # connect; a shell/menu with no login prompt => no auth
# some accept any user, empty password:
(printf 'anyuser\n\n'; sleep 1; printf 'id\n') | nc <target> 23
# scan a range for open/no-auth telnet
nmap -p23 --open <range>
```

## Notes d'exploitation

- L'indice est l'absence (ou le non-respect) d'une invite de login : un shell direct, un menu, ou l'acceptation de n'importe quel nom d'utilisateur avec un mot de passe vide.
- Les interfaces de debug embarquées et les backdoors constructeur sont la source courante ; elles se trouvent souvent aussi sur des ports non standards, donc scannez au-delà de 23 sur les équipements.
- L'accès est généralement privilégié (root/admin sur l'appareil), de sorte qu'un Telnet non authentifié équivaut généralement à un contrôle total immédiat de l'appareil.
- Là où un login est imposé, repliez-vous sur les [identifiants par défaut](default-credentials.md) et la [force brute](password-brute-force.md).

## Références

- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
