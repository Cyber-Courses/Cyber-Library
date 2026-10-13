---
order: 1
title: "Authentification : attaquer le contrôle d'accès VNC"
description: "Le contrôle d'accès VNC est fréquemment faible : des serveurs tournent sans authentification, avec des mots de passe constructeur ou devinables, ou avec le schéma VNC classique dont le mot de passe DES de huit caractères peut être forcé par force brute. Des implémentations spécifiques ajoutent de véritables contournements d'authentification. Obtenir l'accès donne un bureau interactif complet sur la cible."
keywords:
  - authentification vnc
  - aucune authentification
  - mot de passe par défaut
  - force brute
  - contournement d'auth
---

# Authentification

L'authentification VNC est faible de plusieurs façons indépendantes, et n'importe laquelle donne un bureau interactif complet. Les serveurs sont couramment laissés avec le type de sécurité « None » (aucune authentification). Le schéma de mot de passe VNC classique est un challenge-response DES sur un mot de passe tronqué à huit caractères, qui peut être forcé par force brute en ligne et cassé hors ligne à partir d'un challenge capturé, et de tels mots de passe sont souvent des valeurs par défaut constructeur ou triviales. Et des implémentations particulières (RealVNC, TightVNC) ont eu des contournements d'authentification qui sautent entièrement la vérification. Le gain est le même quoi qu'il arrive : le contrôle interactif du bureau distant.

```bash
vncviewer <target>::5900                       # prompts (or not) depending on security type
nmap -p5900 --script vnc-info,realvnc-auth-bypass <target>
```

## Sous-thèmes

- **[Aucune authentification](no-authentication.md)** : serveurs proposant le type de sécurité None.
- **[Mots de passe par défaut](default-passwords.md)** : mots de passe VNC constructeur et devinables.
- **[Force brute de mot de passe](password-brute-force.md)** : attaques en ligne et hors ligne sur le mot de passe VNC.
- **[Contournement d'auth RealVNC](realvnc-auth-bypass.md)** : le contournement de type de sécurité RealVNC.
- **[Contournement TightVNC](tightvnc-bypass.md)** : failles d'authentification TightVNC.

## Références

- [RFC 6143: security](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
