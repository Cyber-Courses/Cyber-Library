---
title: "UnPAC-the-hash : récupérer le hash NT depuis PKINIT"
order: 7
description: "Récupérer le hash NT d'un compte depuis une authentification Kerberos PKINIT (par certificat) en lisant le NTLM_SUPPLEMENTAL_CREDENTIAL que le KDC renvoie dans le PAC, transformant un accès par certificat en un hash réutilisable."
keywords:
  - UnPAC the hash
  - PKINIT
  - NTLM_SUPPLEMENTAL_CREDENTIAL
  - certificate
  - NT hash
---

# UnPAC-the-hash

Lorsqu'un compte s'authentifie avec un **certificat** via PKINIT, le KDC a tout de même besoin de prendre en charge le NTLM hérité pour cet utilisateur, il renvoie donc le **hash NT** du compte à l'intérieur du PAC, dans une structure `NTLM_SUPPLEMENTAL_CREDENTIAL` chiffrée, afin que NTLM puisse fonctionner après une ouverture de session par certificat. **UnPAC-the-hash** se contente de lire ce hash dans l'échange de TGT. Il convertit tout accès par certificat en un hash NT réutilisable du compte, sans aucun cassage.

## La technique

```bash
# PKINITtools: PKINIT with the certificate, then decrypt the PAC credential
gettgtpkinit.py -cert-pfx cred.pfx example.local/victim victim.ccache
export KRB5CCNAME=victim.ccache
getnthash.py -key <AS-REP-key-from-step-1> example.local/victim

# Certipy does both in one step
certipy auth -pfx victim.pfx -username victim -domain example.local
```

`gettgtpkinit.py` affiche la clé de chiffrement de l'AS-REP ; `getnthash.py` l'utilise pour déchiffrer le credential supplémentaire et récupérer le hash NT.

## D'où vient le certificat

Tout chemin qui produit un certificat pour le compte alimente UnPAC :

- **[Shadow credentials](shadow-credentials.md)** : ajouter un key credential, obtenir un certificat, faire un UnPAC du hash.
- **Enrôlement AD CS / abus ESC** : un certificat obtenu depuis un template vulnérable.
- **Relais vers AD CS (ESC8)** : un certificat pour une machine ou un utilisateur forcé.

## Notes d'exploitation

- Le gain est un **hash NT persistant et réutilisable** à partir de ce qui pourrait autrement être un certificat à durée limitée, permettant le [pass-the-hash](../ntlm/pass-the-hash.md) et l'[overpass-the-hash](pass-the-key-and-overpass-the-hash.md) longtemps après la fin du travail sur le certificat.
- C'est la suite habituelle des shadow credentials et de tout certificat AD CS : obtenir le certificat, faire un UnPAC vers un hash, se déplacer latéralement avec le hash.
- Fonctionne parce que le KDC inclut du matériel NTLM pour l'interopérabilité ; cela ne nécessite aucun droit supplémentaire au-delà de la détention du certificat.

## Outils

- **Certipy** (`auth`) : PKINIT plus récupération du hash en une seule commande.
- **PKINITtools** (`gettgtpkinit.py`, `getnthash.py`) : les deux étapes explicites.

## Références

- [PKINITtools (dirkjanm): gettgtpkinit and getnthash](https://github.com/dirkjanm/PKINITtools)
- [Certipy (ly4k): auth and hash recovery](https://github.com/ly4k/Certipy)
- [SpecterOps: Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
