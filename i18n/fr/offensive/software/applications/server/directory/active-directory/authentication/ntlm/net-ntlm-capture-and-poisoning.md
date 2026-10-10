---
title: "Capture et empoisonnement Net-NTLM : récolter l'authentification sur le réseau"
order: 1
description: "Capturer les défis-réponses NetNTLMv2 en empoisonnant la résolution de noms par diffusion (LLMNR, NBT-NS, mDNS) et en répondant pour les noms qui échouent à se résoudre, forçant les hôtes à s'authentifier auprès d'un écouteur contrôlé par l'attaquant."
keywords:
  - LLMNR poisoning
  - NBT-NS
  - Responder
  - NetNTLMv2
  - mDNS
---

# Capture et empoisonnement Net-NTLM

Lorsqu'un hôte Windows tente d'atteindre un nom que le DNS ne peut pas résoudre (une faute de frappe, un partage mis hors service, une recherche WPAD), il se rabat sur les protocoles de résolution de noms par **diffusion** : LLMNR, NBT-NS et mDNS. Ceux-ci n'ont aucune authentification : tout hôte du segment peut répondre « c'est moi ». En répondant, vous amenez la victime à se connecter à vous et à s'authentifier, vous remettant un défi-réponse **NetNTLMv2** que vous pouvez casser hors ligne ou relayer.

**Filiation.** La surface d'empoisonnement est un vestige de la résolution de noms WINS et NetBIOS, dont LLMNR et NBT-NS ont hérité le repli par diffusion. Responder en a rendu l'abus routinier dès 2012, et mDNS et IPv6 (mitm6, DHCPv6 et WPAD piégés) sont la même idée transposée sur des piles plus récentes.

## Pourquoi cela fonctionne

- **LLMNR** (UDP 5355) et **NBT-NS** (UDP 137) sont des replis multicast/broadcast sans validation de source.
- **mDNS** (UDP 5353) se comporte de façon similaire pour les noms `.local`.
- Un partage mal tapé, un mapping de lecteur obsolète ou l'auto-découverte de proxy **WPAD** produit un flux constant de recherches échouées à empoisonner.
- Lorsque la victime se connecte à votre faux service SMB/HTTP, Windows envoie automatiquement une réponse NetNTLMv2, souvent pour un utilisateur connecté et parfois pour un compte privilégié.

## Capture

```bash
# Responder: answer LLMNR/NBT-NS/mDNS and run rogue SMB/HTTP/etc. to capture NetNTLMv2
responder -I eth0 -wv

# Responder writes captured hashes to per-module files under logs/ (not the
# session log), e.g. logs/SMB-NTLMv2-SSP-<ip>.txt, in hashcat -m 5600 format
hashcat -m 5600 logs/SMB-NTLMv2-SSP-10.0.0.5.txt wordlist.txt -r rules/best64.rule
```

Ciblez spécifiquement WPAD : beaucoup de réseaux ne définissent jamais de véritable enregistrement DNS `wpad`, de sorte que la recherche de proxy de chaque navigateur peut recevoir une réponse, capturant largement l'authentification.

## De la capture à l'impact

Une réponse NetNTLMv2 capturée a deux usages :

- **La casser** (`-m 5600`) pour récupérer le texte en clair, puis réutiliser l'identifiant pour le [spraying](../credentials/password-spraying.md) et l'accès authentifié. Faisable uniquement si le mot de passe est faible.
- **La [relayer](relay.md)** sans la casser : réacheminer l'authentification en temps réel vers un autre service et y agir en tant que la victime. C'est le chemin à plus forte valeur, car il fonctionne quelle que soit la robustesse du mot de passe, tant que la cible n'impose pas la signature.

## Downgrade vers NetNTLMv1

Là où un hôte négocie encore l'ancien **NetNTLMv1**, la réponse est dérivée avec un DES faible et, face à un **défi serveur connu**, se casse pour retrouver le **hash NT lui-même** (et non seulement le mot de passe) en quelques heures sur un service DES dédié. Servez un défi fixe, capturez une réponse v1, et convertissez-la :

```bash
# Responder.conf: set Challenge = 1122334455667788 to force a static, crackable challenge
# then coerce a host (ideally a machine account) to authenticate and capture NetNTLMv1
# submit the captured NetNTLMv1 to crack.sh, which returns the NT hash near-instantly
```

Un NetNTLMv1 de **compte machine** forcé puis cassé pour retrouver son hash NT est immédiatement réutilisable en [pass-the-hash](pass-the-hash.md) pour cet ordinateur, c'est pourquoi downgrader la réponse d'un contrôleur de domaine est une voie connue vers son hash. Cela nécessite que la cible accepte encore NetNTLMv1, c'est donc une technique d'hôtes hérités, mais elle transforme une capture ponctuelle en un identifiant durable.

## Notes d'exploitation

- NetNTLMv2 n'est **pas** réutilisable comme un hash NT : il est lié à un défi serveur, il ne se casse ou ne se relaie donc que, jamais de pass-the-hash.
- Les réponses de comptes machine (`HOST$`) sont courantes et utiles : le NetNTLM d'un compte machine peut être relayé alors même que son mot de passe est aléatoire et incassable.
- `mitm6` abuse de l'autoconfiguration IPv6 (DHCPv6 piégé + DNS) pour rediriger la résolution de noms de façon plus fiable que l'empoisonnement par diffusion dans les réseaux IPv6, et se combine avec le relais.

## Outils

- **Responder** : empoisonnement LLMNR/NBT-NS/mDNS avec serveurs de capture piégés.
- **mitm6** : prise de contrôle DNS en IPv6 pour rediriger et capturer/relayer l'authentification.
- **Inveigh** : empoisonneur PowerShell/C# pour les hôtes d'attaque Windows.

## Références

- [Responder (lgandx): the maintained LLMNR/NBT-NS/mDNS poisoner](https://github.com/lgandx/Responder)
- [Inveigh (Kevin-Robertson): .NET IPv4/IPv6 MITM poisoner](https://github.com/Kevin-Robertson/Inveigh)
- [mitm6 (dirkjanm): IPv6 DNS takeover](https://github.com/dirkjanm/mitm6)
- [The Hacker Recipes: LLMNR/NBT-NS/mDNS poisoning](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/llmnr-nbtns-mdns-spoofing)
