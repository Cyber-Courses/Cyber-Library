---
title: "Spoofing ADIDNS : ajouter des enregistrements DNS pour empoisonner la résolution de noms"
order: 5
description: "Abus du DNS intégré à Active Directory, où les utilisateurs authentifiés peuvent créer des enregistrements, pour ajouter un enregistrement joker ou ciblé (WPAD, SCCM) qui transforme le serveur DNS du domaine en répondeur à l'échelle de l'entreprise, pour capturer ou relayer l'authentification."
keywords:
  - ADIDNS
  - wildcard record
  - WPAD
  - Powermad
  - dnstool
---

# Spoofing ADIDNS

Lorsque les zones DNS d'AD sont **intégrées à Active Directory (ADIDNS)**, la zone est stockée dans l'annuaire, et par défaut les **Authenticated Users** peuvent y **créer** des objets enfants (`dnsNode`). Ainsi, n'importe quel compte de domaine peut **ajouter des enregistrements DNS**. C'est une puissante primitive d'empoisonnement : contrairement à LLMNR/NBT-NS, qui ne répondent qu'aux recherches échouées sur le segment local, un enregistrement dans ADIDNS répond pour le **domaine entier**, de manière faisant autorité, tant qu'il existe.

## L'astuce du joker

Ajouter un enregistrement **joker** (`*`) fait résoudre par le serveur DNS du domaine **tout nom par ailleurs inconnu** vers votre hôte, le transformant en répondeur à l'échelle du domaine, comme l'empoisonnement LLMNR mais à l'échelle de l'entreprise et persistant :

```bash
# dnstool.py (from the krbrelayx repo): add a wildcard A record to your IP
dnstool.py -u 'EXAMPLE\user' -p 'password' --record '*' --action add --data <attacker-ip> <dc>
```

```powershell
# Powermad (on Windows): add a record via secure dynamic update
Invoke-DNSUpdate -DNSType A -DNSName evil -DNSData <attacker-ip>
```

Lancez ensuite [Responder](net-ntlm-capture-and-poisoning.md) ou [ntlmrelayx](relay.md) pour capturer ou relayer l'authentification qui afflue désormais vers vous.

## Enregistrements ciblés

Là où un joker serait trop bruyant, ajoutez un seul nom à forte valeur :

- **WPAD** : un enregistrement `wpad` (ou un enregistrement NS contournant la Global Query Block List) fait auto-découvrir votre proxy par tous les navigateurs, capturant largement l'authentification HTTP. La GQBL bloque un enregistrement A `wpad` direct, on utilise donc le contournement par enregistrement NS.
- **Noms SCCM / WSUS / WDS** : usurper l'infrastructure de mise à jour ou de déploiement pour forcer l'authentification ou pousser du contenu.
- Un hôte précis dont vous voulez détourner le nom pour une chaîne de relais.

## Notes d'exploitation

- L'enregistrement **fait autorité et couvre tout le domaine**, il atteint donc des clients qu'un empoisonneur link-local n'atteindrait jamais, et il persiste jusqu'à sa suppression ; nettoyez-le ensuite.
- Il ne nécessite qu'**un seul compte de domaine**, ce qui en fait un bon mouvement précoce pour sourcer des authentifications destinées au [relais](relay.md) (vers LDAP pour RBCD, ou AD CS ESC8).
- La mise en cache DNS implique que les effets ne sont pas instantanés et persistent après la suppression ; tenez compte des TTL lors du timing d'une capture.
- Associez cela à `mitm6` là où IPv6 est disponible, ou utilisez ADIDNS là où IPv6 est désactivé, pour sourcer les mêmes authentifications.

## Outils

- **dnstool.py** (dépôt krbrelayx) : ajouter/modifier/supprimer des enregistrements ADIDNS depuis Linux.
- **Powermad** (`Invoke-DNSUpdate`) : ajouter des enregistrements depuis Windows via la mise à jour dynamique sécurisée.
- **Responder / ntlmrelayx** : capturer ou relayer l'authentification redirigée.

## Références

- [The Hacker Recipes: ADIDNS spoofing](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing)
- [krbrelayx / dnstool (dirkjanm)](https://github.com/dirkjanm/krbrelayx)
- [HackTricks: AD DNS records](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-dns-records.html)
