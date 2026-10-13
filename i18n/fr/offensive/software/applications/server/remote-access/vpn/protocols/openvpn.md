---
order: 2
title: "OpenVPN : attaques de configuration, d'informations d'identification et de gestion des clés"
description: "La cryptographie d'OpenVPN est solide, de sorte que les attaques visent sa configuration et sa gestion des clés : profils clients volés et clés intégrées, un HMAC tls-auth/tls-crypt manquant ou faible qui expose le canal de contrôle, une authentification par nom d'utilisateur/mot de passe sprayée contre le serveur, et des certificats clients et clés statiques réutilisés ou mal protégés."
keywords:
  - openvpn
  - tls-auth
  - profil client
  - clé statique
  - information d'identification
---

# OpenVPN

OpenVPN est bien considéré sur le plan cryptographique, de sorte que l'attaquer revient à attaquer la façon dont il est déployé plutôt que le protocole. La cible la plus riche est le profil client (`.ovpn`) : il intègre souvent la CA, le certificat client, la clé privée, et parfois les informations d'identification, de sorte qu'un profil volé authentifie directement l'attaquant. Le HMAC optionnel `tls-auth`/`tls-crypt` protège le canal de contrôle ; lorsqu'il est absent ou que sa clé fuit (elle aussi réside dans le profil), le canal de contrôle est exposé au sondage et à certaines attaques. Là où OpenVPN utilise l'authentification par nom d'utilisateur/mot de passe, le serveur est une cible de spraying. Et la réutilisation de clé statique ou de certificat entre clients fait qu'une seule clé récupérée ouvre le VPN.

```bash
# steal/examine a client profile for embedded keys and creds
grep -iE '<cert>|<key>|<tls-auth>|auth-user-pass' client.ovpn
openssl rsa -in client.key -noout -text 2>/dev/null     # confirm a usable private key
# reuse the profile to connect as that client
openvpn --config client.ovpn
# where username/password auth is used, spray the server (see password brute force)
```

## Notes d'exploitation

- Les profils clients sont le butin : cherchez les fichiers `.ovpn` (et `.key`/`.crt`) sur les terminaux, les sauvegardes, les images et les dépôts ; une clé privée et un certificat intégrés authentifient en tant que ce client sans aucune information d'identification supplémentaire.
- Un `tls-auth`/`tls-crypt` manquant, ou une clé HMAC fuitée (récupérable depuis un profil), supprime la protection du canal de contrôle qui bloque autrement le sondage non authentifié du serveur OpenVPN.
- Des clés partagées/statiques et des certificats clients réutilisés font qu'une seule information d'identification récupérée ouvre le VPN pour l'attaquant ; vérifiez si le déploiement émet des certificats par client ou en réutilise un seul.
- OpenVPN en nom d'utilisateur/mot de passe est sprayable comme tout portail ([brute force de mot de passe](../authentication/password-brute-force.md)) ; la réutilisation et le vol de certificat relèvent de l'[abus de certificat](../authentication/certificate-abuse.md).

## Références

- [Sécurité OpenVPN et tls-auth](https://openvpn.net/community-resources/hardening-openvpn-security/)
- [HackTricks: OpenVPN](https://book.hacktricks.xyz/network-services-pentesting/openvpn)
