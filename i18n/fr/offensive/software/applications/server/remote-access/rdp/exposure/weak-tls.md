---
title: "TLS faible : faiblesses du transport RDP permettant le MITM"
description: "RDP peut utiliser l'ancienne sécurité RDP native (basée sur RC4, avec une faiblesse MITM connue) ou un TLS potentiellement mal configuré avec des chiffrements faibles et un certificat auto-signé non approuvé. Un attaquant bien positionné exploite la faiblesse de la sécurité native ou un TLS faible/non vérifié pour s'interposer en machine du milieu dans la connexion, capturant les identifiants et le contenu de la session."
keywords:
  - rdp tls
  - native rdp security
  - rc4
  - mitm
  - self-signed certificate
---

# TLS faible

La sécurité du transport de RDP varie, et les configurations faibles permettent l'attaque de la machine du milieu. L'ancienne « sécurité RDP native » utilise RC4 avec une faiblesse de conception qui permettait un MITM RDP classique (la clé publique du serveur n'est pas authentifiée, donc un attaquant s'interpose). L'option moderne est TLS, mais elle est fréquemment affaiblie : certificats auto-signés par défaut que les clients sont habitués à accepter, versions de protocole obsolètes et suites de chiffrement faibles. Dans tous les cas, un attaquant bien positionné qui exploite l'absence d'authentification du serveur dans la sécurité native, ou un client qui ne vérifie pas le certificat TLS, s'interpose pour capturer les identifiants et la session.

```bash
# determine the security layer and TLS posture
nmap -p3389 --script rdp-enum-encryption <target>      # native RDP security vs TLS/CredSSP
nmap -p3389 --script ssl-enum-ciphers <target>         # weak ciphers/protocols if TLS
openssl s_client -connect <target>:3389 -starttls rdp 2>/dev/null | openssl x509 -noout -issuer -subject
# with an on-path position, run an RDP MITM that presents a substitute cert/key
#   (tools like Seth automate RDP downgrade + MITM against native security / unverified TLS)
```

## Notes d'exploitation

- La « RDP Security » native est le constat le plus fort ici : sa couche basée sur RC4 n'authentifie pas le serveur, donc un attaquant sur le chemin réalise un MITM complet, par rétrogradation et interception, capturant les identifiants en clair saisis à l'ouverture de session.
- Pour un RDP protégé par TLS, la faiblesse est généralement la confiance : des certificats auto-signés par défaut que les utilisateurs valident d'un clic, permettant un certificat présenté par l'attaquant ; l'acceptation de chiffrements/protocoles faibles est une préoccupation secondaire.
- Le bénéfice pratique d'un MITM RDP est la capture d'identifiants (l'utilisateur s'authentifie auprès de l'attaquant) plus l'observation de la session ; cela nécessite une position réseau (ARP/DNS/route).
- Énumérez d'abord le niveau de sécurité ([paramètres de sécurité](../enumeration/security-settings.md)) ; c'est la sécurité native ou un TLS non vérifié qui rend le MITM viable.

## Outils

- [Seth (RDP MITM)](https://github.com/SySS-Research/Seth)

## Références

- [SySS: RDP MITM research](https://www.syss.de/en/)
- [Microsoft: RDP security layers](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
