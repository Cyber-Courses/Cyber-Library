---
order: 3
title: "NTLM relay : transférer l'authentification pour agir en tant que la victime"
description: "Transférer une authentification NTLM capturée ou coercée en temps réel vers un service tiers (SMB, LDAP, HTTP, AD CS) pour agir en tant que la victime sans connaître le mot de passe, et les protections de signature et de channel binding qui l'arrêtent."
keywords:
  - NTLM relay
  - ntlmrelayx
  - RBCD
  - ESC8
  - signature SMB
---

# Relay

NTLM n'a aucun lien entre l'authentification et le service auquel elle était destinée. Ainsi, une authentification produite pour le serveur A peut être **transférée** vers le serveur B, où vous la complétez et agissez en tant que la victime, le tout sans jamais connaître son mot de passe. Le relay transforme une authentification [capturée](net-ntlm-capture-and-poisoning.md) ou [coercée](coercion.md) directement en accès sur un autre système. Ce vers quoi vous pouvez relayer est décidé entièrement par la posture de signature de la cible.

**Lignée.** Le relay a débuté avec SMBRelay en 2001, reflétant l'authentification directement vers l'hôte d'origine. Microsoft a bloqué l'auto-relay puis a renforcé la signature SMB et LDAP, le MIC, et le channel binding au fil des versions successives, chacune fermant un chemin. Le relay moderne répond en traversant les protocoles, transférant vers LDAP, vers l'enrôlement web AD CS, et même de retour dans Kerberos, partout où la signature ou le channel binding reste non imposé.

## Ce qui arrête un relay

La signature et le channel binding lient l'authentification à la session ou au canal TLS, cassant le transfert :

- **Signature SMB** : si la cible SMB exige la signature, le relay SMB vers elle échoue. Les DC l'exigent ; les serveurs membres souvent non.
- **[Signature LDAP et channel binding](../../../ldap/protocol/signing-and-channel-binding.md)** : le relay LDAP nécessite que la signature **ne** soit **pas** imposée ; le relay LDAPS nécessite en plus que le channel binding (EPA) soit désactivé. Les DC avec les deux imposés ne peuvent pas être ciblés par un relay sur LDAP.
- **HTTP (EPA)** : l'enrôlement web AD CS et d'autres endpoints HTTP sont relayables lorsque l'Extended Protection for Authentication n'est pas imposé.

Le travail consiste à trouver un chemin où la *source* peut être coercée et où la *cible* n'impose pas la protection concernée.

## Cibles de relay à forte valeur

```bash
# Relay to LDAP: set RBCD on the victim computer object, or dump the domain
ntlmrelayx.py -t ldap://<dc> --delegate-access --no-dump

# Relay coerced DC auth to AD CS web enrollment (ESC8) -> DC certificate
ntlmrelayx.py -t http://<ca>/certsrv/certfnsh.asp --adcs --template DomainController

# Relay to SMB on a non-signing host and execute
ntlmrelayx.py -t smb://<host> -c 'whoami'

# Relay to LDAP to add Shadow Credentials on a victim (no RBCD computer needed)
ntlmrelayx.py -t ldap://<dc> --shadow-credentials --shadow-target 'victim$'
```

`ntlmrelayx` relaie aussi vers **MSSQL** (par exemple un serveur de site SCCM coercé vers sa base de données de site, voir [prise de contrôle de site SCCM](../../../../configuration/sccm/site-takeover.md)) et les endpoints HTTP Exchange, et prend en charge le **multi-relay** : il répond à la victime avec une **redirection HTTP 307** pour que le client se réauthentifie pour chaque cible supplémentaire. Un seul challenge-response capturé ne peut pas satisfaire plusieurs challenges serveur, donc il s'agit d'un **déclencheur de coercition** pilotant des authentifications fraîches répétées, et non d'une authentification réutilisée plusieurs fois, et cela ne fonctionne que lorsque le client coercé suit les redirections.

- **Relay vers LDAP + RBCD** : configurer la délégation contrainte basée sur les ressources sur un objet ordinateur, que vous pouvez ensuite usurper en tant que n'importe quel utilisateur, un chemin courant de l'authentification machine coercée vers l'admin local sur cette machine.
- **Relay vers AD CS (ESC8)** : relayer une authentification HTTP de DC ou d'utilisateur coercée vers l'enrôlement web de la CA et obtenir un certificat en tant que ce principal ; un certificat de DC est une compromission du domaine.
- **Relay vers SMB** : exécuter ou extraire le SAM sur les serveurs membres qui n'exigent pas la signature.

## La chaîne complète

La coercition et le relay se combinent dans le chemin canonique du non-authentifié vers le domaine : **coercer un DC sur HTTP (WebClient) -> relayer vers AD CS ESC8 -> certificat de DC -> s'authentifier en tant que le DC -> DCSync.** Chaque maillon dépend d'une protection manquante (coercition HTTP possible, EPA désactivé sur la CA).

## Notes d'exploitation

- Vous ne pouvez pas relayer une authentification vers l'hôte qui l'a produite sous Windows moderne (le correctif de réflexion), donc le relay se fait toujours vers une cible *différente*.
- Les authentifications de compte machine sont entièrement relayables même si leurs mots de passe sont incrackables, ce qui explique pourquoi coercer `DC$` et le relayer bat l'idée d'essayer de le cracker.
- Combinez le relay avec `mitm6` (prise de contrôle DNS IPv6) pour alimenter largement les authentifications et relayer vers LDAP pour du RBCD à l'échelle du domaine.
- **Windows Server 2025 réduit les classiques, mais par hôte et par service** : un **DC** 2025 active le channel binding LDAP et la signature SMB plus largement, donc le relay-vers-LDAP(S) contre lui meurt. **ESC8 dépend de l'hôte d'enrôlement web AD CS, pas du DC** : il ne se ferme que si l'hôte de la CA exécute une build/configuration avec EPA et HTTPS imposés, donc dans un parc mixte (DC 2025, CA plus ancienne avec EPA toujours désactivé) ESC8 reste viable. Vérifiez la version de l'hôte de la CA et l'état d'EPA séparément de ceux du DC. De nouvelles techniques réflectives (abusant des drapeaux de négociation de sign/seal SMB) maintiennent le relay en vie quoi qu'il en soit, donc testez la posture réelle plutôt que de la supposer.

## Outils

- **ntlmrelayx.py** (Impacket) : le moteur de relay, avec `--delegate-access` (RBCD), `--shadow-credentials`, `--adcs` (ESC8), exécution SMB, dump LDAP, et multi-relay.
- **Responder / mitm6** : alimenter les authentifications à relayer.
- **krbrelayx** : chemins de relay Kerberos / délégation non contrainte associés.

## Références

- [SpecterOps: the renaissance of NTLM relay attacks](https://posts.specterops.io/the-renaissance-of-ntlm-relay-attacks-everything-you-need-to-know-abfc3677c34e)
- [SecureAuth: a technical guide to relaying credentials everywhere](https://secureauth.com/resources/blog/we-love-relaying-credentials-technical-guide)
- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
- [The Hacker Recipes: NTLM relay](https://www.thehacker.recipes/ad/movement/ntlm/relay)
