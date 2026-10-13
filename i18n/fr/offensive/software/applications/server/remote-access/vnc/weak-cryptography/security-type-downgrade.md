---
title: "Downgrade du type de sécurité : forcer un type de sécurité VNC plus faible"
description: "Dans RFB 3.7 et au-delà, le serveur propose une liste de types de sécurité et le client en choisit un. Un attaquant bien positionné interceptant la négociation retire les types forts de la liste du serveur ou force le client à choisir None ou l'authentification VNC faible, dégradant la session vers aucune ou une authentification faible et déjouant des options configurées plus robustes."
keywords:
  - downgrade du type de sécurité
  - rfb 3.7
  - négociation
  - none
  - mitm
---

# Downgrade du type de sécurité

À partir du protocole RFB 3.7, le serveur envoie une liste de types de sécurité pris en charge et le client en sélectionne un. Cette négociation n'est pas authentifiée et, sur un canal non chiffré, modifiable en transit, ce qui permet un downgrade. Un attaquant en machine-in-the-middle retire les types forts (VeNCrypt/TLS, voire l'authentification VNC) de la liste proposée par le serveur avant qu'elle n'atteigne le client, ne laissant que None ou le schéma de mot de passe VNC faible, de sorte que le client poursuit avec aucune ou une authentification faible. Cela déjoue un serveur configuré pour exiger une sécurité plus robuste, car rien n'authentifie la liste des types.

```bash
# from an on-path position, intercept the RFB handshake and rewrite the security-type
# list the server offers, removing VeNCrypt(19)/strong types, leaving None(1) or VNC(2)
# the client then negotiates the weak/none type, and the session proceeds unprotected
# (combine with ARP/DNS/route positioning; the 3.3 protocol has the server dictate the
#  single type, so downgrade applies to 3.7+ where the client chooses from a list)
```

## Notes d'exploitation

- Le downgrade fonctionne parce que la négociation du type de sécurité n'est ni chiffrée ni protégée en intégrité sur un canal en clair ; l'attaquant édite la liste proposée en transit.
- Retirer VeNCrypt/TLS de la liste fait retomber la session sur une authentification VNC non chiffrée ou sur None, la réexposant à la capture en [clair](cleartext-transmission.md) et au [cassage de mot de passe](../authentication/password-brute-force.md).
- Cela nécessite une position sur le chemin et s'applique à RFB 3.7+ (où le client choisit dans une liste) ; les serveurs RFB 3.3 dictent l'unique type, de sorte qu'il n'y a rien à dégrader là, seulement à intercepter.
- C'est le complément actif à la lecture passive d'une session en clair : il force la faiblesse là où le serveur proposait de la robustesse.

## Références

- [RFC 6143 : négociation du type de sécurité du protocole 3.7](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.2)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
