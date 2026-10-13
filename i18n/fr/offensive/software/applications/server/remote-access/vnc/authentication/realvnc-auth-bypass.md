---
title: "Contournement d'authentification RealVNC : forcer le type de sécurité None"
description: "Une faille de RealVNC permettait à un client d'imposer le type de sécurité pendant la négociation RFB, de sorte qu'un client malveillant sélectionnait None même lorsque le serveur exigeait un mot de passe. Le serveur respectait le choix du client et accordait une session non authentifiée, un contournement complet de l'authentification sur les versions de RealVNC affectées."
keywords:
  - realvnc
  - contournement d'authentification
  - type de sécurité
  - none
  - handshake
---

# Contournement d'authentification RealVNC

RealVNC a livré une faille dans la négociation du type de sécurité RFB : le protocole prévoit que le serveur propose un ensemble de types de sécurité et que le client en choisisse un parmi ceux proposés, mais les versions de RealVNC affectées laissaient le client sélectionner un type que le serveur n'avait pas proposé, en l'occurrence « None » (type 1). Un client malveillant, après avoir reçu la liste du serveur, annonçait simplement qu'il utiliserait None, et le serveur accordait alors une session non authentifiée, en sautant entièrement le mot de passe. Il s'agit d'un contournement pré-authentification complet sur les versions affectées, donnant un bureau interactif sans aucun identifiant.

```bash
# detect affected RealVNC servers
nmap -p5900 --script realvnc-auth-bypass <target>     # reports vulnerable + grants a session
# the exploit: complete the RFB handshake but respond with security type 1 (None)
# even when the server offered only type 2 (VNC auth); affected servers accept it.
```

## Notes d'exploitation

- Le bug est que le serveur fait confiance au type de sécurité choisi par le client sans vérifier qu'il a bien été proposé ; l'attaque consiste à forcer None là où le serveur exigeait un mot de passe, de sorte que le correctif a consisté à valider la sélection du client.
- `realvnc-auth-bypass` détecte et démontre la faille en obtenant une session ; un résultat positif correspond à un accès immédiat et non authentifié au bureau.
- Cette faille est spécifique aux builds de RealVNC affectés ; identifiez d'abord l'[implémentation](../enumeration/implementation-detection.md) et la version.
- C'est l'accès VNC le plus propre lorsqu'il est présent, sans brute force ni capture nécessaire ; là où il n'est pas présent, repliez-vous sur les voies du mot de passe et de l'absence d'authentification.

## Références

- [Avis sur le contournement du type de sécurité RealVNC](https://www.realvnc.com/en/connect/docs/)
- [nmap realvnc-auth-bypass](https://nmap.org/nsedoc/scripts/realvnc-auth-bypass.html)
