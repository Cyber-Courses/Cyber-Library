---
order: 1
title: "IPsec IKE : divulgation de PSK en mode agressif et faiblesses IKE"
description: "L'échange de clés IPsec via IKE expose plusieurs attaques : le mode agressif divulgue un hash de la clé pré-partagée à un initiateur non authentifié en vue d'un cassage hors ligne, la passerelle peut être identifiée et énumérée via IKE, et des transformations faibles (petits groupes DH, DES, SHA-1) négociées en Phase 1 affaiblissent le tunnel. XAUTH, lorsqu'il est ajouté, est une cible d'information d'identification supplémentaire."
keywords:
  - ipsec
  - ike
  - mode agressif
  - clé pré-partagée
  - xauth
---

# IPsec IKE

IPsec utilise IKE (sur UDP 500, avec NAT-traversal sur 4500) pour authentifier les pairs et établir les clés, et IKE est la principale surface d'attaque. La faiblesse emblématique est le mode agressif : pour achever la Phase 1 en moins de messages, il envoie en clair un hash dérivé de la clé pré-partagée à quiconque initie, de sorte qu'un attaquant non authentifié capture ce hash et casse la PSK hors ligne. IKE identifie aussi la passerelle (ID de fournisseur, ensembles de transformations), révèle la prise en charge de cryptographie faible, et, là où XAUTH ajoute un nom d'utilisateur/mot de passe par-dessus la PSK, présente une seconde information d'identification à attaquer. Le mode principal ne divulgue pas le hash de la PSK, de sorte que la prise en charge des modes par la passerelle importe.

```bash
# fingerprint and detect aggressive mode
ike-scan -M <target>                            # main-mode transforms + vendor ID
ike-scan -A -M -P psk.hash <target>             # aggressive mode; -P captures the PSK hash
# crack the captured PSK offline
psk-crack -d wordlist.txt psk.hash
# with the PSK, attack XAUTH credentials if required, then establish the tunnel
```

## Notes d'exploitation

- Le mode agressif est le constat clé : `ike-scan -A -P` capture le hash dérivé de la PSK à partir d'une négociation non authentifiée, et `psk-crack` récupère une clé faible hors ligne, voir [clés pré-partagées faibles](../authentication/weak-pre-shared-keys.md).
- L'empreinte IKE (ID de fournisseur, transformations) identifie l'appareil et sa prise en charge de cryptographie faible, alimentant le [downgrade de l'échange de clés](../weak-cryptography/key-exchange-downgrade.md) et les exploits spécifiques à l'appareil.
- XAUTH ajoute un nom d'utilisateur/mot de passe après l'authentification PSK ; récupérez la PSK, puis pulvérisez les informations d'identification XAUTH ([brute force de mot de passe](../authentication/password-brute-force.md)).
- L'énumération d'ID de groupe/d'utilisateurs est possible sur certaines passerelles via les réponses IKE ; combinez avec la PSK récupérée pour compléter le tunnel vers le réseau interne.

## Outils

- [ike-scan / psk-crack](https://github.com/royhills/ike-scan)

## Références

- [HackTricks: IPsec/IKE aggressive mode](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
- [RFC 2409 (IKE)](https://datatracker.ietf.org/doc/html/rfc2409)
