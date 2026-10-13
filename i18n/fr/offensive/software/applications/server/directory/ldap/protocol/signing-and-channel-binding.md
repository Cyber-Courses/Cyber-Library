---
title: "Signature LDAP et channel binding : les faiblesses de transport qui permettent le relais"
description: "Comment l'absence de signature LDAP et de channel binding (EPA LDAPS) permet de relayer une authentification contrainte ou capturée vers l'annuaire, et comment déterminer la posture de signature d'un annuaire avant de tenter un relais."
keywords:
  - signature LDAP
  - channel binding
  - EPA
  - relais NTLM vers LDAP
  - LDAPS
---

# Signature LDAP et channel binding

Deux protections de transport décident si l'authentification vers un annuaire peut être **relayée** : la signature LDAP (sur du LDAP en clair, 389) et le channel binding (sur LDAPS, 636). Là où ni l'une ni l'autre n'est imposée, un attaquant qui capture ou contraint l'authentification NTLM d'une victime peut la relayer vers l'annuaire et agir en tant que la victime, ce qui est la porte d'entrée pour écrire des objets d'annuaire (ajouter un ordinateur, configurer la RBCD, accorder DCSync). Cette page est la vérification préalable ; le relais lui-même est traité sous relais NTLM dans Active Directory.

## Pourquoi c'est important

Une cible de relais n'est utile que si elle accepte une authentification relayée et non signée :

- **LDAP (389)** requiert la **signature** pour être sûr. Si l'annuaire n'impose pas la signature, le NTLM relayé sur du LDAP en clair est accepté, et l'attaquant effectue des opérations d'annuaire en tant que la victime relayée.
- **LDAPS (636)** est chiffré, il ne peut donc pas être signé de la même manière ; sa protection est le **channel binding** (EPA), qui lie l'authentification au canal TLS. Sans channel binding imposé, une authentification NTLM capturée ailleurs peut être relayée vers LDAPS.

Pendant longtemps, les deux étaient désactivées par défaut, de sorte qu'un domaine non durci est relayable vers LDAP, LDAPS, ou les deux.

## Vérifier la posture

Déterminez ce que l'annuaire impose avant de planifier un relais :

```bash
# NetExec reports LDAP signing and channel-binding enforcement
nxc ldap <dc> -u user -p pass -M ldap-checker

# Behavioral test: an unsigned simple bind that the server rejects indicates signing is required
ldapsearch -x -H ldap://<dc> -D 'EXAMPLE\user' -w pass -b '' -s base
```

Trois états comptent : signature non requise (le relais vers 389 fonctionne), channel binding non requis (le relais vers 636 fonctionne), et les deux imposés (le relais LDAP est fermé, pivotez vers une cible de relais différente comme SMB ou l'enrôlement HTTP d'AD CS).

## Notes d'exploitation

- La vérification conditionne l'attaque : il est inutile de contraindre l'authentification vers LDAP si l'annuaire impose à la fois la signature et le channel binding.
- Même avec LDAP durci, le point de terminaison d'**enrôlement web d'AD CS** (HTTP) est une cible de relais courante que la signature/le channel binding sur LDAP ne protège pas, donc un annuaire durci ne ferme pas entièrement le relais.
- La signature et le channel binding sont des paramètres machine **par DC**, pas un unique interrupteur à l'échelle du domaine, et ils dérivent (durcissement progressif, surcharges de registre, application de stratégie de groupe échouée). Vérifiez chaque DC accessible plutôt que de traiter un seul résultat comme valable pour tout le domaine ; un seul DC non durci suffit pour y relayer.

## Outils

- **NetExec (nxc) -M ldap-checker** : rapport d'imposition de la signature et du channel binding.
- **ldapsearch** : vérification manuelle du comportement de bind non signé.
- **ntlmrelayx** (Impacket) : le relais lui-même, une fois la posture connue.

## Références

- The Hacker Recipes: NTLM relay
- Microsoft: LDAP channel binding and signing requirements
