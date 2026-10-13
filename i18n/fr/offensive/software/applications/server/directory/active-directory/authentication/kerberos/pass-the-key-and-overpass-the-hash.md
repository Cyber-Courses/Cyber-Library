---
order: 2
title: "Pass-the-key et overpass-the-hash : transformer une clé en ticket"
description: "Demander un TGT Kerberos directement à partir de la clé secrète d'un compte (hash NT ou clé AES) au lieu de son mot de passe en clair, de sorte qu'un hash récupéré devienne une identité Kerberos complète (overpass-the-hash / pass-the-key)."
keywords:
  - overpass the hash
  - pass the key
  - clé AES
  - TGT
  - PKINIT
---

# Pass-the-key et overpass-the-hash

Kerberos n'a jamais besoin du mot de passe en clair : l'échange AS repose sur la **clé secrète** du compte, qui est dérivée du mot de passe (clé RC4 = le hash NT ; clés AES = PBKDF2 sur le mot de passe). Donc si vous détenez le hash NT ou une clé AES, vous pouvez demander un TGT légitime et devenir ce compte dans Kerberos. Lorsque l'entrée est le hash NT, on parle d'**overpass-the-hash** ; lorsque c'est la clé AES, de **pass-the-key**. C'est le pont entre un secret récupéré ([dumping](../credentials/lsass-dumping.md), [DCSync](../credentials/ntds-and-dcsync.md)) et un déplacement Kerberos complet.

## Demander le TGT

```bash
# Overpass-the-hash: NT (RC4) hash -> TGT
getTGT.py -hashes :<nthash> example.local/user
# Pass-the-key: AES256 key -> TGT (stealthier where RC4 is monitored/disabled)
getTGT.py -aesKey <aes256key> example.local/user

export KRB5CCNAME=user.ccache
wmiexec.py -k -no-pass example.local/user@<host>        # use the ticket

# Rubeus, on-host
Rubeus.exe asktgt /user:user /rc4:<nthash> /ptt
Rubeus.exe asktgt /user:user /aes256:<aes256key> /ptt
```

Le TGT résultant est indiscernable de celui d'une connexion normale : c'est un ticket légitime, simplement obtenu à partir de la clé plutôt que du mot de passe.

## Pourquoi choisir ceci plutôt que le pass-the-hash

Le [pass-the-hash](../ntlm/pass-the-hash.md) réutilise le hash NT via **NTLM**. L'overpass-the-hash convertit le même hash en **Kerberos**, ce qui importe lorsque :

- La cible n'accepte que Kerberos (NTLM désactivé ou bloqué).
- Vous voulez vous déplacer avec des tickets (pass-the-ticket, abus de délégation) plutôt qu'avec une authentification NTLM.
- Vous voulez être plus discret : une connexion Kerberos se fond mieux dans le trafic normal qu'une exécution NTLM dans certains environnements.

## RC4 contre AES

- Fournir la clé **RC4** (le hash NT) force un échange chiffré en RC4, qui peut ressortir là où le domaine est en AES uniquement ou là où le downgrade RC4 est surveillé.
- Fournir la clé **AES256** produit un échange AES qui correspond aux clients normaux. Récupérez les clés AES depuis LSASS (`sekurlsa::ekeys`) ou depuis un dump NTDS `-just-dc` et préférez-les.

## Notes d'exploitation

- L'overpass-the-hash combiné à la clé AES est le choix furtif par défaut pour réutiliser un secret de machine ou de service récupéré en tant que Kerberos.
- Un TGT obtenu de cette façon pilote tout ce qui est en aval : [pass-the-ticket](pass-the-ticket.md), abus de [délégation](delegation/index.md) S4U et demandes de tickets de service.
- Le décalage d'horloge compte : Kerberos rejette les tickets hors d'une fenêtre d'environ 5 minutes, synchronisez-vous donc sur l'heure du DC avant de demander.

## Outils

- **Impacket** (`getTGT.py`, `-k`) : demander et utiliser des TGT à partir d'un hash ou d'une clé AES.
- **Rubeus** (`asktgt`) : demande de TGT sur l'hôte avec `/rc4`, `/aes256`, `/ptt`.
- **Mimikatz** (`sekurlsa::ekeys`) : récupérer les clés AES depuis LSASS pour le pass-the-key.

## Références

- [GhostPack Rubeus (asktgt)](https://github.com/GhostPack/Rubeus)
- [Impacket getTGT](https://github.com/fortra/impacket)
- [Microsoft: Kerberos authentication](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)
