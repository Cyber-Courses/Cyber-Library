---
title: "NTLM : capturer, relayer et rejouer l'authentification Windows"
order: 2
description: "Abus de l'authentification NTLM dans Active Directory : capture des défis-réponses NetNTLM par empoisonnement de la résolution de noms, coercition de l'authentification depuis des machines privilégiées, relais vers d'autres services, et rejeu direct d'un hash avec pass-the-hash."
keywords:
  - NTLM
  - NetNTLMv2
  - NTLM relay
  - pass the hash
  - coercion
---

# NTLM

NTLM est l'ancien protocole d'authentification défi-réponse de Windows, encore activé presque partout aux côtés de Kerberos. Ses faiblesses sont structurelles : la réponse est dérivée directement du hash NT (de sorte que le hash seul authentifie, sans mot de passe nécessaire), et le protocole n'a aucun lien intrinsèque avec le service auquel il est envoyé (de sorte qu'une authentification capturée peut être réacheminée ailleurs). Ces deux faits produisent toute la famille des attaques NTLM.

## Les quatre mouvements

- **Capture** : forcer une victime à s'authentifier auprès de vous (en empoisonnant la résolution de noms), et enregistrer la réponse NetNTLMv2 pour la casser hors ligne.
- **Coercition** : amener une machine privilégiée, en particulier un contrôleur de domaine, à s'authentifier auprès de vous à la demande via un déclencheur RPC ou de protocole.
- **Relais** : réacheminer une authentification capturée ou forcée vers un service tiers en temps réel et y agir en tant que la victime, sans jamais connaître le mot de passe.
- **Rejeu (pass-the-hash)** : s'authentifier directement avec un hash NT volé, puisque le protocole n'a jamais besoin du texte en clair.

La capture et la coercition produisent l'authentification ; le relais et le pass-the-hash la consomment. Les protections de la cible du relais (signature SMB, [signature LDAP et channel binding](../../../ldap/protocol/signing-and-channel-binding.md)) décident si le relais est possible.

## Pages

- **[Capture et empoisonnement Net-NTLM](net-ntlm-capture-and-poisoning.md)** : récolter l'authentification par empoisonnement LLMNR/NBT-NS/mDNS.
- **[Spoofing ADIDNS](adidns.md)** : ajouter des enregistrements DNS intégrés à AD pour empoisonner la résolution de noms à l'échelle du domaine.
- **[Coercition](coercion.md)** : forcer les machines privilégiées à s'authentifier (PetitPotam, PrinterBug, et d'autres).
- **[Relais](relay.md)** : réacheminer l'authentification vers des cibles SMB, LDAP et HTTP.
- **[Pass-the-hash](pass-the-hash.md)** : s'authentifier et exécuter avec un hash NT volé.

## Références

- [SpecterOps: the renaissance of NTLM relay attacks](https://posts.specterops.io/the-renaissance-of-ntlm-relay-attacks-everything-you-need-to-know-abfc3677c34e)
- [Microsoft (MS-NLMP): NTLM authentication protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/b38c36ed-2804-4868-a9ff-8dd3182128e4)
- [The Hacker Recipes: NTLM](https://www.thehacker.recipes/ad/movement/ntlm/)
