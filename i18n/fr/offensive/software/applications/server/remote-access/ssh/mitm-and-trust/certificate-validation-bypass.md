---
title: "Contournement de validation de certificat : abus de la confiance SSH basée sur les certificats"
description: "Le SSH basé sur les certificats remplace l'épinglage par hôte de known_hosts par une CA qui signe des certificats d'hôte et d'utilisateur. Les faiblesses, une clé de CA compromise ou à portée trop large, des vérifications de principaux et de validité manquantes, l'acceptation de certificats auto-signés ou issus de la mauvaise CA, permettent à un attaquant de présenter un certificat d'hôte de confiance pour le machine-in-the-middle ou de forger des certificats d'utilisateur pour l'authentification."
keywords:
  - certificat ssh
  - clé de ca
  - principaux
  - autorité de certification
  - confiance
---

# Contournement de validation de certificat

Les déploiements SSH plus importants remplacent la gestion par hôte de `known_hosts` par des certificats : une autorité de certification signe des certificats d'hôte (donc les clients font confiance à tout hôte dont la clé a été signée par la CA) et des certificats d'utilisateur (donc les serveurs acceptent tout utilisateur dont la clé a été signée par la CA, dans les principaux et la validité indiqués). Cela met la confiance à l'échelle mais la concentre dans la CA, et les faiblesses du modèle ou de sa validation constituent la surface d'attaque. Une clé de signature de CA compromise ou à portée trop large permet à un attaquant de forger des certificats d'hôte de confiance (pour le MITM) ou des certificats d'utilisateur (pour l'authentification) ; et les clients ou serveurs qui ne vérifient pas les principaux, les fenêtres de validité ou la bonne CA acceptent des certificats qu'ils devraient rejeter.

```bash
# inspect a certificate's scope and validity
ssh-keygen -L -f id_ed25519-cert.pub           # principals, validity, CA, options
# with a compromised CA signing key, mint a host cert for MITM or a user cert for auth
ssh-keygen -s ca_key -I attacker -h -n <target-hostname> host_key.pub          # host cert
ssh-keygen -s ca_key -I attacker -n root -V +1d user_key.pub                   # user cert as root
ssh -i user_key -o CertificateFile=user_key-cert.pub root@<target>
```

## Notes d'exploitation

- La clé de signature de la CA est le joyau de la couronne : sa possession permet de forger à la fois des certificats d'hôte (présenter une clé de confiance pour le [SSH MITM](ssh-mitm.md)) et des certificats d'utilisateur (s'authentifier en tant que n'importe quel principal que le serveur accepte), donc chassez la clé de la CA comme vous le feriez pour n'importe quelle clé privée de grande valeur.
- Les lacunes de validation permettent des attaques sans la clé de la CA : un serveur qui n'applique pas la liste `principals`, ignore la fenêtre de validité ou fait confiance à la mauvaise CA accepte des certificats qu'il ne devrait pas ; testez avec un certificat hors principal ou expiré.
- Les certificats d'utilisateur forgés pour un principal privilégié (`root`, un compte admin) sont une authentification directe, bornée uniquement par ce que les `TrustedUserCAKeys`/principaux du serveur autorisent.
- `ssh-keygen -L` révèle les principaux, la validité et les options d'un certificat, les faits qui déterminent si un certificat donné est accepté et où.

## Références

- [OpenSSH certificate authentication (ssh-keygen -s)](https://man.openbsd.org/ssh-keygen#CERTIFICATES)
- [Facebook/Uber SSH CA design writeups](https://engineering.fb.com/2016/09/12/security/scalable-and-secure-access-with-ssh/)
