---
title: "TLS faible et validation de certificat : intercepter FTPS"
description: "FTPS ne protège FTP que lorsque son TLS est robuste et que le client vérifie le certificat du serveur. Les serveurs proposant des versions de protocole obsolètes ou des suites de chiffrement faibles, autorisant un repli en clair, ou les clients qui ignorent la validation de certificat permettent à un attaquant en position réseau de rétrograder ou d'intercepter la connexion, récupérant les identifiants et les données que FTPS était censé protéger."
keywords:
  - ftps tls
  - certificate validation
  - downgrade
  - man in the middle
  - starttls
---

# TLS faible et validation de certificat

La sécurité de FTPS ne vaut que celle de son TLS, et plusieurs faiblesses courantes l'annulent pour un attaquant en position réseau. Un serveur proposant des versions de TLS obsolètes ou des suites de chiffrement faibles peut être forcé vers une session cassable. Un client qui ne vérifie pas le certificat du serveur accepte le certificat d'un attaquant, permettant une attaque de l'homme du milieu directe. Et un FTPS explicite qui autorise un repli en clair (ou dont `AUTH TLS` peut être supprimé) permet à une attaque de rétrogradation de ramener la session en clair. Chacune de ces faiblesses récupère les identifiants et les données de fichiers que FTPS est censé chiffrer.

```bash
# assess the server's TLS posture
nmap -p990,21 --script ssl-enum-ciphers <target>     # weak protocols/ciphers offered?
openssl s_client -connect <target>:990 -tls1          # does it accept obsolete TLS?
# explicit FTPS downgrade: strip AUTH TLS so the client stays plaintext (MITM position)
#   intercept the control channel and remove/deny the AUTH TLS capability/command
# certificate check: a self-signed or unverified cert accepted by the client => MITM
openssl s_client -connect <target>:990 | openssl x509 -noout -issuer -subject
```

## Notes d'exploitation

- L'attaque nécessite une position réseau (on-path ou via empoisonnement de la résolution de noms) ; de là, un client qui ignore la validation de certificat est trivialement intercepté avec un certificat présenté par l'attaquant.
- Le FTPS explicite (`AUTH TLS` sur 21) est la variante la plus interceptable car la session débute en clair ; supprimer ou refuser la bascule TLS la maintient en clair si le client le tolère.
- L'acceptation de suites faibles ou de protocoles obsolètes permet de déchiffrer une session par ailleurs protégée par TLS ; `ssl-enum-ciphers` note ce que le serveur autorise.
- Le gain est le même que l'interception d'un FTP simple : identifiants capturés (souvent des comptes système réutilisables) et contenus de fichiers ; les problèmes au niveau FTP (anonyme, brute force) subsistent indépendamment de TLS.

## Références

- [RFC 4217 : securing FTP with TLS](https://datatracker.ietf.org/doc/html/rfc4217)
- [nmap ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)
