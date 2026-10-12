---
title: "Abus de certificats : certificats volés et contournement de la validation"
description: "Les VPN qui s'authentifient avec des certificats clients sont attaqués en volant le certificat et sa clé privée à un client (exportés depuis le magasin, récupérés depuis la configuration ou les sauvegardes) puis en les réutilisant, et en exploitant les passerelles qui valident faiblement les certificats, acceptant des certificats auto-signés, expirés ou d'une mauvaise AC, pour s'authentifier sans certificat légitime."
keywords:
  - certificat vpn
  - certificat client
  - vol de clé privée
  - contournement de validation
  - mutual tls
---

# Abus de certificats

L'authentification VPN par certificat repose sur le fait que le client détient une clé privée que la passerelle approuve, et sur le fait que la passerelle valide correctement le certificat, deux points attaquables. Vol : un certificat client et sa clé privée sont exportés depuis le magasin de certificats Windows/macOS, récupérés depuis la configuration ou le profil d'un client VPN, ou extraits de sauvegardes et d'images ; avec eux, l'attaquant s'authentifie en tant que ce client. Contournement de la validation : une passerelle qui ne vérifie pas correctement l'AC émettrice, la fenêtre de validité, la révocation ou le lien certificat-identité peut être satisfaite par un certificat auto-signé, expiré ou autrement illégitime.

```bash
# steal a client certificate + key from a client you have access to
#   Windows: export from the cert store (if marked exportable) or dump with mimikatz/crypto APIs
#   files: VPN client profiles, .p12/.pfx, .ovpn embedded certs, backups
openssl pkcs12 -in client.pfx -nodes -out client.pem      # extract cert+key
# reuse it to authenticate the VPN
openvpn --config client.ovpn --cert client.crt --key client.key
# validation bypass: present a self-signed/wrong-CA cert where the gateway fails to verify
```

## Notes d'exploitation

- Le vol est la voie la plus courante : cherchez les fichiers `.pfx`/`.p12`, les `.ovpn` avec clés intégrées et les profils clients sur les terminaux compromis, dans les sauvegardes et dans les images ; une clé privée exportable permet de s'authentifier en tant que son propriétaire.
- Le certificat correspond souvent à une identité utilisateur/appareil spécifique sur la passerelle, si bien qu'un certificat volé peut aussi satisfaire les contrôles de posture de l'appareil, et pas seulement l'authentification.
- Les failles de contournement de la validation (ignorer l'AC, la validité, la révocation ou le lien d'identité) permettent à un certificat généré par l'attaquant de s'authentifier ; testez si la passerelle accepte un certificat client auto-signé ou d'une mauvaise AC.
- Combinez avec le [brute force de mots de passe](password-brute-force.md) lorsque le VPN utilise un certificat plus un mot de passe ; un certificat volé prend en charge un facteur.

## Références

- [OpenVPN PKI and client certs](https://openvpn.net/community-resources/)
- [HackTricks: VPN certificates](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
