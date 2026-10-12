---
title: "Paramètres de sécurité : NLA, chiffrement et posture du certificat RDP"
description: "Les paramètres de sécurité de RDP décident de la façon dont il peut être attaqué : si l'authentification au niveau du réseau est requise (forçant les identifiants avant une session), quel niveau de sécurité et de chiffrement est négocié (sécurité RDP native versus TLS), et le certificat du serveur. Les énumérer sépare les hôtes durcis de ceux ouverts à une connexion non authentifiée et à l'exploitation en pré-authentification."
keywords:
  - nla
  - encryption level
  - security layer
  - tls certificate
  - rdp configuration
---

# Paramètres de sécurité

La façon dont un hôte RDP est configuré détermine quelles attaques sont même possibles, c'est pourquoi énumérer ses paramètres de sécurité est décisif. Trois choses comptent. L'authentification au niveau du réseau (NLA, utilisant CredSSP) exige que le client s'authentifie avant toute création de session ; avec la NLA désactivée, un client atteint la surface d'ouverture de session (et historiquement le code vulnérable en pré-authentification) sans identifiants. Le niveau de sécurité et de chiffrement révèle si l'hôte utilise l'ancienne sécurité RDP native (faible) ou TLS, et avec quelle robustesse. Et le certificat TLS nomme l'hôte et le date. Ensemble, ces éléments séparent un hôte durci, imposant la NLA et en TLS, d'un hôte ancien, NLA désactivée, ouvert à la connexion non authentifiée et aux bugs de pré-authentification.

```bash
# security layer, encryption level, and NLA requirement
nmap -p3389 --script rdp-enum-encryption <target>
#   reports: Security layer (RDP/SSL/CredSSP/Hybrid), Encryption level, NLA supported/required
# inspect the TLS certificate (host identity, validity, key strength)
openssl s_client -connect <target>:3389 -starttls rdp 2>/dev/null | openssl x509 -noout -subject -dates
```

## Notes d'exploitation

- Le statut de la NLA est le constat pivot : NLA requise signifie que des identifiants sont nécessaires avant une session, ce qui émousse l'exploitation en pré-authentification et fait de la force brute le chemin ; NLA non requise expose la surface de pré-authentification et laisse un client atteindre l'écran d'ouverture de session sans authentification.
- Le niveau de sécurité raconte l'histoire cryptographique : la « RDP Security » native (basée sur RC4) est faible et sujette au MITM, tandis que CredSSP/TLS est plus robuste ; un niveau hérité signale un hôte ancien et probablement non corrigé.
- Le caractère auto-signé par défaut du certificat, son sujet (nom d'hôte) et ses dates de validité corroborent l'ancienneté de l'OS et l'identité de l'hôte ; un certificat expiré depuis longtemps ou par défaut signale une négligence.
- À combiner avec le [numéro de build](banner-grabbing.md) : un ancien build plus une NLA désactivée est une cible de choix pour une [RCE en pré-authentification](../pre-authentication-flaws/index.md) ; un build moderne avec NLA requise vous oriente vers l'[authentification](../authentication/index.md) et l'[exposition](../exposure/index.md).

## Références

- [nmap rdp-enum-encryption](https://nmap.org/nsedoc/scripts/rdp-enum-encryption.html)
- [Microsoft: Network Level Authentication](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
